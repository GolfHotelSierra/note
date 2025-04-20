[toc]

> [源码](https://github.com/lllyasviel/ControlNet)

# 基础模块

> controlnet 的项目结构和 stable diffusion 的 official code 很相似

- 在 stable diffusion 中大致涉及两个类 `LatentDiffusion` (绑定并调用 text encoder、vae、unet 等)，`UNetModel` (对 unet 的具体实现)；controlnet 同样需要和两个类有类似的逻辑，而 controlnet 的主体部分大体上除了照搬了 sd encoder 的代码 (从`__init__()` 函数中复制)，还需要加上 zero module 并删除 sd decoder 的逻辑，需要额外的一个类来实现

  > 理论上还有个 XxxSampler 类，但是 controlnet 不涉及这部分的修改，忽略

- 综上，controlnet 对应 <u>*3 个类*</u>，**`class ControlNet(nn.Module)`** (实现 controlnet)，**`class ControlLDM(LatentDiffusion)`** (额外绑定并调用 controlnet)，**`class ControlledUnetModel(UNetModel)`** (重写了 `forward()` 函数，增加和 controlnet 的输出相加融合的逻辑)



## ControlNet

> 从原版的 UNetModel 的 `__init__()` 函数中复制了很多内容

**`__init__()`**

```python
class ControlNet(nn.Module):
    def __init__(
            ...
    ):
        ...

        self.input_blocks = ... # 同样用input_blocks作为sd encoder
        self.zero_convs = nn.ModuleList([self.make_zero_conv(model_channels)]) # 零卷积

        self.input_hint_block = TimestepEmbedSequential( # 在进入controlnet前有一个hint_block, 进行分辨率和通道对齐
            conv_nd(dims, hint_channels, 16, 3, padding=1),
            nn.SiLU(),
            conv_nd(dims, 16, 16, 3, padding=1),
            nn.SiLU(),
            conv_nd(dims, 16, 32, 3, padding=1, stride=2),
            nn.SiLU(),
            conv_nd(dims, 32, 32, 3, padding=1),
            nn.SiLU(),
            conv_nd(dims, 32, 96, 3, padding=1, stride=2),
            nn.SiLU(),
            conv_nd(dims, 96, 96, 3, padding=1),
            nn.SiLU(),
            conv_nd(dims, 96, 256, 3, padding=1, stride=2),
            nn.SiLU(),
            zero_module(conv_nd(dims, 256, model_channels, 3, padding=1))
        )

        ...
        
        for level, mult in enumerate(channel_mult):
            for nr in range(self.num_res_blocks[level]):
                layers = [
                    ...
                ]
                ...
                self.input_blocks.append(TimestepEmbedSequential(*layers))
                self.zero_convs.append(self.make_zero_conv(ch)) # zero module不能直接被TimestepEmbedSequential管理, 所以额外维护一个list, 和input_blocks对齐; 在foward()函数中可以看到两者是分别被调用的
                self._feature_size += ch
                input_block_chans.append(ch)
            if level != len(channel_mult) - 1:
                ... # 判断是否要添加down/upsample
                input_block_chans.append(ch)
                self.zero_convs.append(self.make_zero_conv(ch)) # 一样的操作, 增加zero module
                ds *= 2
                self._feature_size += ch

        ...
        
        self.middle_block = ...
        self.middle_block_out = self.make_zero_conv(ch) # 增加zero module
        self._feature_size += ch

        
    def make_zero_conv(self, channels):
        return TimestepEmbedSequential(zero_module(conv_nd(self.dims, channels, channels, 1, padding=0)))
```

**`forward()`**

```python
def forward(self, x, hint, timesteps, context, **kwargs):
    t_emb = timestep_embedding(timesteps, self.model_channels, repeat_only=False)
    emb = self.time_embed(t_emb)

    guided_hint = self.input_hint_block(hint, emb, context) # 实例化hint_block

    outs = []

    h = x.type(self.dtype)
    for module, zero_conv in zip(self.input_blocks, self.zero_convs):
        if guided_hint is not None: # hint_block只执行一次, 执行结束就被赋值为None了
            h = module(h, emb, context)
            h += guided_hint
            guided_hint = None
        else:
            h = module(h, emb, context)
        outs.append(zero_conv(h, emb, context))

    h = self.middle_block(h, emb, context)
    outs.append(self.middle_block_out(h, emb, context))

    return outs # controlnet每层的输出存储在一个list中
```



## ControlLDM

**`apply_model()`**

```python
class ControlLDM(LatentDiffusion):

    def apply_model(self, x_noisy, t, cond, *args, **kwargs):
        assert isinstance(cond, dict)
        diffusion_model = self.model.diffusion_model

        cond_txt = torch.cat(cond['c_crossattn'], 1)

        if cond['c_concat'] is None:
            eps = diffusion_model(x=x_noisy, timesteps=t, context=cond_txt, control=None, only_mid_control=self.only_mid_control)
        else:
            control = self.control_model(x=x_noisy, hint=torch.cat(cond['c_concat'], 1), timesteps=t, context=cond_txt) # ControlNet的forward()返回的list
            control = [c * scale for c, scale in zip(control, self.control_scales)]
            eps = diffusion_model(x=x_noisy, timesteps=t, context=cond_txt, control=control, only_mid_control=self.only_mid_control) # 两个分支中的UNetModel的forward()函数都多了一个control参数, 所以这部分也需要重写

        return eps
```



## ControlledUnetModel

**`forward()`**

```python
class ControlledUnetModel(UNetModel):
    def forward(self, x, timesteps=None, context=None, control=None, only_mid_control=False, **kwargs):
        hs = []
        with torch.no_grad():
            t_emb = timestep_embedding(timesteps, self.model_channels, repeat_only=False)
            emb = self.time_embed(t_emb)
            h = x.type(self.dtype)
            for module in self.input_blocks:
                h = module(h, emb, context)
                hs.append(h)
            h = self.middle_block(h, emb, context)

        if control is not None:
            h += control.pop() # controlnet结构图中有对middle blocke的输出额外和controlnet做一次相加融合; 即only_mid_control参数控制的部分

        for i, module in enumerate(self.output_blocks):
            if only_mid_control or control is None:
                h = torch.cat([h, hs.pop()], dim=1)
            else:
                h = torch.cat([h, hs.pop() + control.pop()], dim=1) # hs.pop()+control.pop()是controlnet相加融合的部分, 而外层的concat则是skip connection
            h = module(h, emb, context)

        h = h.type(x.dtype)
        return self.out(h)
```

