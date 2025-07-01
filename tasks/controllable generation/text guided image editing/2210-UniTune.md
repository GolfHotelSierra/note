[toc]

> [UniTune: Text-Driven Image Editing by Fine Tuning a Diffusion Model on a Single Image](https://arxiv.org/abs/2210.09477)
>
> [official code](https://github.com/xuduo35/UniTune)

# 问题提出

- 之前的模型需要额外的用户输入 (e.g. mask)
- 之前的模型无法跨任务使用 (i.e. 单一模型只能专用于某种特定的编辑或特定的图像)



# 贡献

- 利用 DreamBooth 的思路，利用 concat 得到一个包含原图信息和标记信息的新 prompt



# 思路

- 该算法包含以下步骤，
  0. $c$ 为对编辑操作描述的 prompt，$x^{(b)}$ 表示输入的原始图像，$x_0$ 表示编辑后的图像，$c^{(b)}$ 和 DreamBooth 中的 placeholder 类似
  
  1. 与 **DreamBooth 的训练方式**类似，冻结 unet，对 $c^{(b)}$ 进行训练
  
  2. 将 **$c$ 和 $c^{(b)}$ concat 在一起**，i.e. `[rare_tokens] edit_prompt]`，然后使用 concat 版本的 prompt **搭配一个高权重 (论文设置为 32) 的 cfg** 即可生成图像，<u>*不需要额外的训练*</u>
  
     > 该论文方法中的 cfg 中 unconditioned 的 prompt 为空字符串，和一般做法一致
  
  3. 在推理时<u>*不使用高斯噪声作为初始化*</u>，而是**对 $x^{(b)}$ 加噪作为初始化** (相当于是一个 i2i 任务)；论文发现对 $x^{(b)}$ 的加噪<u>*在 $0.8\leq t \leq0.98$ 最优*</u> (i.e. 可以加一个比较重的噪声)

- UniTune *<u>不需要对每一个新的 prompt 重新训练</u>*



# Limitation

- 对原图的忠实和对编辑的忠实依然会有冲突

  可能的解决方案，是使用更好的采样方式