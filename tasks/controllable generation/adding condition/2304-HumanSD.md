[toc]

> [HumanSD: A Native Skeleton-Guided Diffusion Model for Human Image Generation](https://arxiv.org/abs/2304.04269)
>
> [源码](https://github.com/IDEA-Research/HumanSD)

# 贡献

- 提出了 ControlNet 这种双分支 (Dual Branch) 结构引入 condition 时可能存在矛盾
- 通过 **concat 的方式**引入 pose condition
- 提出了利用 heatmap，增加对人物部分生成的监督的权重，并且**权重可以根据训练过程自适应调整** (基本上是**去噪前期权重要大一些，后期权重要小一些**)





# 思路

- ControlNet 在生成过程中的矛盾

  <img src="assets/image-20250326233341194.png" alt="image-20250326233341194" style="zoom:50%;" />

  简言之，由于 ControlNet 其实是在 SD 已经通过 encoder 进行了一定的生成后再引入控制的，那么除了要额外引入 condition 外，<u>*还需要修改已经生成的 latent image 中生成错误的部分*</u> (e.g. 上图中 Negative Features，黑色表示要抑制 $f_{\theta}^O$ 中位置生成不正确的红色高亮部分)；论文认为**使用 concat 的方式让控制信息在生成的一开始就被引入**是更好的选择

  > 但目前还是有很多工作是基于 ControlNet 的

-  **The Heatmap-guided Denoise Loss**

  将**预测出的噪声和噪声的 gt 的差值作为输入**，训练一个模型预测人物的大致 pose

  > 详见 A.4. Detailed Implementation of the HeatmapGuided Denoising Loss 章节
  >
  > 论文中表示这个模型是之前论文已经有提出的，很好奇什么姿态预测模型是将噪声的差值作为输入的？

  <img src="assets/image-20250326234129526.png" alt="image-20250326234129526" style="zoom:50%;" />

  <img src="assets/image-20250326234331693.png" alt="image-20250326234331693" style="zoom:55%;" />

  可以简单理解为，差值越大，就代表这是去噪的早期阶段，模型越容易预测出人物的姿态，就会基于更高的权重进行损失函数的计算