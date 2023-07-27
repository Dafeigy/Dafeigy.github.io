---
title: LSTM初探
mathjax: true
tags:
- Pytorch
- 计算机视觉
- 时序分析
- LSTM
categories:
- 深度学习
---

<img src="https://pic4.zhimg.com/v2-03c41f0aaee75d920c1ba7bd756ae207.png" alt="preview" style="zoom:50%;" />

长短期记忆（Long Short Term Memory）在一般的循环神经网络的基础上补充完善了对信息的记忆机制，对时序信息有更好的处理能力，不但在NLP任务中大放光彩，甚至在一些视频、医学重建任务中也能发挥作用。下面则是我对LSTM的一个学习记录。

<!-- more -->

## RNN的简单回顾

RNN的一般结构如下：

<img src="https://s2.loli.net/2022/10/14/BAHdpuG2mlFzkqc.jpg" alt="img" style="zoom: 50%;" />

其中黑色方框的$x^t$为输入，$h^{t-1}$为上一次输入时隐藏层的输出信息，也可以理解为被高度抽象化的特征，$y^{t}$为目标输出，以及当前的隐藏层状态$h^{t}$输出。可以看到不管在哪个时刻的输入，都会有上一个输入产生的输出的影响$h^{t-1}$，理论上RNN是可以保留从第一次输入到当前输入这段时期的所有序列信息（对于第一次输入，我们可以将$h^{t_0}$设为全零），那么我们为什么还需要引入长短期记忆呢？



答案很简单：因为不是过往所有的信息都是我们需要关心的。好比一个对话机器人bot，他不会去对用户一个月前的输入产生兴趣并揪着这个问题不断地询问用户。因此，我们需要一种“遗忘”的机制，以更加真实地去贴近人类处理时序信息地方式。



另一个更细节的问题就是：输入是随机的，但是随机的信息也会作为一个输入存留在RNN里，它带来的问题不只是简单的像通信领域用滤波器滤掉噪声这么单纯，它还会带来梯度爆炸或者梯度消失的问题。这两个问题在上了很深的网络上无疑是致命的，所以我们需要对过往信息的处理，而处理的最好方式就是“遗忘”。

## LSTM的引入

LSTM的基本结构如图所示，可以看出相比于一般的RNN，LSTM的输入中多了一个$c^{t-1}$变成了三个。

<img src="https://s2.loli.net/2022/10/14/9amJuXiEDCSTdPb.jpg" alt="img" style="zoom:50%;" />

新加进来的这个的状态$c^t$叫做`cell state`，原来的$h^t$我们称为叫做`hidden state`。其中对于传递下去的$c^t$改变得很慢，通常输出的$c^t$是上一个状态传过来的 $c^{t-1}$加上一些数值，而$h^t$在不同节点下通常有很大区别。

### LSTM的层次结构与功能关系

<img src="https://s2.loli.net/2022/10/14/YXLjJ1dcvhRbeAV.jpg" alt="img" style="zoom:50%;" />

这张图比上一个图看上去更复杂点，但是其实它的本质还是一般的神经网络运算过程，和普通的MLP的单一$\mathrm y=\mathrm W \mathrm x+\mathrm b$区别在于多了几个需要运算的状态。我们先看看中间的$z$状态是怎么来的：

<center><img src = 'https://s2.loli.net/2022/10/14/X9bugtqRQGONCe5.webp' style = "zoom:35%"><img src = 'https://s2.loli.net/2022/10/14/NOZ3IwRkCmcbpVY.jpg' style = "zoom:35%"></center>

可以看到这四个玩意都是通过拼接两个输入状态然后与矩阵相乘后通过激活函数得到的。其中$\sigma$代表的是sigmoid函数（映射到(0,1)区间），把输入映射到`(0，1)`区间是一种类似门控的作用：
$$
S(x)=\frac{1}{1+e^{-x}}
$$


tanh代表的是双曲正切函数（映射到(1，1)区间），映射到`(-1,1)`理解成将其作为输入数据，而不是门控信息。：
$$
tanh(x) = \frac{e^{x}-e^{-x}}{e^{x}+e^{-x}}
$$
那么LSTM的工作如下所示：
$$
c^t = z^f\odot c^{t-1} + z^i\odot z\\
h^t = z^o\odot tanh(c^t)\\
y^t =\sigma(W'h^t)
$$

### LSTM运行原理

LSTM内部有三个阶段：遗忘段、选择记忆段以及输出段。

* 这个阶段主要是对上一个节点传进来的输入进行**选择性**忘记。简单来说就是会 “忘记不重要的，记住重要的”。

  具体来说是通过计算得到的 $z^f$（f表示forget）来作为忘记门控，来控制上一个状态的$c^{t-1}$哪些需要留哪些需要忘。

* 选择记忆阶段：这个阶段将这个阶段的输入有选择性地进行“记忆”。主要是会对输入$x^t$进行选择记忆。哪些重要则着重记录下来，哪些不重要，则少记一些。当前的输入内容由前面计算得到的$z$表示。而选择的门控信号则是由$z^i$（i代表information）来进行控制。把上述两个阶段的结果相加即可得到输出阶段需要的$c^t$，也就是上面的第一个公式。

* 这个阶段将决定哪些将会被当成当前状态的输出。主要是通过$z^o$来进行控制的。并且还对上一阶段得到的$c^0$进行了放缩（通过一个tanh激活函数进行变化）。



可以看到，在输出阶段$z^o$起到门限控制作用，对进行了映射后的融合信息$h^t$进行控制输出。



## Pytorch中的LSTM使用

[官方文档](https://pytorch.org/docs/stable/generated/torch.nn.LSTM.html)可以看到详细的参数设定。我们一般关注以下几个参数：

* `input_size`：输入数据的特征维数。
* `hidden_size`：LSTM中隐层的维度。
* `num_layers`：循环神经网络的层数。

把网络看成一个黑箱，我们在用是肯定是输入一个向量，然后网络处理后输出一个向量，所以我们必须要告诉网络输入的向量是多少维，输出的为多少维，因此前两个参数就决定了输入和输出向量的维度。当然，`hidden_size`只是指定从LSTM输出的向量的维度，并不是最后的维度，因为LSTM层之后可能还会接其他层，如全连接层（FC），因此`hidden_siz`e对应的维度也就是FC层的输入维度。第三个参数`num_layers`为隐藏层的层数，这个比较好理解，官方的例程里面建议一般设置为1或者2.

<img src="https://s2.loli.net/2022/10/14/vuUZE39CXFdTVxi.png" alt="image-20221014111604955" style="zoom: 50%;" />

下面给出一个应用的案例：

```python
import torch.nn as nn
import torch

class lstm(nn.Module):
    def __init__(self):
        super(lstm, self).__init__()
        # 定义LSTM
        self.rnn = nn.LSTM(input_size, hidden_size, hidden_num_layers)
        # 定义回归层网络，输入的特征维度等于LSTM的输出，输出维度为1
        self.reg = nn.Sequential(
            nn.Linear(hidden_size, 1)
        )

    def forward(self, x):
        x, (ht,ct) = self.rnn(x)
        seq_len, batch_size, hidden_size= x.shape
        x = x.view(-1, hidden_size)
        x = self.reg(x)
        x = x.view(seq_len, batch_size, -1)
        return x

```



```python
output,(h_n,c_n) = lstm (x, [ht_1, ct_1])
```

