---
title: LS信道估计与OAI代码工程优化
tags: 
- 信道估计
- OAI
---

所谓信道估计，就是从接收数据中将假定的某个信道模型的模型参数出来的过程。如果信道是线性的话，那么信道估计就是对系统冲激响应进行估计。需强调的是**信道估计是信道对输入信号影响的一种数学表示**，而“好”的信道估计则是**使得某种估计误差最小化的估计算法**。

<!-- more -->

## 信道估计分类

### 基于训练序列的估计

这种方法的原理就是在发射数据符号外，还需要发射前导（Preamble）或导频（pilot）信号；如最小二乘LS、最小均方误差MMSE等基于训练序列的信道估计算法被广泛应用于信道估计；

- 优点：训练符号能够提供较好的性能；

- 缺点：由于除了发射数据符号外，还需要发射前导或导频信号，由此训练序列过长会降低频谱效率；

### 盲/半盲信道估计

从接收信号的结构和统计信息中获取信道状态信息CSI (channel state information)或均衡器系数，无需或很少训练序列。

- 优点：资源的开销较少；

- 缺点：性能比基于训练序列的信道估计算法差；

## LS信道估计

“LS”是最小二乘(Least Square)的意思，从字面意思不难看出，它是通过最小化发射和接收信号的距离平方的一种信道估计方法。这种信道估计的模型构建如下：
$$
\mathrm{Y=HX}
$$
我们现在已知的数据有发射的 $X$和 接收到的$Y$，我们需要得到 $H$的估计 $\hat H$，根据上面的优化目标不难写出需要最小化的损失函数：
$$
\begin{aligned}
J(\hat H) =&\mathrm{||Y-X \hat H||^2}\\
=&\mathrm{(Y-X\hat H)^{H}(Y-X\hat H)}\\
=&\mathrm{Y^H Y-Y^H X\hat H - \hat H^H X^H Y+\hat H^H X^H X\hat H }
\end{aligned}
$$
因为要通过改变$\hat H$使损失最小，故令$J(\hat H)$对$\hat H$的偏导为0，得：
$$
\frac{dJ(\hat H)}{d\hat H}=0
$$
此时可得：
$$
\mathrm{\hat H=(X^H X)^{-1}X^H Y=X^{-1}Y}
$$

## OAI的LS实现

在上面我们已经知道LS估计的核心公式了。在OFDM系统中，LS估计是以子载波为单位进行估计的。设 $N$为子载波个数，则有：
$$
\mathrm{\hat H_{LS}}[k]=\mathrm{\frac{Y[k]}{X[k]}},k=0,1,2,\dots,N-1
$$
具体点的，如果信号的形式为$a+bi$，那么每一个子载波的估计结果就为：
$$
\hat H_{LS}[k]=\frac{a_r+b_ri}{a_g+b_gi}\\
=\frac{(a_r+b_ri)(a_g-b_gi)}{a_g^2+b_g^2}\\
=\frac{a_ra_g+b_rb_g}{a_g^2+b_g^2}+i\frac{a_gb_r-a_rb_g}{a_g^2+b_g^2}
$$
OK接下来我们以SRS信道估计部分的代码作为例子看看OAI是怎么实现的。在`openair1/PHY/NR_ESTIMATION/nr_ul_channel_estimation.c`中的`nr_srs_channel_estimation()`函数：

```C
// We know that nr_srs_info->srs_generated_signal_bits bits are enough to represent the generated_real and generated_imag.
            // So we only need a nr_srs_info->srs_generated_signal_bits shift to ensure that the result fits into 16 bits.
            ls_estimated[0] += (int16_t)(((int32_t)generated_real*received_real + (int32_t)generated_imag*received_imag)>>nr_srs_info->srs_generated_signal_bits);
            ls_estimated[1] += (int16_t)(((int32_t)generated_real*received_imag - (int32_t)generated_imag*received_real)>>nr_srs_info->srs_generated_signal_bits);
```

注意到OAI实现的过程中没有用除法，而是进行了位移操作。OAI得到的数据的实部和虚部都是整数，因此支持这样的位移操作，即将分母用右移操作替代了。另外一个重要原因是SRS序列是恒模的因此分母部分是恒定的，因此这是个非常巧妙的工程实现。

## LS估计的性能

LS信道估计的均方误差为：
$$
\begin{aligned}
MSE_LS=&E\mathrm{[(H-\hat H_{LS})^H(H-\hat H_{LS})]}\\
=&E\mathrm{[(H-\hat X^{-1}Y)^H(H-\hat X^{-1}Y)]}\\
=&E\mathrm{[(X^{-1}Z)^H(X^{-1}Z)]}\\
=&E\mathrm{[Z^H(XX^H)^{-1}Z]}\\
=&\frac{\sigma_z^2}{\sigma_x^2}
\end{aligned}
$$
上式中，$\mathrm Z$为噪声向量，$\mathrm{Z}=[Z[0],Z[1],\dots,Z[N-1]]^T$满足 $E[Z[k]]=0,Var[Z[k]]=\sigma_z^2$。可以观察到LS估计算法的MSE和信噪比成反比，这意味着LS估计增强了噪声，在信道深度衰落时更严重。然而由于LS信道估计算法的简单性而广泛地应用于信道估计中,，比如在OFDM系统中和单载波频域均衡SC-FDE系统中会经常用到LS信道估计算法。