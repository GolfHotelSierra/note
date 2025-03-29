[toc]

# 基础模块

## Pipeline

**`XxxPipeline` 类**中<u>*将 VAE、U-Net、采样器等部件组合在一起*</u>

> 不同的 Pipeline 可能实现细节不同，但大体结构一致

类中关键的是 **`__call__()` 方法**，其中<u>*调用 VAE、U-Net、采样器等部件*</u>并推理出 1 个 batch 的图像

```python
def __call__(
    self,
    prompt: Union[str, List[str]] = None,
    height: Optional[int] = None,
    width: Optional[int] = None,
    num_inference_steps: int = 50,
    timesteps: List[int] = None,
    guidance_scale: float = 7.5,
    negative_prompt: Optional[Union[str, List[str]]] = None,
    num_images_per_prompt: Optional[int] = 1,
    eta: float = 0.0,
    ...
):
```

在 `__init__()` 中对于 **VAE、U-Net、采样器等部件的实现类**

```python
def __init__(
    self,
    vae: AutoencoderKL,
    text_encoder: CLIPTextModel,
    tokenizer: CLIPTokenizer,
    unet: UNet2DConditionModel,
    scheduler: KarrasDiffusionSchedulers,
    ...
):
```

1. **获取 prompt 并进行 encode**

```python
# 3. Encode input prompt
prompt_embeds, negative_prompt_embeds = self.encode_prompt(...)
```

2. **从 sheduler 中随机抽取 timestep**

```python
# 4. Prepare timesteps
timesteps, num_inference_steps = retrieve_timesteps(self.scheduler, num_inference_steps, device, timesteps)
```

3. **生成随机噪声作为初始化的 latent 图像**

```python
# 5. Prepare latent variables
num_channels_latents = self.unet.config.in_channels
latents = self.prepare_latents( # latent是一个随机噪声
    ...
)
```

4. **调用 unet 网络进行噪声预测**

 ```python
 with self.progress_bar(total=num_inference_steps) as progress_bar:
     for i, t in enumerate(timesteps):
         # 如果使用cfg需要将输入翻倍
         latent_model_input = torch.cat([latents] * 2) if self.do_classifier_free_guidance else latents
         latent_model_input = self.scheduler.scale_model_input(latent_model_input, t)
 
         # 使用unet进行一次噪声预测
         noise_pred = self.unet(
             latent_model_input,
             t,
             encoder_hidden_states=prompt_embeds,
             ...
         )[0]
 
         # 如果是cfg需要把输出拆开来, 一个conditioned, 一个unconditioned
         if self.do_classifier_free_guidance:
             noise_pred_uncond, noise_pred_text = noise_pred.chunk(2)
             noise_pred = noise_pred_uncond + self.guidance_scale * (noise_pred_text - noise_pred_uncond)
 
         if self.do_classifier_free_guidance and self.guidance_rescale > 0.0:
             # Based on 3.4. in https://arxiv.org/pdf/2305.08891.pdf
             noise_pred = rescale_noise_cfg(noise_pred, noise_pred_text, guidance_rescale=self.guidance_rescale)
         
         ...
         
         # 根据预测出的噪声进行一个时间步的去噪
         latents = self.scheduler.step(noise_pred, t, latents, **extra_step_kwargs, return_dict=False)[0]
 ```

5. **scheduler 中的 `step()` 函数会根据预测出的噪声还原为 $x_{t-1}$**

   > 这个方法也可以根据当前时间步预测出的噪声，直接还原出 $x_0$

```python
latents = self.scheduler.step(noise_pred, t, latents, **extra_step_kwargs, return_dict=False)[0]
```

6. **通过 `vae.decode()` 函数将 latent 图像还原回 image 中**

   还原后的 image 是 `Image` 对象

```python
if not output_type == "latent":
    image = self.vae.decode(latents / self.vae.config.scaling_factor, return_dict=False, generator=generator)[0]
    image, has_nsfw_concept = self.run_safety_checker(image, device, prompt_embeds.dtype)
else:
    image = latents
    has_nsfw_concept = None

...
# 可以选择是否将返回值封装为一个对象
if not return_dict:
	return (image,)

return StableDiffusionPipelineOutput(images=image, nsfw_content_detected=has_nsfw_concept)
```



## unet: UNet2DConditionModel

**`UNet2DConditionModel` 类是 diffuers 中常用的对 unet 的实现**

主要包括 **`time_proj, time_embedding, down_blocks, mid_block, up_blocks, conv_in, conv_out` 等结构**

1. <u>*UNet2DConditionModel 中实例化了上面这些结构*</u>，每个结构的<u>*具体实现则在配置文件中*</u>个性化指定

```python
class UNet2DConditionModel(ModelMixin, ConfigMixin, UNet2DConditionLoadersMixin):
    def __init__(...):
        ...
        self.conv_in = nn.Conv2d(
            in_channels, block_out_channels[0], kernel_size=conv_in_kernel, padding=conv_in_padding
        )
        ...
        # 将timestep进行sin-cos编码
        elif time_embedding_type == "positional":
            self.time_proj = Timesteps(block_out_channels[0], flip_sin_to_cos, freq_shift)
        ...
        # 对timestep encoding进行embedding
        self.time_embedding = TimestepEmbedding(...)
        self.down_blocks = nn.ModuleList([])
        self.up_blocks = nn.ModuleList([])
        # down_block_type从配置文件中读取
        for i, down_block_type in enumerate(down_block_types):
            ...
            down_block = get_down_block(...)
        
        if mid_block_type == ...
            self.mid_block = ...

        for i, up_block_type in enumerate(up_block_types):
            up_block = get_up_block(...)

        self.conv_out = nn.Conv2d(...)
```

2. **`unet/config.json`** 可能存在类似的配置，

   > CrossAttnDownBlock2D = (Resblk + SpatialTransformer)*n + downsample

```python
{
    ...
    "down_block_types": [
    "CrossAttnDownBlock2D",
    "CrossAttnDownBlock2D",
    "CrossAttnDownBlock2D",
    "DownBlock2D"
  ],
  "mid_block_type": "UNetMidBlock2DCrossAttn",
  "up_block_types": [
    "UpBlock2D",
    "CrossAttnUpBlock2D",
    "CrossAttnUpBlock2D",
    "CrossAttnUpBlock2D"
  ],
  ...
}
```

3. **`CrossAttnDownBlock2D` 类**，实现 <u>*timestep 融合 + condition 融合*</u>

```python
class CrossAttnDownBlock2D(nn.Module):
    def __init__(...):
        for i in range(num_layers): # *n
            resnets.append(ResnetBlock2D(...)) # Resblk
            if not dual_cross_attention:
                attentions.append(Transformer2DModel(...)) # SpatialTransformer
        if add_downsample:
            self.downsamplers = nn.ModuleList( # downsample
                [
                    Downsample2D(
                        out_channels, use_conv=True, out_channels=out_channels, padding=downsample_padding, name="op"
                    )
                ]
            )
```

4. UNet2DConditionModel 的 **`forward()` 函数**调用每个结构

   > 每个 `downsample_block` 会返回多个残差输出的元组 `res_samples`
   >
   > i.e. 每一层 (CrossAttnDownBlock2D e.t.c.) 可能会收集到好几个输出？

```python
def forward(
        self,
        sample: torch.FloatTensor,
        timestep: Union[torch.Tensor, float, int],
        encoder_hidden_states: torch.Tensor,
        ...):

    # 0. center input if necessary
    if self.config.center_input_sample:
        sample = 2 * sample - 1.0

    # 1. 获取time embedding
    timesteps = timestep
    t_emb = self.time_proj(timesteps)
    emb = self.time_embedding(t_emb, timestep_cond)

    # 2. pre-process
    sample = self.conv_in(sample)

    # 3. down 
    # temb注入time embedding, encoder_hidden_states注入condition(e.g. text embedding)
    down_block_res_samples = (sample,) # downblk的输出暂存在down_block_res_samples中, skip connection中使用
    for downsample_block in self.down_blocks:
        sample, res_samples = downsample_block(
            hidden_states=sample,
            temb=emb,
            encoder_hidden_states=encoder_hidden_states,
            ...)
        down_block_res_samples += res_samples # 存储downblk的输出
    # 4. mid
    sample = self.mid_block(
            sample,
            emb,
            encoder_hidden_states=encoder_hidden_states,
            ...)

    # 5. up
    for i, upsample_block in enumerate(self.up_blocks):
        res_samples = down_block_res_samples[-len(upsample_block.resnets) :] # 相当于peek操作
        down_block_res_samples = down_block_res_samples[: -len(upsample_block.resnets)] # 相当于pop操作
        sample = upsample_block( # skip connection
            hidden_states=sample,
            temb=emb,
            res_hidden_states_tuple=res_samples,
            encoder_hidden_states=encoder_hidden_states,
            ...)

     # 6. post-process
    sample = self.conv_out(sample)

    return UNet2DConditionOutput(sample=sample)
```





# Inference

## T2I

```python
import torch
from diffusers import StableDiffusionXLPipeline

pipe = StableDiffusionXLPipeline.from_pretrained(
    "stabilityai/stable-diffusion-xl-base-1.0", torch_dtype=torch.float16
)
pipe = pipe.to("cuda")

prompt = "a photo of an astronaut riding a horse on mars"
image = pipe(prompt).images[0]
```





# 修改模块

## `AttnProcessor` 修改 attention 的调用逻辑

diffusers 将**调用 attn blk 的逻辑封装在了 `AttnProcessor` 中**

> AttnProcessor 修改的是调用逻辑，而不是 attn 本身的结构 (如果想修改 attn 本身的结构，需要直接实现新的 attn，并在配置文件中进行配置)

```python
class AttnProcessor:
    def __call__(
        self,
        attn: Attention, # attn会被传入
        hidden_states: torch.Tensor,
        encoder_hidden_states: Optional[torch.Tensor] = None,
        attention_mask: Optional[torch.Tensor] = None,
        temb: Optional[torch.Tensor] = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        
        query = attn.to_q(hidden_states) # 调用attn中的方法
        ...
        return hidden_states
```

1. `AttnProcessor` <u>*不需要继承*</u>，只要实现一个**拥有 `__call__()` 方法并且函数参数保持一致**的类
2. 通过 **`set_attn_processor()` 函数**传入自定义的 AttnProcessor

```python
pipe.unet.set_attn_processor(custom_processor)
```





# 参考资料

- [sd 官方源码 + diffusers 源码解读](https://zhouyifan.net/2024/01/23/20230713-SD3/)