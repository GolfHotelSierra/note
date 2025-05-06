[toc]

> [源码](https://github.com/CompVis/stable-diffusion)

# Inference

## T2I

> diffusers 中将 sampler、unet、tokenizer、encoder 都封装到了 pipeline 中，而 official code 则可以看做主要交给了 model 负责

```python
uc = None
# cfg需要额外生成一个不受text embedding控制的latent feature
# get_learned_conditioning()表示对text进行embedding; 一般使用OpenAICLIPEmbedder
if scale != 1.0:
    uc = model.get_learned_conditioning(
        batch_size * [""]) # 通过输入空字符串表示没有text控制
if isinstance(prompts, tuple):
    prompts = list(prompts)

c = model.get_learned_conditioning(prompts)
shape = [C, H // f, W // f]

# 采样器的实例化对象管理大部分生成过程; 调用sample()函数生成最终的latent image(循环去噪逻辑在sample()内部)
samples_ddim, _ = sampler.sample(S=ddim_steps, # 采样步数
                                  conditioning=c,
                                  batch_size=n_samples,
                                  shape=shape,
                                  verbose=False,
                                  unconditional_guidance_scale=scale,
                                  unconditional_conditioning=uc,
                                  eta=ddim_eta,
                                  x_T=start_code)

x_samples_ddim = model.decode_first_stage(samples_ddim) # 通过VAE decoder还原为image; vae是放在model身上的
```





# 基础模块

> 主要关注这些目录以其中的内容， `models` 和 `configs` 目录主要是存放配置文件，`ldm` 目录是主要模块的代码实现，`scripts` 目录是包括推理在内的一些脚本

## `DDIMSampler`

- sampler 的实例化对象默认使用 `sampler = DDIMSampler(model)`，来自 `from ldm.models.diffusion.ddim import DDIMSampler`

**`__init__()`**

```python
def __init__(self, model, schedule="linear", **kwargs):
    super().__init__()
    self.model = model # unet、encoder、vae都绑在了model上; 一般是一个LatentDiffusion实例
    self.ddpm_num_timesteps = model.num_timesteps
    self.schedule = schedule # scheduler
```

**`sampler.sample()`**

```python
@torch.no_grad()
def sample(self,
            S,
            batch_size,
            shape,
            conditioning=None,
            ...
            ):
    if conditioning is not None:
        ...

    self.make_schedule(ddim_num_steps=S, ddim_eta=eta, verbose=verbose) # 根据不同的sheduler计算并存储中间值(e.g. \alpha相关的常数)
    # sampling
    C, H, W = shape
    size = (batch_size, C, H, W)
    print(f'Data shape for DDIM sampling is {size}, eta {eta}')

    samples, intermediates = self.ddim_sampling(...) # 主要的采样流程
```

- **`ddim_sampling()`**

  ```python
  @torch.no_grad() # 注意这个函数仅用于推理, 训练时的采样不在这里实现
  def ddim_sampling(self, ...):
      device = self.model.betas.device
      b = shape[0]
      img = torch.randn(shape, device=device) # 生成x_T输入的高斯噪声
      timesteps = self.ddim_timesteps
      intermediates = ...
      time_range = np.flip(timesteps)
      total_steps = timesteps.shape[0]
  
      iterator = tqdm(time_range, desc='DDIM Sampler', total=total_steps) # 根据采样步数, 生成一个迭代器(e.g. [1,2,...,50])
  
      for i, step in enumerate(iterator):
          index = total_steps - i - 1
          ts = torch.full((b,), step, device=device, dtype=torch.long) # 当前时间步的矩阵
  
          outs = self.p_sample_ddim(img, cond, ts, ...)
          img, pred_x0 = outs
  
      return img, intermediates
  ```

- **`p_sample_ddim()`**

  ```python
  @torch.no_grad() # 相当于将训练时预测噪声的函数完全绑定在model上, 损失函数计算来自model.apply_model()的返回值
  def p_sample_ddim(self, x, c, t, ...):
      b, *_, device = *x.shape, x.device
  
      # 通过apply_model()函数调用unet进行噪声预测
      if unconditional_conditioning is None or unconditional_guidance_scale == 1.:
          e_t = self.model.apply_model(x, t, c)
      else:
          # 如果使用cfg, 需要额外进行一次生成, 所以需要额外一份输入, 这里使用在batch_size维度拼接的方法实现
          x_in = torch.cat([x] * 2)
          t_in = torch.cat([t] * 2)
          c_in = torch.cat([unconditional_conditioning, c])
          e_t_uncond, e_t = self.model.apply_model(x_in, t_in, c_in).chunk(2) # 同时输出有text控制和没有的预测噪声
          e_t = e_t_uncond + unconditional_guidance_scale * (e_t - e_t_uncond) # cfg公式
  
  
      # Prepare variables
      ...
  
      # 通过预测出的噪声还原出下一个时间步的latent image
      # current prediction for x_0
      pred_x0 = (x - sqrt_one_minus_at * e_t) / a_t.sqrt()
      if quantize_denoised:
          pred_x0, _, *_ = self.model.first_stage_model.quantize(pred_x0)
      # direction pointing to x_t
      dir_xt = (1. - a_prev - sigma_t**2).sqrt() * e_t
      noise = sigma_t * noise_like(x.shape, device, repeat_noise) * temperature
      if noise_dropout > 0.:
          noise = torch.nn.functional.dropout(noise, p=noise_dropout)
      x_prev = a_prev.sqrt() * pred_x0 + dir_xt + noise
      return x_prev, pred_x0
  ```



## unet

### `LatentDiffuion`、`DDPM`

- 通过<u>*配置文件*</u>指定实例化时使用的类

```python
class DDPM(pl.LightningModule):
    # classic DDPM with Gaussian diffusion, in image space
    def __init__(self,
                 unet_config,
                 ...):
        self.model = DiffusionWrapper(unet_config, conditioning_key) # 这个model是单纯的unet了; DiffusionWrapper是额外的封装, 去掉封装可以看做是个UNetModel实例
        

class LatentDiffusion(DDPM):
    """main class"""
    def __init__(self,
                 first_stage_config,
                 cond_stage_config,
                 ...):

        self.instantiate_first_stage(first_stage_config)
        self.instantiate_cond_stage(cond_stage_config)
```

**`apply_model()`**

```python
def apply_model(self, x_noisy, t, cond, return_ids=False):
    if isinstance(cond, dict):
        # hybrid case, cond is exptected to be a dict
        pass
    else:
        if not isinstance(cond, list):
            cond = [cond]
        key = 'c_concat' if self.model.conditioning_key == 'concat' else 'c_crossattn' # 在SpatialTransformer中使用cross attn融合text embedding
        cond = {key: cond}

    x_recon = self.model(x_noisy, t, **cond) # 预测噪声

    if isinstance(x_recon, tuple) and not return_ids:
        return x_recon[0]
    else:
        return x_recon
```



### `TimestepEmbedSequential`

- 类似于 `nn.Sequential`，将<u>*多个 module 组成在一起*</u>

```python
class TimestepEmbedSequential(nn.Sequential, TimestepBlock):

    def forward(self, x, emb, context=None):
        for layer in self: # 循环执行每个module
            if isinstance(layer, TimestepBlock):
                x = layer(x, emb)
            elif isinstance(layer, SpatialTransformer):
                x = layer(x, context)
            else: # conv, down/upsample等
                x = layer(x)
        return x
```



### `UNetModel`

- 对去噪 <u>*unet 网络*</u>的实现

**`__init__()`**

```python
def __init__(self, ...):

    self.time_embed = nn.Sequential(
        linear(model_channels, time_embed_dim),
        nn.SiLU(),
        linear(time_embed_dim, time_embed_dim),
    )

    # 将多个TimestepEmbedSequential(即level)存储在一个nn.ModuleList中
    self.input_blocks = nn.ModuleList( # 在最开始加入一个conv对输入通道数进行调整
        [
            TimestepEmbedSequential(
                conv_nd(dims, in_channels, model_channels, 3, padding=1)
            )
        ]
    )

    # 开始组装input_blocks
    for level, mult in enumerate(channel_mult): # mult是放缩系数, 指明down/upsample对通道数的放缩倍数
        for _ in range(num_res_blocks): # 每个level塞num_res_blocks个(ResBlock, SpatialTransformer)组合块
            layers = [
                ResBlock(...)]
            ch = mult * model_channels
            if ds in attention_resolutions:
                 layers.append(
                    AttentionBlock(...) if not use_spatial_transformer else SpatialTransformer(...))

            self.input_blocks.append(TimestepEmbedSequential(*layers))
        if level != len(channel_mult) - 1: # 除了最后一层都加一个downsample模块
            out_ch = ch
            self.input_blocks.append(
                TimestepEmbedSequential(
                    ResBlock(...)
                    if resblock_updown
                    else Downsample(...)
                )
            )

    self.middle_block = TimestepEmbedSequential(
        ResBlock(...),
        AttentionBlock(...) if not use_spatial_transformer else SpatialTransformer(...),
        ResBlock(...),
    )

    self.output_blocks = nn.ModuleList([])
    for level, mult in list(enumerate(channel_mult))[::-1]:
        for i in range(num_res_blocks + 1):
            ich = input_block_chans.pop()
            layers = [
                ResBlock(...)
            ]
            ch = model_channels * mult
            if ds in attention_resolutions:
                layers.append(
                    AttentionBlock(...) if not use_spatial_transformer else SpatialTransformer(...)
                )
            if level and i == num_res_blocks:
                out_ch = ch
                layers.append(
                    ResBlock(...)
                    if resblock_updown
                    else Upsample(...)
                )
                ds //= 2
            self.output_blocks.append(TimestepEmbedSequential(*layers))
    self.out = nn.Sequential( # 调整输出特征图通道数
            normalization(ch),
            nn.SiLU(),
            zero_module(conv_nd(dims, model_channels, out_channels, 3, padding=1)),
        )
```

**`forward()`**

```python
def forward(self, x, timesteps=None, context=None, y=None,**kwargs):
    hs = []
    t_emb = timestep_embedding(timesteps, self.model_channels, repeat_only=False) # sin-cos编码
    emb = self.time_embed(t_emb) # 计算time embedding

    h = x.type(self.dtype)
    for module in self.input_blocks:
        h = module(h, emb, context) # module是TimestepEmbedSequential实例, 可以认为一个level就是一个TimestepEmbedSequential实例
        hs.append(h) # 用于skip connection(因为input_blocks里面先放了一个单独的conv, 所以sc提供给decoder的就是对应层的输入)
    h = self.middle_block(h, emb, context)
    for module in self.output_blocks:
        h = th.cat([h, hs.pop()], dim=1) # skip connection在channel维度上拼接
        h = module(h, emb, context)
    h = h.type(x.dtype)
    return self.out(h) # 调整通道数
```



#### `ResBlock`

- 用于提取特征图并融合 time embedding

```python
class ResBlock(TimestepBlock):
    def __init__(self, ...):
        super().__init__()
        ...

        self.in_layers = nn.Sequential(
            normalization(channels),
            nn.SiLU(),
            conv_nd(dims, channels, self.out_channels, 3, padding=1),
        )

        self.emb_layers = nn.Sequential( # 将time channel和特征图的channel对齐
            nn.SiLU(),
            linear(
                emb_channels,
                2 * self.out_channels if use_scale_shift_norm else self.out_channels,
            ),
        )
        self.out_layers = nn.Sequential(
            normalization(self.out_channels),
            nn.SiLU(),
            nn.Dropout(p=dropout),
            zero_module(
                conv_nd(dims, self.out_channels, self.out_channels, 3, padding=1)
            ),
        )

        if self.out_channels == channels: # unet的encoder部分没有sc, 使用全1矩阵代替
            self.skip_connection = nn.Identity()
        # 对sc(即channel上concat)后的通道数进行调整, 通道对齐才能进行残差
        elif use_conv:
            self.skip_connection = conv_nd(
                dims, channels, self.out_channels, 3, padding=1
            )
        else:
            self.skip_connection = conv_nd(dims, channels, self.out_channels, 1)

    def forward(self, x, emb):
        h = self.in_layers(x)
        emb_out = self.emb_layers(emb).type(h.dtype)
        while len(emb_out.shape) < len(h.shape): # 特征图可能是(bs,h,w)也可能是(bs,c,h,w), 而原始的emb_out维度为(bs,c), 需要补充1或2个维度才能进行广播
            emb_out = emb_out[..., None]
        h = h + emb_out # time embedding和feture相加融合
        h = self.out_layers(h)
        return self.skip_connection(x) + h # 残差
```



#### `SpatialTransformer` 与 `BasicTransformerBlock`

**`SpatialTransformer`**

- 融合 condition

```python
class SpatialTransformer(nn.Module):

    def __init__(self, in_channels, n_heads, d_head,
                 depth=1, dropout=0., context_dim=None):
        super().__init__()
        self.in_channels = in_channels
        inner_dim = n_heads * d_head
        self.norm = Normalize(in_channels)
		# 将latent feature的channel和transformer需要的seq的dim进行对齐(虽然有to_q,to_k,to_v, 但这里好像又对齐了一次)
        self.proj_in = nn.Conv2d(in_channels, 
                                 inner_dim,
                                 kernel_size=1,
                                 stride=1,
                                 padding=0)

        self.transformer_blocks = nn.ModuleList(
            [BasicTransformerBlock(inner_dim, n_heads, d_head, dropout=dropout, context_dim=context_dim)
                for d in range(depth)]
        )

        # 把dim维度还原回channel维度
        self.proj_out = zero_module(nn.Conv2d(inner_dim,
                                              in_channels,
                                              kernel_size=1,
                                              stride=1,
                                              padding=0))
    
    def forward(self, x, context=None):
        b, c, h, w = x.shape
        x_in = x
        x = self.norm(x)
        x = self.proj_in(x)
        x = rearrange(x, 'b c h w -> b (h w) c') # (bs,c,h,w)变为(bs,h*w,c)对应(bs,seq_len,dim)
        for block in self.transformer_blocks:
            x = block(x, context=context)
        x = rearrange(x, 'b (h w) c -> b c h w', h=h, w=w)
        x = self.proj_out(x)
        return x + x_in
```

**`BasicTransformerBlock`**

- stable diffusion 中利用 cross attn 实现 self attn 的功能

```python
class BasicTransformerBlock(nn.Module):
    def __init__(self, dim, n_heads, d_head, dropout=0., context_dim=None, gated_ff=True, checkpoint=True):
        super().__init__()
        self.attn1 = CrossAttention(query_dim=dim, heads=n_heads, dim_head=d_head, dropout=dropout)  # is a self-attention
        self.ff = FeedForward(dim, dropout=dropout, glu=gated_ff)
        self.attn2 = CrossAttention(query_dim=dim, context_dim=context_dim,
                                    heads=n_heads, dim_head=d_head, dropout=dropout)  # is self-attn if context is none
        self.norm1 = nn.LayerNorm(dim)
        self.norm2 = nn.LayerNorm(dim)
        self.norm3 = nn.LayerNorm(dim)
        self.checkpoint = checkpoint

    def forward(self, x, context=None):
        x = self.attn1(self.norm1(x)) + x # self attn
        x = self.attn2(self.norm2(x), context=context) + x # cross attn
        x = self.ff(self.norm3(x)) + x # feed forward
        return x
```

**`CrossAttention`**

> 有一些实现会将多头拆分为 (bs, h, seq_len, small_dim)，但是更常用的还是 (b*h, n, d) 的风格

```python
class CrossAttention(nn.Module):
    def __init__(self, query_dim, context_dim=None, heads=8, dim_head=64, dropout=0.):
        super().__init__()
        inner_dim = dim_head * heads
        context_dim = default(context_dim, query_dim)

        self.scale = dim_head ** -0.5
        self.heads = heads

        self.to_q = nn.Linear(query_dim, inner_dim, bias=False)
        self.to_k = nn.Linear(context_dim, inner_dim, bias=False)
        self.to_v = nn.Linear(context_dim, inner_dim, bias=False)

        self.to_out = nn.Sequential(
            nn.Linear(inner_dim, query_dim),
            nn.Dropout(dropout)
        )

    def forward(self, x, context=None, mask=None):
        h = self.heads

        q = self.to_q(x)
        context = default(context, x)
        k = self.to_k(context)
        v = self.to_v(context)

        q, k, v = map(lambda t: rearrange(t, 'b n (h d) -> (b h) n d', h=h), (q, k, v)) # 拆分多头

        sim = einsum('b i d, b j d -> b i j', q, k) * self.scale

        if exists(mask):
            ...

        # attention, what we cannot get enough of
        attn = sim.softmax(dim=-1)

        out = einsum('b i j, b j d -> b i d', attn, v)
        out = rearrange(out, '(b h) n d -> b n (h d)', h=h)
        return self.to_out(out)
```

