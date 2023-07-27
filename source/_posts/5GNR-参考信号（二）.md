---
title: NR参考信号学习（二）
mathjax: true
tags:
- 5GNR
categories:
- 学习记录
- 5G
---

<p align="center">
    <img src="https://s2.loli.net/2023/04/17/NOadov1XT46nl3r.png" alt="logo" style="zoom:90%" />
</p>

首先想先介绍一下ZC序列以及它的一些特性，然后接下来会总结一下SRS到物理资源的映射过程和方式。

<!-- more -->

## 为什么是ZC序列

### ZC序列的特性

1962 年Zadoff提出的循环自相关为零的多相编码是一种用于无线通信中的序列编码技术。它的特点是具有循环自相关为零的性质，可以有效地降低多径干扰和频偏影响，提高系统的抗干扰性能和频谱利用效率,此外，它还具有码长可变、相位连续可调、码间互不干扰等优点。然而，Zadoff序列的长度必须是质数，因此其码长受到限制。此外，由于其相位连续可调的特性，Zadoff序列在频域上存在较大的旁瓣，可能会对其他用户的信号造成干扰。因此，在实际应用中需要进行合理的设计和选择。

> **自相关：**指一个信号与其自身在不同时间点上的相似程度。自相关函数（Autocorrelation Function，ACF）描述了信号在不同时间点上的自相关性，即信号在不同时间点上的相似程度。自相关函数的峰值表示信号的周期性，而自相关函数的宽度表示信号的带宽。
>
> **互相关：**指两个信号之间的相似程度。互相关函数（Cross-correlation Function，CCF）描述了两个信号之间的相似程度，即它们在不同时间点上的相关性。互相关函数的峰值表示两个信号之间的时延，而互相关函数的宽度表示两个信号之间的带宽。

Zadoff-Chu 序列一个关键特点是**经过离散傅里叶变换，生成的新序列依然是 Zadoff-Chu 序列**。Zadoff-Chu 序列的另一个特点是序列**在时域和频域的幅度都是恒定的**。时域信号的幅度恒定有助于提高功放效率。频域信号的幅度恒定意味着序列经过任意非零循环移位与原序列零相关。这就是说，同一个 Zadoff-Chu 序列在时域上经过不同的循环移位所产生的两个序列信号之间是正交的。时域的循环移位等效于在频域上进行连续的相位旋转。

<img src="https://s2.loli.net/2023/04/17/NOadov1XT46nl3r.png" alt="logo" style="zoom:90%" />

Zadoff-Chu序列在随机接入过程中的preamble也有使用，我们可以看一下这种序列是如何使用的：

![image-20230417105349269](https://s2.loli.net/2023/04/21/hjO12pRJPFfxNo9.png)

### ZC序列的生成

Zadoff-Chu序列生成方法如下： 假设Zadoff-Chu序列的长度为N，相位因子（更广泛的说法是ZC序列的根Root，也用$R$表示）为$p$，则第$k$个元素可以表示为： $$ x_k = e^{-j\pi pk(k+1)/N} $$ 其中，$k$的取值范围为$0$到$N-1$。 Zadoff-Chu序列的生成过程可以通过以下步骤实现：

1. 选择一个质数N和一个整数p，满足N和p互质。 
2.  生成一个长度为N的序列，其中第k个元素为： $$ x_k = e^{-j\pi pk(k+1)/N} $$ 
3.  如果需要将序列进行循环移位，则将序列循环移位k个位置，其中k为任意整数。 
4.  如果需要将序列进行相位旋转，则将序列中的每个元素乘以一个相位因子$e^{j\theta}$，其中$\theta$为任意实数。 通过以上步骤，可以生成一个循环自相关为零的Zadoff-Chu序列，用于无线通信中的序列编码和解码。

虽然 Zadoff-Chu 序列具有以上优点，但它**并不能直接在 LTE/NR 系统中被使用**。一个原因是 **SRS 序列长度并不是质数**（见下文物理资源映射部分），另一个原因是 Zadoff-Chu**序列长度较短时，没有足够的可用序列**。[Marshall](https://marshallcomm.cn/2020/11/21/r15_sounding_reference_signal/#用于下行-csi-获取的-ue-探测过程)在博客中说道：

> 原始的 Zadoff-Chu 序列可以为任意长度，但人们往往关注ZC序列长度 $N_{ZC}$ 为质数的情况，因为质数长度会拥有最多可用的 Zadoff-Chu 序列。也就是说，在 Zadoff-Chu序列长度  $N_{ZC}$ 为质数的情况下，可以找到  $N_{ZC}-1$个不同的 Zadoff-Chu序列。因此 SRS 序列采用 Zadoff-Chu序列循环扩展，从长度  $N_{ZC}$ 扩展到长度  $M_{ZC}$。SRS 序列是频域序列，由于是在频域上扩展生成，所以依然保持了完美的循环自相关特性，但在时域上会破坏 Zadoff-Chu序列原有的幅度恒定特性，因此时域幅度上会产生波动。

对于长度超过 36 的 SRS 序列，都会使用扩展 Zadoff-Chu 序列。对于长度小于 36 的 SRS 序列，NR 标准通过计算机穷举法，找到了一组合适的序列，这些频域上平坦的序列具有良好的时域包络特性。长度小于 36 的 SRS 序列不使用 Zadoff-Chu 序列，主要原因是找不到足够的可用 Zadoff-Chu 序列。

ZC 序列具有零相关性是指，同一个基序列的不同循环移位得到任意两个序列之间的相关系数为零，另一方面，理论上两个不同的基序列（p不同，无循环移位）的互相关系数为 1/sqrt(N)。

### ZC序列性质的简单研究（TODO）

matlab下进行数值计算：

```matlab
N  = 63;
M1 = 61;
M2 = 59;
m  = 5;
 
% 1. Zadoff-Chu
ak1 = zeros(1,N);
bk1 = zeros(1,N);
for k = 0:N-1
    ak1(k+1) = exp(1j * (M1 * pi * k * (k+1) / N));
    bk1(k+1) = exp(1j * (M2 * pi * k * (k+1) / N));
end
 
% 2. cyclic shift
ak2 = [ak1(1, m+1:end), ak1(1, 1:m)];
 
% 3. autocorrelation
autoCorr = abs(sum(ak1 .* conj(ak2)) / N)
 
% 4. cross correlation
crossCorr = abs(sum(ak1 .* conj(bk1)) / N)

```

输出结果为：

```matlab
autoCorr =

   4.4623e-14

crossCorr =

    0.1260
    
```

## SRS到物理资源的映射

### 频域资源

对应 SRS 资源的每个 OFDM 符号$l'$和天线端口$p_i$，需要先将 SRS 序列 $r^{(p_i)}(n,l')$ 乘以幅度扩展因子 $\beta_{SRS}$ 以符合 TS 38.213 规定的传输功率。然后对每个天线端口$p_i$，从 $r^{(p_i)}(0,l')$开始在一个时隙内根据下式映射到资源元素**（Resource Element）**$(k,l)$：

![img](https://s2.loli.net/2023/04/17/W1l3QhAcFPLIUBr.png)

而SRS信号的长度是通过下式给定的：
$$
M_{sc,b}^{SRS}=m_{SRS,b}N_{sc}^{RB}/(K_{TC}P_F)
$$
其中$m_{SRS,b}$是通过下表中通过$b=B_{SRS}$对应得到的：

![image-20230417140047252](https://s2.loli.net/2023/04/17/yQ8qBGrvwMfFS3R.png)

<center><small>SRS信号带宽配置表</small></center>

$B_{SRS}\in\{0,1,2,3\}$是由高层参数`freqHopping` 中的字段` b-SRS `给定，$C_{SRS}\in\{0,1,2,\dots,63\}$由高层参数`freqHopping` 中的字段` c-SRS `给定查表时通过 $C_{SRS}$选中某一行，再通过 $B_{SRS}$ 选中某一列，即可最终确定 SRS 的带宽信息。NR 系统支持 64 种 SRS 带宽配置方式，一个 SRS 资源可配置的最小带宽为 4 个 RB，最大带宽为 272 个RB；$K_{TC}$是之前说过的传输梳数，$P_F$是频率因子，默认为1，如果配置了高层参数的`FreqScalingFactor`后可以配置为$\{2,4\}$。当其被配置后，UE则会收到长度是6的倍数的SRS序列。

确认完带宽信息后，还需要确定 SRS 的频域起始位置。起始位置$k_{0}^{(p_i)}$定义为：

<img src="https://s2.loli.net/2023/04/17/18pw9KVTLSNEAYa.png" alt="image-20230417163847159" style="zoom:80%;" />

其中：

![image-20230417172651677](https://s2.loli.net/2023/04/17/SHvq8xluTcgWarM.png)

且：

- $k_F\in \{0,1,\dots,P_F -1\}$由高层参数`StartRBIndex`配置，未配置时设为$k_F=0$

- $k_{hop}$通过下表以及如下式子确定：

  ![image-20230417164237549](https://s2.loli.net/2023/04/17/fi3uhZEL4KPRxvD.png)

  ![image-20230417164216348](https://s2.loli.net/2023/04/17/D6tSC1RFjf5oZM4.png)

  

接下来则是和SRS跳频相关的内容。频域移位值$n_{shift}$用来调整 SRS 资源和 CRB 的 4 的倍数网格对齐，它包含在高层参数`freqDomainShift` 中。如果 $N_{BWP}^{start}\leq n_{shift}$，则 $k_{0}^{(pi)}=0$的参考点是 CRB 0 的子载波 0；如果$N_{BWP}^{start}> n_{shift}$，则$k_{0}^{(pi)}=0$的参考点是 BWP 的最低编号的子载波。SRS 是否跳频由高层参数` freqHopping `中的字段` b-hop` 决定。$b_{hop}$ 取值范围为0~3。

如果$b_{hop}>B_{SRS}$，则跳频关闭，频域位置索引$n_b$保持为常量（除非被重新配置）。此时，对于SRS资源中的全部$N_{symb}^{SRS}$个OFDM符号，$n_b$的定义为：
$$
n_b=[\frac{4n_{}RRC}{m_{SRS,b}}]\mod N_b
$$
其中$n_{RRC}$由高层参数`freqDomainPosition`给定。$m_{SRS,b}$和$N_b$的值通过给定的$C_{SRS}$和$b=B_{SRS}$查`SRS信号带宽配置表`得到。

如果$b_{hop}<B_{SRS}$，则SRS跳频打开，频域位置索引$n_b$由下式定义：
$$
\begin{aligned}
n_b=\begin{cases}
[\frac{4n_{}RRC}{m_{SRS,b}}]\mod N_b\qquad &b\leq b_{hop}\\
(F_b(n_{SRS})+[[\frac{4n_{}RRC}{m_{SRS,b}}]\mod N_b])\mod N_b \qquad &otherwise
\end{cases}
\end{aligned}
$$
其中$N_b$也是通过查询`SRS信号带宽配置表`得到。$F_b(n_{SRS})$则是通过下式得到：
$$
\begin{aligned}
F_b(n_{SRS})=\begin{cases}
(N_b/2)[\frac{n_{SRS}\mod \prod_{b'=b_{hop}}^{b}N_{b'}}{\prod_{b'=b_{hop}}^{b-1}N_{b'}}]+[\frac{n_{SRS}\mod \prod_{b'=b_{hop}}^{b}N_{b'}}{2\prod_{b'=b_{hop}}^{b-1}}]\quad &\mathrm{if} \, N_b\,\mathrm{even}\\
[N_b/2][n_{SRS}/\prod_{b'=b_{hop}}^{b-1}N_{b'}]\quad &\mathrm{if} \, N_b\,\mathrm{odd}\\
\end{cases}
\end{aligned}
$$
此时，不论$N_b$取值如何，$N_{b_{hop}}=1$。$n_{SRS}$是传输SRS信号的次数。如果高层参数 `resourceType` 被配置为 “aperiodic”，则在时隙内发送 $N_{symb}^{SRS}$ 个符号的 SRS 资源为$n_{SRS}=⌊\frac{l'}{n}⌋$。重复因子 $R\leq N_{symb}^{SRS}$ 由高层参数` resourceMapping `中的字段` repetitionFactor` 给定，如果没有配置的话则$R=N_{symb}^{SRS}$。

如果在高层参数中配置了`resourceType`为"periodic"或"semi-persistent"的话，则SRS计数$n_{SRS}$通过下式得到：
$$
n_{SRS}=
\begin{bmatrix}
N_{slot}^{frame,\mu}n_f + n_{s,f}^{\mu}-T_{offset}\\
T_{SRS}
\end{bmatrix}\cdot
\begin{bmatrix}
N_{symb}^{SRS}\\
R
\end{bmatrix}+
\begin{bmatrix}
l'\\
R
\end{bmatrix}
$$

### 时域资源

以时隙表示的 SRS 周期$T_{SRS}$和时隙偏移量$T_{offset}$根据高层参数 `periodicityAndOffset-p` 或 `periodicityAndOffset-sp` 确定。如果` resourceType = “periodic”`，则对应 `periodicityAndOffset-p`；如果` resourceType = “semi-persistent”`，则对应 `periodicityAndOffset-sp`。在配置的SRS资源中可用于 SRS 传输的候选时隙需要满足:
$$
(N_{slot}^{frame,\mu}n_f+n_{s,f}^{\mu}-T_{offset})\mod T_{SRS}=0
$$

