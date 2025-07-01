[toc]

> [official code](https://github.com/tencent-ailab/IP-Adapter)

# Preliminary

- **计算公式**

  <img src="assets/image-20250301151425774.png" alt="image-20250301151425774" style="zoom:40%;" />

- IP-Adapter 源码基于 diffusers 实现，主要**利用了 AttnProcessor 机制**

- 主要关注 `IP-Adapter/ip_adapter/attention_processor.py` 和 `IP-Adapter/tutorial_train.py`

  > `IP-Adapter/ip_adapter/ip_adapter.py` 中也有 IPAdapter 的实现，做了更多封装同时只用于推理

- 对于 **attention 的权重命名规则**规则为，

  e.g. `down_blocks.1.attentions.0.transformer_blocks.0.attn1.processor`，

  - `down_blocks.1`: 代表在整个 UNet 当中 downblocks 当中索引;
  - `attentions.0`: 代表第几个 SpatialTransformer 块
  - `transformer_blocks.0`: 代表第几个 BasicTransformerBlock (每个 block 里一般 1 个 self-attn + 1 个 cross-attn)
  - **`attn1.processor` 代表自注意力层, `attn2.processor`代表交叉注意力层**





# 代码实现

## `IPAdapter` / 准备 `encoder_hidden_state`

```python
# IP-Adapter/tutorial_train.py
class IPAdapter(torch.nn.Module):
    def forward(self, noisy_latents, timesteps, encoder_hidden_states, image_embeds):
        
        # 获取额外控制信息的token
        ip_tokens = self.image_proj_model(image_embeds)
        # 将所有控制信息concat在一起, 在attn_processor中再拆开来处理
        encoder_hidden_states = torch.cat([encoder_hidden_states, ip_tokens], dim=1)
        
        ...
```



## `XxAttnProcessor`

- 当类名出现 `2_0` 后缀，表示必须使用 pytorch>=2.0 的版本

- `AttnProcessor` 用于 self attn，默认的 processor，没有修改任何逻辑

  **`IPAttnProcessor` 用于替换 cross attn，核心处理逻辑**

  `CNAttnProcessor` 用于 ControlNet

```python
class IPAttnProcessor(nn.Module):
    r"""
    Attention processor for IP-Adapater.
    Args:
        hidden_size (`int`):
            The hidden size of the attention layer.
        cross_attention_dim (`int`):
            The number of channels in the `encoder_hidden_states`.
        scale (`float`, defaults to 1.0):
            the weight scale of image prompt.
        num_tokens (`int`, defaults to 4 when do ip_adapter_plus it should be 16):
            The context length of the image features.
    """
    def __init__(self, hidden_size, cross_attention_dim=None, scale=1.0, num_tokens=4):
        super().__init__()

        self.hidden_size = hidden_size
        self.cross_attention_dim = cross_attention_dim
        self.scale = scale
        self.num_tokens = num_tokens

        # 对额外控制信息的训练网络参数
        self.to_k_ip = nn.Linear(cross_attention_dim or hidden_size, hidden_size, bias=False)
        self.to_v_ip = nn.Linear(cross_attention_dim or hidden_size, hidden_size, bias=False)

    def __call__(
        self,
        attn,
        hidden_states,
        encoder_hidden_states=None,
        attention_mask=None,
        temb=None,
        *args,
        **kwargs,
    ):
        ...

        query = attn.to_q(hidden_states)

        if encoder_hidden_states is None:
            encoder_hidden_states = hidden_states
        else:
            # 额外控制信息的token长度是固定的, 从而对encoder_hidden_state进行拆分
            end_pos = encoder_hidden_states.shape[1] - self.num_tokens
            encoder_hidden_states, ip_hidden_states = (
                encoder_hidden_states[:, :end_pos, :],
                encoder_hidden_states[:, end_pos:, :],
            )
            if attn.norm_cross:
                encoder_hidden_states = attn.norm_encoder_hidden_states(encoder_hidden_states)

        ...

        # 对额外信息计算Key和Value
        ip_key = self.to_k_ip(ip_hidden_states)
        ip_value = self.to_v_ip(ip_hidden_states)

        ...
        
		# 相加融合(公式实现)
        hidden_states = hidden_states + self.scale * ip_hidden_states

        ...
        
        return hidden_states
```



## 将 `XxAttnProcessor` 绑定回 UNet 上

```python
def main():
    ...
    
    #ip-adapter
    image_proj_model = ImageProjModel(
        cross_attention_dim=unet.config.cross_attention_dim,
        clip_embeddings_dim=image_encoder.config.projection_dim,
        clip_extra_context_tokens=4,
    )
    
    # init adapter modules
    attn_procs = {}
    unet_sd = unet.state_dict()
    # 按照attn的网络名绑定不同的AttnProcessor
    for name in unet.attn_processors.keys():
        cross_attention_dim = None if name.endswith("attn1.processor") else unet.config.cross_attention_dim
        if name.startswith("mid_block"):
            hidden_size = unet.config.block_out_channels[-1]
        elif name.startswith("up_blocks"):
            block_id = int(name[len("up_blocks.")])
            hidden_size = list(reversed(unet.config.block_out_channels))[block_id]
        elif name.startswith("down_blocks"):
            block_id = int(name[len("down_blocks.")])
            hidden_size = unet.config.block_out_channels[block_id]
        if cross_attention_dim is None:
            attn_procs[name] = AttnProcessor()
        else:
            layer_name = name.split(".processor")[0]
            weights = {
                "to_k_ip.weight": unet_sd[layer_name + ".to_k.weight"],
                "to_v_ip.weight": unet_sd[layer_name + ".to_v.weight"],
            }
            attn_procs[name] = IPAttnProcessor(hidden_size=hidden_size, cross_attention_dim=cross_attention_dim)
            attn_procs[name].load_state_dict(weights)
    # 通过set_attn_processor()绑定回unet上
    unet.set_attn_processor(attn_procs)
    adapter_modules = torch.nn.ModuleList(unet.attn_processors.values())
    
    # 将绑定好的同一个unet实例传入IPAdapter
    ip_adapter = IPAdapter(unet, image_proj_model, adapter_modules, args.pretrained_ip_adapter_path)
    
    # 正常训练过程
    ...
```





# 参考资料

- [知乎【AIGC 03】IP_Adapter 代码讲解](https://zhuanlan.zhihu.com/p/699420409)

- [【AIGC图像实践篇 01】IP-Adapter 原理和实践](https://zhuanlan.zhihu.com/p/683504661)