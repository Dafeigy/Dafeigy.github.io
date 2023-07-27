---
title: ISAC学习记录
mathjax: true
tags:
- ISAC
categories:
- ISAC
---

<p align="center">
    <img src="https://s2.loli.net/2023/03/07/eON3wACGtfIKyUQ.png" alt="ISAC" />
</p>
<h1 align="center">ISAC 学习笔记</h1>
<p align="center">
    <em>待更新</em>
</p>


ISAC笔记

<!-- more -->

## 引言部分

下一代的通信（B5G、6G）技术对无线连接能力和准确、高鲁棒的感知能力提出了高要求，这些技术都有一个共同的主题——感知。感知是嵌入在网络中的一种技术，或者说是一种能力，是用来给未来的潜在应用赋能或收集数据的。`S&C`即*Sensing and Communication*的功能集合需要在未来网络进行联合的设计，即*Integrated Sensing and Communication*s（`ISAC`）。

感知和通信对应着两个方面。感知从含噪声的观测中收取并提炼信息，通信则是关注使用特定的信号传输信息然后将信息在接收端恢复。二者的统一是ISAC的终极目的，实现提升频谱或能效同时减少硬件与信令的消耗。

ISAC在之前有一堆乱七八糟的名字——RC、JCR、JRC、DFRC，主要集中在雷达感知领域，曾经也是ISAC的主流方向。雷达传感的联合感知与通信将是ISAC的核心。ISAC的发展历程如下：

![image-20230306092652694](https://s2.loli.net/2023/03/06/2HAE5Oh3RWJwK4i.png)

感知在未来的无线网络中是一种原生的能力而不是一种额外附加的能力，本身就能为大量用户提供服务。联合增益与整体增益是ISAC发展的两个可能方向。具体的参照下图：

![image-20230306095419941](https://s2.loli.net/2023/03/06/s4aWZFCxYT13JEA.png)

## 工业进展与应用

<img src="https://s2.loli.net/2023/03/08/UI1dKJsEhAFm4CV.png" alt="图片" style="zoom:80%;" />

感知服务：

* 增强定位于跟踪：全球卫星导航下的卫星定位精度——米级。5G NR的最高定位标准是垂直/水平精度分别为0.2m/1 m， 难以满足未来的工业物联网需求。机器人操作精度5mm，室内人物动作识别精度1cm，距离多普勒处理。
* 区域成像：射频城乡的侵扰性更低，米级别的分辨率。毫米波以及mMIMO技术可以放到BS上面，RAN可以作为一个分布式的MIMO雷达，最终让UE感知到周边的环境。
* 无人机监测于管理：某些无人机体积小，热成像或摄像头难以监测，可能会出现擅入禁区的情况，ISAC的蜂窝网络可以提供监测和管理。

智能家居和室内感知：

* 人类活动识别：疲劳驾驶监测。智能人类居住环境的保障，比如说检测到睡眠状态降低环境中的环境的资源消耗之类的。
* 空间感知计算：室内设备的空间几何关系，对增强虚拟现实应用提供支持。

V2X：

* 车辆运作管理：传统方法基于领导-跟随者的框架，依赖多跳的V2V通信传输每辆车的状态信息，延迟很大。
* SLAM

智能建造和工业IoT：集群导航、节点的调度

遥感和地理科学：战区信息广播、空基雷达监测地球变化，4维度映射，安全相关。

环境监测：传输信号在不同的环境下的性能差异监测环境的变化

人机交互：虚拟键盘，谷歌Soli项目，无接触的人机交互，微多普勒识别的准确性。

总结如下：

![image-20230307104531125](https://s2.loli.net/2023/03/07/oJCzSXxOI6FhDWP.png)

## 性能评估

从信息论的角度来看，感知的性能并不像通信的性能那样明确，信息论层面的感知性能分析还鲜有报道。因此，需要新的分析技术来评价通感一体系统，揭示两者在信息论层面上的基本理论极限。

### 感知的评估

* 检测：MMSE
* 估计：CRLB
* 认知：

### 通信的评估

* 有效性：传输速率
* 可靠性：误码率

## 信息论的权衡

和ISAC相关的重要理论桥梁是最小均方误差和互信息的关系，它连接了通信衡量指标的输入输出的互信息和检测指标：
$$
\frac{d}{d\,\mathrm{snr}}I(\mathrm{snr})=\frac{1}{2}MMSE(\mathrm{snr})
$$
这里就有一个平衡的关系：一个高斯输入可以使得式子左边变大，但是同样会使得检测部分的错误上升，二者构成一个平衡。另一个容量-衰减的权衡是在一个经典场景下构建的：

<img src="https://s2.loli.net/2023/03/07/B8fH6g45hv7oAiJ.png" alt="image-20230307230625015" style="zoom:67%;" />

发送端想传输纯信息（用$W\in\{1,2,\dotsm2^{nR}\}$表示信息的索引）和信道状态$S$的描述$\hat S$到接收端。在给定$W$和$S$的情况下，发射端以速率$R$传输一个编码$X(W,S)$到接收端。接收端的观测如下：
$$
Y\sim\prod_{i=1}^{n}p(y_i|{x_i,s_i})
$$
接收端随后从$Y$中解码信息$\hat W(Y)\in\{1,2,\dots,2^{nR}\}$，同时对状态进行估计为$\hat S(Y)$。译码的错误概率和状态估计的误差定义如下：
$$
\begin{aligned}
\mathcal P_e^{(n)}&=\frac{1}{2^{nR}}\sum_{i=1}^{2^{nR}}\Pr\{\hat W\neq i|W=i\}\\
D&=\mathbb E\{d(S,\hat S\}
\end{aligned}
$$
其中$d(S,\hat S)$是$S$和$\hat S$之间的失真度量。一个率失真对$(R,D)$如果存在这样一个序列$(2^{nR},n)$对$X(W,S)$进行编码，那么我们说它是可达的：
$$
\mathbb E{\{d(S,\hat S)\}}\leq D,\mathcal P_{e}^{(n)}\rightarrow0,n\rightarrow \infty
$$
如果失真函数选择为状态估计误差的平方，则MSE的估计可以表述为：
$$
\mathbb E{\{d(S,\hat S)\}}=\frac{1}{n}\mathbb E{||S-\hat S(Y)||^2}
$$
接下来还需要考虑状态独立的高斯信道$Y=X(W,S)+S+N$，其中$S_i\sim\mathcal N(0,Q_S)$，$N_i\sim(0,Q_N)$。举个列子，以$\gamma\in[0,1]$$(R,D)$对的帕累托最优边界为：
$$
\begin{aligned}
(R,D)=&(\frac{1}{2}\log(1+\frac{\gamma P}{Q_N}),\\
&Q_S\frac{(\gamma P + Q_N)}{(\sqrt{Q_S}+\sqrt{(1-\gamma)P})^2 + \gamma P+Q_N}
)
\end{aligned}
$$
其中的$P\geq\frac{1}{n}\mathbb E{\{\sum_{i=1}^{n}X_i^2(W,S)\}}$，表示为发射信号$X$的功率约束。可以看到传输的功率分割为两部分：$\gamma P$和$(1-\gamma)P$，分别对应着传输纯信息和状态信息，达到一种权衡。功率资源由纯信息和信道状态信息共享，这种方法也会在PHY的tradeoff中再次体现。

需要指出，上述的率失真的权衡不能捕捉到经典的ISAC场景中的某些特征，比如说某个目标的反射波的估计。在单静态雷达中，发射端是不能预先知道目标的信道状态的。另外，其也不需要感知目标。Kobayashi 和 Caire提出了将目标的返回信号建模为一个延迟回馈信道，如下图所示：

<img src="https://s2.loli.net/2023/03/08/B2bRJCX5GQ6947n.png" alt="image-20230308092411212" style="zoom: 67%;" />



信道的状态在接收端是可知的，但是发射端是不知道的。在每次传输的时候，传输端利用一个估计器从延迟反馈的输出$Z\in\mathcal Z$对信道状态进行估计得到$\hat S$。选择一条信息$W$后，传输端使用一个基于$W$和$\hat S$的编码器发送一个符号$X\in\mathcal X$。信道的输出$Y\in\mathcal Y$会传输到接收端，随后返回一个信道状态到发送端。联合的分布$SXYZ\hat S$可以表述为：
$$
\begin{aligned}
&P_{SXYZ\hat S}(s,x,y,z,\hat s)\\
&\quad =P_S(s)P_X(x)P_{YZ|XS}(y,z|x,s)P_{\hat S|XZ}(\hat s|x,z)
\end{aligned}
$$
给定了一个失真$D$后，信道的容量$C$和失真的权衡表示为：
$$
C(D)=\max_{P_X:\frac{1}{n}\sum_{i=1}^{n}\mathbb E\{X_i^2\leq P\}}I(X;Y|S)\\
s.t.\,\mathbb E(d(S,\hat S))\leq D
$$
其中$P_X$代表了信道输入$X$的分布。上述的问题一般来说是一个凸优化问题，可以通过Blahut-Arimoto算法求解。

## 物理层的权衡

在通信感知一体化场景中，感知和通信通常需要集成到同一套硬件平台并且共享相同的无线资源，并且在给定的资源配置中，感知性能和通信性能往往是相互对立、此消彼长的关系。物理层的权衡其实可以原生的通信/感知的评估指标进行表述，就在上文提到的信息论的框架下进行。当然也可以对感知提出新的评估指标，因为上文也说到了感知这方面的评估基础还比较空缺。

### 原生的S&C评估

物理层的感知性能一般用检测概率和MSE进行衡量。然而即便有MSE的闭式表示也不一定能获取到相关得计算数据，而CRB下界或许是一个可选项。

* 检测vs.通信

  <img src="https://s2.loli.net/2023/03/08/lPIFLciRx3X1uUp.png" alt="image-20230308102531886" style="zoom:33%;" />

  ISAC系统的发送端在总功率限制下发射一个感知波形$s_R(t)$和通信$s_C(t)$波形。两个信号使用的是正交的资源（时频）以防止出现互相干扰。感知接收器SR从直接信道和监督信道中接受$s_R(t)$，期望能在稍后检测到目标的存在与否。另一方面通信用户CU接收到包含有用信息的$s_C(t)$。下面的关键问题就是如何分配功率到感知/通信功能上，建模为如下优化问题：
  $$
  \max_{P_R,P_C}\mathcal P_D \quad s.t.\, R\geq R_{th},P_R+P_C=P_T
  $$
  $\mathcal P_D$代表了雷达的检测概率，$R=\log(1+P_C\gamma_c)$是可达速率，其中$\gamma_c$是含噪信道归一化增益，$R_{th}$是一个阈值。检测问题可以建模为二元假设测试问题：
  $$
  \mathcal H_0:\begin{cases}
  \mathrm y_d=\gamma_d\mathrm G_d\mathrm s_R+\mathrm n_d\\
  \mathrm y_s=\mathrm n_s,
  \end{cases} 
  \qquad
  \mathcal H_1:\begin{cases}
  \mathrm y_d=\gamma_d\mathrm G_d\mathrm s_R+\mathrm n_d\\
  \mathrm y_s=\gamma_s\mathrm G_s\mathrm s_R+\mathrm n_s,
  \end{cases}
  $$
  $\mathrm y_d$和$\mathrm y_s$是从直接信道于监督信道接收到的信号，$\mathrm G_d$和$\mathrm G_s$是一$L\times L$的单一多普勒算子矩阵矩阵，分别对应直接信道和监督信道，$\gamma_d,\gamma_s$分别是两个信道的标量系数，然后$n_d,n_s$是加性高斯白噪声，其方差为$\sigma ^2$。检测通过广义似然比测试进行评估，对应的$\mathcal P_D$可以如下近似：
  $$
  \mathcal P_D \approx Q_1(\sqrt{\frac{2P_R|\gamma_d|^2}{\sigma^2}},\sqrt{2\gamma})
  $$
  其中$Q_1(a,b)$表示了Marcum Q函数(第一类零阶修正贝塞尔函数)，$\gamma$表示了阈值。考虑到功率的约束以及速率的表达式，检测概率可以表示为：
  $$
  \mathcal P_D\approx Q_1(\sqrt{2(P_T-\frac{1}{\gamma_c}(2^{R_{th}}-1)\frac{|\gamma_d|^2}{\sigma^2})},\sqrt{2\gamma})
  $$
  
* 估计vs.通信

  一般的分析不会让资源复用，虽然便于理论分析但是造成了资源的极大浪费。一种多天线的ISAC收发机有$N_t$个发射天线和$N_r\geq N_t$个接收天线，为$K$个单天线用户服务，并同时检测目标，如图所示：

  <img src="https://s2.loli.net/2023/03/08/yVS3hD5H6sGfdlO.png" alt="image-20230308200417927" style="zoom: 50%;" />

  这是种多用户多输入单输出的下行通信系统，也成为monostatic/active MIMO 雷达。使用ISAC波形矩阵$\mathrm X\in\mathbb C^{N_t\times L}$进行传输，受到功率约束$P_T$，基站接收到如下的回传信号：
  $$
  \mathrm{Y}_R=\mathrm{GX}+\mathrm{N}_R
  $$
  其中$\mathrm G \in \mathbb C^{N_r \times N_t}$代表了目标的响应矩阵（TRM），对于不同的目标模型会有不同的形式，同时$N_R$是方差为$\sigma_R^2$的AWGN矩阵。多用户通信的接受的信号模型表示为：
  $$
  \mathrm{Y}_c=\mathrm{HX}+\mathrm{N}_C
  $$
  其中$\mathrm{H}=[\mathrm{h_1,h_2,\dots,h_K}]^H\in\mathcal C^{K\times N_t}$是通信信道矩阵，且基站已知。同时$N_C$是方差为$\sigma_C^2$是AWGN矩阵。ISAC的波形设计可以建模为如下式子的优化问题：
  $$
  \min_\mathrm{X}\, \mathrm{tr}(\mathrm{R_X^{-1}})\,\mathrm{s.t.}||\mathrm{X}||_F^2 \leq LP_T,c_i(\mathrm{X}\odot C_i)
  $$
  
