[toc]

# 背景: ELBO, VAE, and Hierarchical VAE

- 假设给定了一个**图像数据集 $\{x_i\}$**，这些图像是从**真实分布 p(x)** 中采样得到的，**生成模型的目的就是根据 $\{x_i\}$ 反过来预测出 p(x)** (预测的分布一般记作 $p_{\theta}(x)$)

  利用 $p_{\theta}(x)$ 可以采样得到新的图像 (i.e. 生成出新的图像) 或判断一个图像是否在这个分布中



## ELBO 与 VAE

- 如果希望计算 p(x)，需要从另一个分布 p(z) 中进行采样

  > 反过来想，如果计算 p(x) 不需要从其它分布采样，i.e. 可以从 p(x) 中采样，那么 p(x) 就是已知的了，与假设矛盾

  p(z) 一般称作 **latent distribution (隐变量分布)**，**隐变量 (latent variable)**一般<u>*维度更低、包含更核心信息*</u> (e.g. 颜色、亮度、纹理等)

  > 但是，如果 p(z) 完全没有约束，也很难仅通过 Decoder 还原出有意义的图像 (相比于完全随机采样，按照正态分布进行采样是属于增加了约束的)

- 理论上，可以通过 $p(x)=\int p(x,z)dz$ 或 $p(x)=\frac{p(x,z)}{p(z|x)}$ 来计算 $p(x)$，但是，<u>*并不是很好计算*</u>；**ELBO** 的核心思想是**使用其它的分布来计算/替代 p(x)**

- **ELBO** 的推理方式如下，

  > $p(x)$ 当 $x\sim p(x)$ 时，$p(x)$ 要尽可能大
  >
  > 简单来说，$q_{\phi}(z|x)$ 是拥有可学习参数的分布，一般使用某种网络结构代替
  >
  > $D_{KL}$ 表示 KL 散度，表示两个分布之间的差距

  <img src="assets/2025年1月8日093552.png" alt="2025年1月8日093552" style="zoom: 40%;" />

  - **最大化 ELBO**，相当于最小化 $D_{KL}$ (i.e. 模拟的<u>*分布 q 和真实的分布 p 更加接近*</u>)，相当于 <u>*ELBO 更加接近 p(x)*</u>

- 对 ELBO 进一步拆解，

  > 这里的 $p_{\theta}(x|z)$ 也是拥有可学习参数的分布

  <img src="assets/image-20250108093907862.png" alt="image-20250108093907862" style="zoom: 67%;" />

  - 最大化 ELBO，相当于**最大化 reconstruction term**，**最小化 prior matching term**；随着 reconstruction term 接近 ELBO，也意味着接近 p(x)，即可以**使用 ${\log p_{\theta}(x|z)}$ 来计算/代替 p(x)**；同时，**隐变量 z 来自于 $q_{\phi}(z|x)$ 并且受到 $p(z)$ 的约束**

- 根据式 (19) 中的“一来一回”，就有了 **VAE** 中的 <u>*Encoder、Decoder 结构*</u>，

  <img src="assets/image-20250108094851372.png" alt="image-20250108094851372" style="zoom: 67%;" />

- VAE 使用的 **Loss Function** 为，

  <img src="assets/image-20250108095016229.png" alt="image-20250108095016229" style="zoom:60%;" />

  - ${L_2}$ 替换了式 (19) 中的 reconstruction term，i.e. 不使用最大似然，而是直接比较原始图像和生成图像之间的差距

  - 对于先验 $p(z)$，是需要进行猜测的，一般使用<u>*标准正态分布*</u>；考虑到还要计算 $D_{KL}$，$q_{\phi}(z|x)$ 也<u>*使用正态分布进行建模*</u>

    正态分布的 $D_{KL}$ 是有解析式的 (i.e. 上式中的 $L_{KL}$)

- <u>*VAE 的训练*</u>，

  理论上，有了 $q_{\phi}(z|x)$ 就可以从中采样出隐变量 z 了，但是一些采样方式 (e.g. 蒙特卡洛) 是无法反向传播的，通过 **reparameterization trick 重参数技巧** (i.e. 引入随机变量 $\epsilon$，同时不影响反向传播)，实现梯度回传

  <img src="assets/image-20250108095828173.png" alt="image-20250108095828173" style="zoom:60%;" />



## HVAE

- 简单来说，**HVAE** 是<u>*计算隐变量的隐变量*</u>，

  <img src="assets/image-20250108170206739.png" alt="image-20250108170206739" style="zoom:50%;" />

  而 **MHVAE** (i.e. 上图) 则是增加了马尔可夫链约束

  > 马尔科夫链：下一个状态只与当前状态有关

- MHVAE 的 <u>*ELBO*</u> 如下，

  <img src="assets/image-20250108170424235.png" alt="image-20250108170424235" style="zoom: 67%;" />

- 类似的，ELBO 可以进一步拆分，

  <img src="assets/image-20250108170512114.png" alt="image-20250108170512114" style="zoom: 70%;" />





# VDM (Variational Diffusion Models)

- <u>*VDM 是特殊的 MHVAE*</u>，相较于 MHVAE 增加了下面这些限制，

  <img src="assets/image-20250108223124593.png" alt="image-20250108223124593" style="zoom: 50%;" />

  - **隐变量的维度与输入图片维度一致**

  - encoder (i.e. $q_{\phi}$) **不再需要学习**，而是 **$x_{t}$ 是以 $x_{t-1}$ 为中心的高斯分布**

    <img src="assets/image-20250108223319324.png" alt="image-20250108223319324" style="zoom: 67%;" />

    - **$\alpha_t$ 是一个超参数**

  - **$x_T$ 接近标准高斯分布**

    <img src="assets/image-20250108223721647.png" alt="image-20250108223721647" style="zoom: 67%;" />

    > 与 VAE 中假定 $p(z)$ 是一个标准正态分布类似

- <u>*在 VDM 中重新推断 ELBO*</u>，

  ![2025年1月8日224717](assets/2025年1月8日224717.png)

  - 与 VAE 类似，在最小化 $D_{KL}$ 的同时，<u>*最大化 $p_{\theta}(x_0|x_1)$ 就是最大化 ELBO，就是在接近 $p(x)$*</u>

- <u>*consistency term 中现在有 $x_{t-1}$ 和 $x_{t+1}$ 两个变量*</u>，可以<u>*继续减少变量*</u>，减轻优化的负担

  > 就当做一个结论来理解吧

  利用<u>*马尔科夫链的性质*</u>，重新推导 ELBO 得到，、

  <img src="assets/image-20250108230057985.png" alt="image-20250108230057985" style="zoom: 70%;" />

  - **reconstruction term**：和 VAE 中的类似，但是不再使用 L2 Loss 来代替

  - **prior matching term**：也与 VAE 中的类似，只要 T 足够大 (i.e. 加噪的次数足够多)，$x_t$ 总会逐渐接近标准高斯分布，i.e. 这一项的 $D_{KL}$ 会接近 0，<u>*可以忽视*</u>

  - **denoising matching term**：**真实的去噪结果 (i.e. $q$) 和推测的去噪结果 (i.e. $p_{\theta}$) 能够尽量接近**

    > 这一项从 $t=2$ 开始计算到 $t=T$，所以 VDM 虽然可以“一步加噪”，但去噪过程要一步一步来

- 在式 58 中，$q(x_{t-1}|x_t,x_0)$ 通过贝叶斯可以转化为加噪过程 (i.e. $q(x_{t}|x_{t-1})$)，进而不需要通过参数学习得到，

  > $p(x)$ 和 $p(z)$ 不是同一个 p.d.f.，所以没有限定 $q$ 只能表示加噪过程而不能表示去噪过程
  >
  > 但通过下标可以判断 $p$ 不存在需要学习的参数，而 $p_{\theta}$ 是需要学习的

  - 首先，可以先推理出<u>*“一步加噪”*</u>的公式 (i.e. 直接通过 $x_0$ 加噪到 $x_t$ 而不需要一步一步加噪)，

    <img src="assets/image-20250108231243709.png" alt="image-20250108231243709" style="zoom:60%;" />

    之后通过<u>*重参数技巧*</u>，就可以直接从 $x_0$ 采样 $x_t$ 了

  - 将式 70 代入，利用贝叶斯公式整理式 58 中 <u>*denoising matching term*</u> 项中 $q(x_{t-1}|x_t,x_0)$ 项，

    <img src="assets/2025年1月8日231522.png" alt="2025年1月8日231522" style="zoom: 33%;" />

    > 在<u>推理过程中</u>，<u>*$\alpha_t$ 也是需要提前准备的*</u>；不过一般 $x_T$ 不是通过公式计算得到的，而是<u>*直接从正态分布中采样*</u>

    i.e. denoising matching term 中的真值计算完成了

    > $\sum_q(t)$ 中不存在任何需要学习的参数，在 $p_{\theta}$ 中这一项可以是学习的，也可以直接拿过去用
    >
    > $\mu_q(x_t,x_0)$ 这一项包含了 $x_0$，而在推理过程中是不知道 $x_0$ 的，所以是需要学习的
  
  - 根据式 84，真值是一个正态分布，那么自然地对于式 58 中的 <u>*$p_{\theta}(x_{t-1}|x_t)$ 的建模也是一个正态分布 (i.e. 通过网络学习均值 $\mu_{\theta}$ 和方差)*</u>
  
    对于式 58 中的 denoising matching term，两个正态分布的 $D_{KL}$ 是可以套用公式的 (VAE 中有类似的操作)，
  
    <img src="assets/image-20250109125714604.png" alt="image-20250109125714604" style="zoom:60%;" />
  
    i.e. 为了最小化这个 $D_{KL}$，等价于 $p_{\theta}$ 中学习到的均值 $\mu_{\theta}$ 要尽量接近式 84 中的真值
  
  - 根据式 92 进一步拆分，上面提到 $\mu_q$ 中需要学习的是 $x_0$，那么引入，
  
    <img src="assets/image-20250109131942690.png" alt="image-20250109131942690" style="zoom:60%;" />
  
    > 与 $\mu_q$ 相比只是将 $x_0$ 变成了可学习的
  
    <img src="assets/2025年1月9日131453.png" alt="2025年1月9日131453"  />
  
    但是，实践表明使用网络<u>*直接预测 t 时刻的完整的图片是比较困难*</u>
  
  - 重新利用式 70 的“一步加噪”的公式，
  
    <img src="assets/image-20250109131800404.png" alt="image-20250109131800404" style="zoom:60%;" />
  
    > 理论上，在推理过程中，可以直接从 $x_t$ 通过 $\epsilon_0$ 来预测 $x_0$，但是这样的效果一般是不好的，因为在损失函数中，<u>*只要求 $x_t$ 对 $x_{t-1}$ 的去噪是尽量准确的*</u>，但从 $x_t$ 预测 $x_0$ 就不一定精准了
  
    再次<u>*重新整理出 $\mu_q$ 和 $\mu_{\theta}$，*</u>
  
    <img src="assets/2025年1月9日132112.png" alt="2025年1月9日132112" style="zoom:30%;" />
  
    根据式 92，代入整理，
  
    ![2025年1月9日132315](assets/2025年1月9日132315.png)
  
    i.e. 不需要预测 t 时刻的整张图像，而是**使用网络在 t 时刻预测 $x_t$ 相对于 $x_0$ 一共添加了多少的噪声**
  
  
  
  

# SGM (Score-based Generative Models)

- **SGM** 是 VDM 的另一种解释视角

- 首先，需要引入 **Tweedie’s Formula**，对于<u>*符合正态分布的随机变量 z*</u>，

  <img src="assets/image-20250109134039130.png" alt="image-20250109134039130" style="zoom: 67%;" />

  > 当做结论来用就好

- 根据 Tweedie’s Formula，以及根据式 70，$x_t$ 是一个符合正态分布的随机变量且 $\mu_{x_t}=\sqrt{\bar{\alpha}_t}x_0$

  <img src="assets/image-20250109134406080.png" alt="image-20250109134406080" style="zoom:60%;" />

  代入整理可得，

  <img src="assets/image-20250109134605462.png" alt="image-20250109134605462" style="zoom:60%;" />

- 代入式 84 同样替换掉 $x_0$，这里得到新的 $\mu_q$ 和 $\mu_{\theta}$ 的表达，

  <img src="assets/2025年1月9日135333.png" alt="2025年1月9日135333" style="zoom:30%;" />

  同样代入式 92 重新计算 $D_{KL}$ 的表达式，

  ![2025年1月9日140132](assets/2025年1月9日140132.png)

- $\nabla\log p(x_t)$ 不太好计算，利用式 115 和式 133，可以得到，

  <img src="assets/image-20250109225159123.png" alt="image-20250109225159123" style="zoom:60%;" />

  i.e. $\nabla\log p(x_t)$ 是加噪 $\epsilon_0$ 的反方向；$\nabla\log p(x_t)$ 也称作 **score function**，其意义是<u>*去噪方向*</u>





# Guidance

## Classifier Guidance

- 除了 $p(x)$，有时我们更关注的是在条件 $y$ 下的 $p(x|y)$；这里的条件 $y$，可以是 prompt 也可以是 image 等

- 根据 SGM 中的思路，展开 $p(x|y)$ 的 score function

  > score function 和 $\epsilon$ 只差了一个系数，所以两者的展开也没有差很多

  <img src="assets/image-20250109230741874.png" alt="image-20250109230741874" style="zoom:60%;" />

  i.e. 除了原始的 score function，<u>*还需要训练一个根据输出图像 $x_t$ 预测条件 $y$ 的分类器*</u>

  > 但是这个分类器是比较难训练的，因为 $x_t$ 可能是一个噪声很大的图

- 为了更好地根据条件 $y$ 来生成，一般会加入一个超参数 $\gamma$，

  <img src="assets/image-20250109231147645.png" alt="image-20250109231147645" style="zoom:60%;" />



## CFG (Classifier-Free Guidance)

- 为了省去额外训练分类器的过程，根据式 165 和式 166 整理得到，

  <img src="assets/image-20250109231327342.png" alt="image-20250109231327342" style="zoom:60%;" />

  i.e. 在生成过程中，**根据条件 $y$ 生成一个图像，再无视条件 $y$ 生成一个图像**

- 一般来说，CFG 中的 <u>*$\gamma$ 是一个大于 1 的值，如 7.5*</u>





# 引用文献

- [论文：从大一统的角度审视扩散模型](https://arxiv.org/abs/2208.11970)



# 参考资料

- [youtube 上的 VAE 介绍](https://www.youtube.com/watch?v=qJeaCHQ1k2w)
