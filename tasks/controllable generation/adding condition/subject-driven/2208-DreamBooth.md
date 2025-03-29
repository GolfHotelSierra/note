[toc]

>[DreamBooth: Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation](https://arxiv.org/abs/2208.12242)
>
>[源码](https://github.com/google/dreambooth)
>
>CVPR 2023

# 贡献

- <u>*Subject-Driven*</u> 的开山作之一；通过 <u>*rare-token 代表要引入的新的 "subject"*</u>，通过 class noun 利用
- 构建新的损失函数，<u>*平衡新引入的 "subject" 的生成效果和原始模型的生成效果*</u>





# 思路

## Framework

**提示词构建：**

- e.g. `a [V] [class noun]`
  - 使用一个特殊的 `[V]` token；这个 token 一般<u>*不选择常见的词汇和随机组成的词汇*</u>，一般是<u>*从 token vocabulary 中选择罕见的 token*</u> (1 到 3 个 token) 组成
  - <u>*class noun*</u> 是对 "subject" 的大致描述 (e.g. 如果 `[V]` 是某只具体的狗，`[class noun]` 是 "dog")

**损失函数：**

- 包含 "subject" 的生成图像的重建损失
- **Class-specific Prior Preservation Loss (PPL)**：仅使用 class noun (e.g. 提示词为 `a [class noun]`) <u>*分别使用原始的网络和经过训练的网络生成图像*</u>，在两个图像间计算 l2 loss

**训练过程：**

- 使用 3 到 5 张图，训练 5 分钟左右可以学习一个新的 "subject"





# Evaluation Metric

- DINO：gt 和生成图像之间的 feature 计算 similarity
- CLIP-I：通过 CLIP 的 image encoder，gt 和生成图像之间的 feature 计算 similarity
- CLIP-T：i.e. CLIP Score





# Ablation

- PPL 损失函数有效
- 加入 `[class noun]` 有效