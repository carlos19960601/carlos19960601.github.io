---
title: '30Days机器学习笔记'
date: 2025-03-20T11:27:36+08:00
draft: true
categories:
  - 机器学习
---

# 使用Pytorch实现一个简单的全连接神经网络(MLP)

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets
from torchvision import transforms
import matplotlib.pyplot as plt

# 1. 定义MLP模型类
class SimpleMLP(nn.Module):
    def __init__(self, input_size=28*28, hidden_size=512, num_classes=10):
        super().__init__()
        self.flatten = nn.Flatten()
        # 定义网络层
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(input_size, hidden_size),
            nn.ReLU(),
            nn.Linear(hidden_size, hidden_size),
            nn.ReLU(),
            nn.Linear(hidden_size, num_classes),
        )

    def forward(self, x):
        x = self.flatten(x)
        logits = self.linear_relu_stack(x)
        return logits

# 2. 数据加载与预处理

# 超参数
learning_rate = 1e-3
batch_size = 64
epochs = 5

transform = transforms.Compose([transforms.ToTensor()])
training_data = datasets.MNIST(root="data", train=True, download=True, transform=transform)
test_data = datasets.MNIST(root="data", train=False, download=True, transform=transform)

train_dataloader = DataLoader(training_data, batch_size=batch_size)
test_dataloader = DataLoader(test_data, batch_size=batch_size)


# 3. 初始化模型、损失函数和优化器
model = SimpleMLP()
loss_fn = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=learning_rate)

# 4. 训练模型
def train_loop(dataloader, model, loss_fn, optimizer):
  size = len(dataloader.dataset)
  # Set the model to training mode - important for batch normalization and dropout layers
  # Unnecessary in this situation but added for best practices
  model.train()
  for batch, (images, labels) in enumerate(dataloader):
    # Compute prediction and loss
    pred = model(images)
    loss = loss_fn(pred, labels)

    # 反向传播
    loss.backward()
    optimizer.step()
    # 调用 optimizer.zero_grad() 来重置模型参数的梯度。梯度默认是累加的；为了防止重复计算，我们在每次迭代时显式地将它们置为零。
    optimizer.zero_grad()

    if batch % 100 == 0:
      loss, current = loss.item(), batch * batch_size + len(images)
      print(f"loss: {loss:>7f}  [{current:>5d}/{size:>5d}]")


def test_loop(dataloader, model, loss_fn):
  # Set the model to evaluation mode - important for batch normalization and dropout layers
  # Unnecessary in this situation but added for best practices
  model.eval()
  size = len(dataloader.dataset)
  num_batches = len(dataloader)
  test_loss, correct = 0, 0

  # Evaluating the model with torch.no_grad() ensures that no gradients are computed during test mode
  # also serves to reduce unnecessary gradient computations and memory usage for tensors with requires_grad=True
  with torch.no_grad():
    for images, labels in dataloader:
      pred = model(images)
      test_loss += loss_fn(pred, labels).item()
      correct += (pred.argmax(1) == labels).type(torch.float).sum().item()

  test_loss /= num_batches
  correct /= size

  print(f"Test Error: \n Accuracy: {(100*correct):>0.1f}%, Avg loss: {test_loss:>8f} \n")

for t in range(epochs):
  print(f"Epoch {t+1}\n-------------------------------")
  train_loop(train_dataloader, model, loss_fn, optimizer)
  test_loop(test_dataloader, model, loss_fn)

print("Done!")
```


# 理解激活函数与损失函数

## 激活函数

### 定义

激活函数是神经网络中每个神经元上的非线性函数，将输入信号转换为输出信号。

### 核心作用

* 引入非线性：使神经网络能够拟合复杂非线性关系（否则多层线性叠加仍为线性模型）。
* 控制输出范围：将神经元的输出限制在特定区间（如0-1、-1-1）。
* 决定神经元是否“激活”：根据输入值决定是否传递信号到下一层。

### 常见激活函数

#### Sigmoid 函数

