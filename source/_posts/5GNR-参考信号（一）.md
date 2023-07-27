---
title: NR参考信号学习（一）
mathjax: true
tags:
- 5GNR
categories:
- 学习记录
- 5G
---

<p align="center">
    <img src="https://s2.loli.net/2023/04/16/xIZ3eipjRtKAMkr.png" alt="logo" style="zoom:50%" />
</p>

## NR-物理参考信号

### 参考信号设计原则

参考信号的设计规范：

1. 避免持续发送的周期性信号。所谓持续发送，是指不经系统配置即发送，也无法关闭的信号，例如LTE的CRS。
2. 物理信号占用的时频资源可灵活配置。
3. 支持大规模波束赋形传输。

<!-- more -->

### 现有参考信号

**NR下行**物理信号包括：

* 信道状态信息参考信号（**C**hannel State **I**nformation **R**eference **S**ignal，CSI-RS）
* 解调参考信号（DM-RS）
* 时频跟踪参考信号（Tracking Reference Signal，TRS）
* 相位噪声跟踪参考信号（Phase noise Tracking Reference Signal，PT-RS）
* RRM测量参考信号
* RLM测量参考信号等。

**NR上行**物理信号包括：

* 探测参考信号（Sounding Reference Signal，**SRS**）
* 解调参考信号（DM-RS）
* 相位噪声跟踪参考信号（PT-RS）等

其中，上行DM-RS和PT-RS与下行的设计基本相同。

## SRS信号

SRS 用于上行信道信息获取、满足信道互易性时的下行信道信息获取以及上行波束管理。NR 定义了 3 种类型的 SRS 传输：周期性 SRS，半持续性 SRS 和非周期性 SRS，通过为 SRS 资源集和 SRS 资源配置关于时域类型的高层参数 resourceType 来实现。

* 周期性 SRS。时域类型被配置为周期 SRS 资源的所有参数由高层信令配置，UE根据所配置的参数进行周期性发送。同一个 SRS 资源集内的所有 SRS 资源具有相同的周期性。考虑到 NR 系统支持各种子载波间隔，不同子载波间隔对应的时隙时长不同，周期 SRS 资源的周期以及周期内的偏移以时隙为单位进行配置。周期 SRS 资源可配置的最小周期为 1 个时隙，最大周期为 2560 个时隙。

* 半持续性 SRS。时域类型被配置为半持续 SRS 资源在激活期间也是周期性发送。它与周期性 SRS 的区别在于 UE 在接收到关于半持续 SRS 资源的高层信令配置后不发送 SRS，只有在接收到 MAC 层发送的关于半持续 SRS 资源的激活信令后才开始周期性地发送半持续 SRS 资源对应的 SRS ，在收到 MAC 层发送的半持续 SRS 资源的去激活命令后停止发送 SRS。因此，相对于周期性 SRS 资源，半持续 SRS 资源的配置以及激活、去激活相比高层信令（RRC信令）更快，更灵活，适用于要求时延较低的业务的快速传输。与周期性 SRS 资源类似，基站通过高层信令为半持续 SRS 资源配置周期和周期内的偏移，同一个 SRS 资源集内的所有SRS资源具有相同的周期性。

* 非周期性 SRS。时域类型被配置为非周期 SRS 资源通过 DCI 信令激活。UE 每接收到一次触发非周期 SRS 资源的 SRS 触发信令，UE 进行一次所触发的 SRS 资源对应的 SRS 发送。DCI 中的 SRS 触发信令包含 2 个比特（如表 Table 7.3.1.1.2-24 所示），2 个比特可表示的 4 个状态。其中中的 1 个状态表示不触发非周期 SRS 发送，其他 3 个状态分别表示触发第一、第二、第三个 SRS 资源组；一个状态可以触发一个或多个 SRS 资源集，一个状态对应的多个 SRS 资源集可以对应多个载波。

  ![img](https://s2.loli.net/2023/04/16/xIZ3eipjRtKAMkr.png)

## SRS资源定义

SRS资源是由**“天线端口数目 $N_{ap}^{SRS}$，连续的 OFDM 符号数目$ N_{symb}^{SRS}$，时域起始符号 $l_0$ 和频域起始位置 $k_0$”**四个信息共同确定。以下是详细的说明：

- **天线端口数 **$N_{ap}^{SRS}$：$N_{ap}^{SRS}$的取值为$\{1,2,4\}$，天线端口索引表示为$\{p_i\}_{i=0}^{N_{ap}^{SRS}-1}$，其中$p_i=1000+i$。
  
  - 如果高层参数 `SRS-ResourceSet` 中的 `usage` 未配置为 `nonCodebook`，则$N_{ap}^{SRS}$ 由 `nrofSRS-Ports` 确定。
  - 如果高层参数 `SRS-ResourceSet` 中的` usage` 配置为 `nonCodebook`，则 $N_{ap}^{SRS}$取决于 section 5.2
  
- **连续的 OFDM 符号数目**$ N_{symb}^{SRS}$：由高层参数 `resourceMapping` 中的 `nrofSymbols` 确定。

- **时域起始符号**$l_0$：
  $$
  l_0=N_{symb}^{slot}-1-l_{offset}
  $$
  其中，$l_{offset}=\{0,1,\dots,5\}$且$l_{offset}\geq N_{symb}^{slot} -1$。

- **频域起始位置**$k_0$：SRS 的频域起始子载波。 

## SRS序列生成

基于Zadoff-Chu序列产生：
$$
r^{(p_i)}(n,l')=r_{u,v}^{(\alpha_i,\delta)}(n)\\
0\leq n \leq M_{sc,b}^{SRS}\\
l'\in\{0,1,\dots, N_{symb}^{SRS}-1\}
$$
$r_{u,v}^{(\alpha_i,\delta)}(n)$是扩展 ZC 序列（注：一种伪随机数序列），序列长度为$M_{sc,b}^{SRS}$，其中$\delta=\log_2(K_{TC})$，$\alpha_i$是对应于天线端口$p_i$的循环移位。$l'$是SRS资源的OFDM符号索引。传输梳齿数目$K_{TC}$由高层参数 `transmissionComb` 确定，循环移位$\alpha_i$由下式给出：
$$
\begin{aligned}
\alpha_i= &2 \pi \frac{n_{SRS}^{cs,i}}{n_{SRS}^{cs,max}}\\
n_{SRS}^{cs,i}=&(n_{SRS}^{cs}+\frac{n_{SRS}^{cs,max}(p_i-1000)}{N_{ap}^{SRS}}) \mod n_{SRS}^{cs,max}
\end{aligned}
$$
$n_{SRS}^{cs}$是由高层参数`transmissionComb`给定的。传输梳齿数目$K_{TC}$和最大循环位移数$n_{SRS}^{cs,max}$的对应关系如下表所示。

| $K_{TC}$ | $n_{SRS}^{cs,max}$ |
| :------: | :----------------: |
|    2     |         8          |
|    4     |         12         |
|    8     |         6          |

**最下面的一行似乎是今年最新版本才加进来的。**循环移位$\alpha_i$是$n_{SRS}^{cs}$和 $p_i$的函数，即使高层给定了 $n_{SRS}^{cs}$，不同的天线端口$p_i$也会使得循环移位$\alpha_i$不同。也就是说，同一个 UE 的不同天线端口所使用的 SRS 序列不同，每个天线端口对应的 SRS 序列是通过不同的循环移位得到的。低峰均比序列生成过程为$r_{u,v}^{(\alpha_i,\delta)}(n)=e^{j\alpha n}\overline r_{u,v}(n)$。其中$e^{j\alpha n}$表示对频域序列进行相位偏移，等效为时域的循环移位。一般会假定基序列$\overline r_{u,v}(n)$保持不变，则唯一变化的是相位偏移量 $e^{j\alpha n}$。尽管数学上称为频域的相位偏移，但 NR 标准统称为循环移位（cyclic shift）。引用一下[Marshall大佬](https://marshallcomm.cn/2020/11/21/r15_sounding_reference_signal/#用于下行-csi-获取的-ue-探测过程)总结的图：

![img](https://s2.loli.net/2023/04/16/ElOkNYuyLcTVPS7.png)

为了干扰随机化，5G NR 系统支持 SRS 序列跳和序列组跳，是否开启这项功能由高层参数 `groupOrSequenceHopping` 决定。SRS 的基序列被分成若干组，每组包含若干序列。如果基站给终端配置为 `groupOrSequenceHopping = neither`，则**每次终端发送的 SRS 序列不变**；如果基站给终端配置为 `groupOrSequenceHopping = groupHopping or sequenceHopping`，则每次终端发送上行 SRS 时按照以下规则采用不同的序列：

- 序列组号$u$:SRS 序列标识$n_{ID}^{SRS}$由高层参数 `sequenceId`给定，取值范围为 0~1023。
  $$
  u=(f_{gh}(n_{s,f}^{\mu},l')+n_{ID}^{SRS}) \mod 30
  $$

- 序列号$v$:

  取决于高层参数 `groupOrSequenceHopping` 的值。

在协议讨论阶段，RAN1#89 次会议同意使用 SRS sequence ID 来生成 SRS 序列。序列组号$u$是 SRS sequence ID 的函数，而 **SRS sequence ID 是 UE 特定的信息**。这意味着 **NR SRS 序列本身带有 UE 信息，而 LTE SRS 序列的生成是不带有 UE 信息的。**这样做的好处是，即使两个 UE 使用了完全相同的时频域资源发送 SRS，由于 SRS sequence ID 不同，又由于 **ZC 序列良好的互相关特性，那么两个 SRS 序列也具有较好的正交性。**



## 参考资料

[5G NR的物理信号和物理信道 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/401875663)

[5G NR - 参考信号(Reference Signal)学习笔记1 - Overview - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/378006580)

[5G NR - 参考信号(Reference Signal)学习笔记2 - Antenna Port(天线端口)的概念 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/381267548)

[5G NR - 参考信号(Reference Signal)学习笔记3 - QCL(准共址) Overview - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/381268009)

[5G NR - 参考信号(Reference Signal)学习笔记4 - QCL Type和各种组合 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/384679680)

[5G NR - 参考信号(Reference Signal)学习笔记5 - TCI Framework overview - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/385420380)

[5G Sounding Reference Signal | Marshall (marshallcomm.cn)](https://marshallcomm.cn/2020/11/21/r15_sounding_reference_signal/)