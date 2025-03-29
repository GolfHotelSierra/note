[toc]

> [官方文档](https://lightning.ai/docs/pytorch/stable/)

# 搭建一个基本的训练流程

## 基本流程

0. **安装 pytorch lightning 库**

```python
import lightning.pytorch as pl
```

1. pytorch lightning 的思路是有一个类将训练中的**常用过程封装起来** (e.g. prepare model、training step、validation step e.t.c.)，然后将这个类**交给 pytorch lightning 管理** (hook 思路)

   （1）**继承 `pl.LightningModule`**

   （2）**实现 `training_step()` 和 `configure_optimizers()`**

   这个类中封装的模型仍然使用常用的 pytorch 提供的 `nn.Module` 实现

```python
class LitAutoEncoder(pl.LightningModule):
    def __init__(self, encoder, decoder):
        super().__init__()
        self.encoder = encoder # nn.Module
        self.decoder = decoder # nn.Module

    def training_step(self, batch, batch_idx):
        x, y = batch
        x = x.view(x.size(0), -1)
        z = self.encoder(x)
        x_hat = self.decoder(z)
        loss = F.mse_loss(x_hat, x)
        return loss

    def configure_optimizers(self):
        optimizer = torch.optim.Adam(self.parameters(), lr=1e-3)
        return optimizer
```

2. **开始训练**

   （1）**实例化 pytorch lightning 类**

   （2）**实例化训练器 Trainer**；**调用 `fit()` 函数开始训练**

```python
# pytorch lightning中可以使用pytorch中的DataLoader
dataset = MNIST(os.getcwd(), download=True, transform=transforms.ToTensor())
train_loader = DataLoader(dataset)

# 实例化pytorch lightning类
autoencoder = LitAutoEncoder(Encoder(), Decoder())

# 实例化训练器Trainer
trainer = pl.Trainer(...) # 实例化时可以传入参数控制训练流程, 下文会逐步介绍这些参数
# 调用fit()开始训练(不需要写for循环)
trainer.fit(model=autoencoder, train_dataloaders=train_loader)
```



## 训练集、验证集、测试集设置

**`validation_step()` 与 `test_set()` 钩子**

```python
class LitAutoEncoder(pl.LightningModule):

	def validation_step(self, batch, batch_idx): # 实现验证流程(验证流程在训练过程中由lightning框架调用)
        ...
    
    def test_step(self, batch, batch_idx): # 实现测试流程
        ...

# 初始化Trainer
trainer = Trainer()

# 执行test方法
trainer.test(model, dataloaders=DataLoader(test_set)) # 不需要先显式调用eval()函数; 当时如果不是调用test()函数, 仍然需要先显式调用eval()函数
```



## 操作权重

**保存权重**

```python
# 设置保存路径, lightning框架会负责剩下的保存流程, 不需要显式调用
trainer = Trainer(default_root_dir="/path/to/ckpt") # 实际上, default_root_dir设置的也是整个训练项目的默认目录, 也会影响其它的存储路径(e.g. tensorboard log)
```

**载入权重**

```python
# 仅载入权重
model = LitAutoEncoder.load_from_checkpoint("/path/to/ckpt/checkpoint.ckpt")

# 载入权重的同时载入训练参数(e.g. learning rate, epoch, step e.t.c.)
trainer.fit(model, ckpt_path="path/to/your/checkpoint.ckpt")
```



## Debug 调试

- 开启 `fast_dev_run` 让模型**分别执行 5 个 batch 的 train、val、test 操作**

```python
trainer = Trainer(fast_dev_run=True) # 如果直接设置数值则可以控制batch的数量(e.g. fast_dev_run=7)
```

- 设置 `profiler` 后，在 fit() 函数执行结束后，会返回各部分的**执行时间**

  > 应该可以搭配 `fast_dev_run` 来找出模型的瓶颈

```python
trainer = Trainer(profiler="simple")
```

- 在实例化 `Trainer` 时，**默认会使用 tensorboard 使用的日志格式**进行记录；默认保存在 `lightning_logs` 目录下

  通过调用 **`log()` 函数** (`pl.LightningModule` 已经实现好的方法) 向日志中进行写入

  > 可以理解为对 tensorboard 里的 `add_scale()` 做的封装
  >
  > 如果想要记录图像等其它数据，`log()` 并不支持

```python
def training_step(self, batch, batch_idx):
    loss = ...
    self.log("loss", loss)
```

