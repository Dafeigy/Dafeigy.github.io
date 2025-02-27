---
title: CSI-RS详解
mathjax: true
tags:
- 5GNR
- 参考信号
categories:
- 学习记录
---

# 5GNR-CSI RS

CSI-RS的一些简要概述如下：

> - 它是一种特殊的参考信号，由gNB传输给UE，用于估计下行无线信道质量。
> - 在LTE中，CRS（小区参考信号）通常用于此目的，但NR没有CRS。我们可以将NR中的CSI RS视为LTE中CRS的对应部分。
> - CRS（LTE）和CSI RS（NR）的主要区别在于，CSI-RS应由RRC配置传输（意味着它不是强制性信号），而CRS始终在传输。
> - 另一个区别是NR CSI-RS在设计时考虑了波束成形。
> - CSI RS端口数有多种选择：1,2,4,8,16,24,32。
> - 单端口配置（p1）主要用于TRS（跟踪参考信号），多端口配置（如p2或更高）主要用于CSI测量。
> - CSI RS可以在任何OFDM符号和子载波中传输，传输方式几乎是任意的（**不是100%任意，而是超级灵活的**），它将在RRC报文中进行配置。

与LTE CSI一样，NR中的CSI（信道状态信息）是UE测量各种无线信道质量并将结果报告给网络（gNB）的机制。CSI操作涉及的因素相当复杂，但在这里将重点介绍CSI信号生成和资源映射。其他CSI操作步骤将在其他页面进行说明。

在大多数LTE情况下，我们不需要任何特殊的CSI信号，因为我们使用的是小区专用参考信号。然而，从LTE TM9开始，我们开始为CSI使用特殊的参考信号。因此，如果对TM9或更高版本的LTE CSI参考信号有一定的了解，那么即使NR CSI RS（参考信号）的构造比LTE CSI RS更复杂，也会相对更容易理解本页内容。

<!-- more -->

在开始前有必要再阐述一下CSI-RS的主要功能：

- **汇报CQI(Channel Quality Indication，信道质量指示)**，这是用于UE向网络侧指示下行测量，UE所感知到的下行链路的信干扰噪声比(SINR)，反映下行PDSCH的信道质量。用0~15来表示PDSCH的信道质量。0表示信道质量最差，15表示信道质量最好。
- **汇报PMI(Precoding Matrix Indicator，预编码矩阵指示)**，这是用于指示码本集合的index。PMI用于指示发送端应该使用哪个预编码矩阵来编码数据，以最大化接收端的信号质量。
- **汇报RI(rank indication，秩指示)**，用于指示无线信道中可以支持的最大MIMO（Multiple-Input Multiple-Output）等级或天线数量。它告知接收端，在当前的信道条件下，可以支持的最大天线数量。RI的取值范围通常是1到4，表示支持的天线数量。也就是说，尽管可能配置了MIMO，但是RI如果依然指示为单天线的话那依然只有单天线。
- **汇报RSRP(Reference Signal Received Power，参考信号接收功率)**，主要是用于波束管理，比如说波束的方向之类的。
- **时频跟踪**，精确测量频偏和时偏，通过TRS实现。
- 速率匹配，主要用于用ZP-CSI-RS实现对PDSCH的RE级别的速率匹配。ZP是Zero-Power的意思，不需要映射到RE上面。
- **CSI-IM**，用于干扰测量。发送的CSI-RS为0的部分，UE如果接收到的不是0，那么说明有干扰。

## 序列生成与资源映射

如下式所示，NR CSI基于伪随机序列。然后，该序列在时域和频域通过各自设计的加权序列进行乘法运算，并通过功率缩放因子进行缩放。然后将该序列映射到资源网格中的一组特定资源元素。所有这些过程可总结如下，在此过程中涉及许多因素。
如果您不是需要实现这部分功能的物理层开发人员，您不需要了解全部细节，但关注一下这个过程中涉及的各种参数（尤其是RRC参数）将会对您有所帮助。

![img](https://www.sharetechnote.com/html/5G/image/NR_CSI_RS_Sequence_ResourceMap_01.png)

通过这位@[**MJstudio小麦**](https://www.bilibili.com/video/BV18M411z7ZS/?vd_source=ab34db443b112b108b42c31ac575fd1f)Up主的分享，我会进行一定的补充。前面说到NR-CSI是基于伪随机序列的，但是从伪随机到真正发出去的CSI-RS还需要经过三步：频域的正交序列化、时域的正交序列化以及功率的倍数调整。

## 资源粒子映射表格

![img](https://www.sharetechnote.com/html/5G/image/38_2_1_Table_7_4_1_5_3_1.png)

![img](https://www.sharetechnote.com/html/5G/image/NR_CSI_RS_RE_Mapping_Table_01.png)

CSI-RS在时域的参考位置由RRC层确定，如下所示:

- $l_0$：`firstOFDMSymbolInTimeDomain`确定。
- $l_1$= `firstOFDMSymbolInTimeDomain2`确定。

CSI-RS在频域的参考位置`（k1,k2,k3）`由RRC层确定，如下所示：

- $[b_3\cdots b_0],k_i=f(i)$
- $[b_{11}\cdots b_0],k_i=f(i)$
- $[b_{2}\cdots b_0],k_i=4f(i)$
- $[b_{5}\cdots b_0],k_i=2f(i)$

$f(i)$是`frequencyDomainAllocation`中设置为1的第`i`位的位号，比如给定的bitmap为 $[1010]$则第一个i为从右往左数的第2位，也就是`1`号位；第二个1为从右往左数的第4位，对应的是`3`号位，每`1/density`重复一次。

端口(Ports)，密度(Density)还有cdm-type通过如下的RRC参数配置：

- `Ports`=`nrofPorts`
- `cdm-Type`=`cdm-Type`
- `Density`=`density`

## CDM序列生成表格

CDM表用于生成CSI-RS信号，如下图所示。

![img](https://www.sharetechnote.com/html/5G/image/5G_CSI_RS_CDM_Table_01.png)

这里其实是将多个端口的数据映射到一个分组的resource里面。我们需要关心的是 $(\overline k, \overline l)$是每一个分组的CSI-RS的起始位置即可。每x个端口映射到同一个时频资源里，但是他们之间通过$w_f$和 $w_t$的编码矩阵映射。

CDM 中的正交序列矩阵定义如下：

<center><big><b>cdm-Type:"noCDM"</b></big></center>

| index | $w_f(0)$ | $w_t(0)$ |
| :---: | :------: | :------: |
|   0   |    1     |    1     |



<center><big><b>cdm-Type:"fd-CDM2"</b></big></center>

| index | $[w_f(0)\qquad w_f(1)]$ | $[w_t(0)\qquad w_t(1)]$ |
| :---: | :---------------------: | :---------------------: |
|   0   |     $[+1 \quad +1]$     |            1            |
|   1   |     $[+1 \quad -1]$     |            1            |

<center><big><b>cdm-Type:"cdm4-FD2-TD2"</b></big></center>

| index | $[w_f(0)\qquad w_f(1)]$ |    $w_t(0)$     |
| :---: | :---------------------: | :-------------: |
|   0   |     $[+1 \quad +1]$     | $[+1 \quad +1]$ |
|   1   |     $[+1 \quad -1]$     | $[+1 \quad +1]$ |
|   2   |     $[+1 \quad +1]$     | $[+1 \quad -1]$ |
|   3   |     $[+1 \quad -1]$     | $[+1 \quad -1]$ |

<center><big><b>cdm-Type:"cdm8-FD2-TD4"</b></big></center>

| index | $[w_f(0)\qquad w_f(1)]$ | $[w_t(0)\quad w_t(1) \quad w_t(2) \quad w_t(3)]$ |
| :---: | :---------------------: | :----------------------------------------------: |
|   0   |     $[+1 \quad +1]$     |        $[+1 \quad +1 \quad +1 \quad +1]$         |
|   1   |     $[+1 \quad -1]$     |        $[+1 \quad +1 \quad +1 \quad +1]$         |
|   2   |     $[+1 \quad +1]$     |        $[+1 \quad -1 \quad +1 \quad -1]$         |
|   3   |     $[+1 \quad -1]$     |        $[+1 \quad -1 \quad +1 \quad -1]$         |
|   4   |     $[+1 \quad +1]$     |        $[+1 \quad +1 \quad +1 \quad +1]$         |
|   5   |     $[+1 \quad -1]$     |        $[+1 \quad +1 \quad +1 \quad +1]$         |
|   6   |     $[+1 \quad +1]$     |        $[+1 \quad -1 \quad +1 \quad -1]$         |
|   7   |     $[+1 \quad -1]$     |        $[+1 \quad -1 \quad +1 \quad -1]$         |

使用CDM其实就是复用RE，举个例子，如图所示：

![image-20230718150836259](https://s2.loli.net/2023/07/18/uKO4dHnGRBhveL2.png)

假设现在有两个符号 $S=[a,b]$，nrofPorts为2，对其进行正交编码：
$$
S*W = 
\begin{bmatrix}
a,b
\end{bmatrix}
*
\begin{bmatrix}
W_0\\
W_1
\end{bmatrix}
=aW_0+bW_1
$$
则实际的RE映射值为：
$$
\alpha_0=
\begin{bmatrix}
a,b
\end{bmatrix}
*
\begin{bmatrix}
+1\\
+1
\end{bmatrix}
=a+b\\
\alpha_1=
\begin{bmatrix}
a,b
\end{bmatrix}
*
\begin{bmatrix}
+1\\
-1
\end{bmatrix}
=a-b\\
$$
最后UE那边对接收到不同的RE信号进行联合正交解码，从而获得原始的Port信号。本质上其实是解方程组。

## RRC参数：NZP-CSI-RS资源

本页几乎所有内容都是关于IE（信息元素）资源映射的。如下所示，IE resourceMapping是NZP-CSI-RS-Resource的一部分。

```
NZP-CSI-RS-Resource ::= SEQUENCE {

   nzp-CSI-RS-ResourceId      NZP-CSI-RS-ResourceId,

   resourceMapping            CSI-RS-ResourceMapping,

   powerControlOffset         INTEGER (-8..15),

   powerControlOffsetSS       ENUMERATED{db-3, db0, db3, db6} OPTIONAL, -- Need R

   scramblingID               ScramblingId,

   periodicityAndOffset       CSI-ResourcePeriodicityAndOffset OPTIONAL,-

   qcl-InfoPeriodicCSI-RS     TCI-StateId OPTIONAL, -- Cond Periodic

   ...

}
```

<font color="  #3CB371">**resourceMapping** </font>：详见[CSI-RS资源映射](#UE如何分辨网络中使用的CSI-RS？)节。

<font color=" #3CB371">**powerControlOffset** </font>：PDSCH的资源粒子到NZP CSI-RS资源粒子功率偏移，以dB为单位。

<font color=" #3CB371">**powerControlOffsetSS** </font>：NZP 的CSI-RS资源粒子到SSS资源粒子的功率偏移，以dB为单位。

<font color=" #3CB371">**qcl-InfoPeriodicCSI-RS** </font>：对于目标周期性CSI-RS，包含对TCI-States中的一个TCI-State的引用，以提供QCL源和QCL类型。对于周期性CSI-RS，源可以是SSB或另一个周期性CSI-RS。参考具有tci-StateId值为此值的TCI-State，该值在与服务小区对应的BWP下行链路中的PDSCH-Config中的tci-StatesToAddModList中定义，并且与资源所属的DL BWP相对应。

> For a target periodic CSI-RS, contains a reference to one TCI-State in TCI-States for providing the QCL source and QCL type. For periodic CSI-RS, the source can be SSB or another periodic-CSI-RS. Refers to the TCI-State which has this value for tci-StateId and is defined in tci-StatesToAddModList in the PDSCH-Config included in the BWPDownlink corresponding to the serving cell and to the DL BWP to which the resource belongs to.

简而言之，**这指示该 CSI-RS 被 QCL 到的 tci-StateId。**

## UE如何分辨网络中使用的CSI-RS？

当网络分配 CSI-RS 时，它会从 38.211-表 7.4.1.5.3-1 中选择特定行，并用特定信号填充一组资源元素。这部分很简单。难理解的部分是 UE 如何确定 gNB（网络）正在使用哪个 CSI-RS。答案是网络通过RRC消息通知UE CSI-RS的所有细节，我们来看看这个过程中涉及了哪些RRC参数：

#### <font color=" #3CB371">如何确定CSI-RS端口和cdm（通过38.211-Table 7.4.1.5.3-1的行号）？</font>

: 其实通过CSI-RS-ResourceMapping中的 `nrofPorts`和 `cdm-Type`配置。

![image-20230717145705515](https://s2.loli.net/2023/07/17/rjMXJ4P9ugdswbp.png)

#### <font color=" #3CB371">如何为选择的CSI-RS确定REs？</font>

: 这是由 CSI-RS-ResourceMapping 的 (k_bar, l_bar) 列以及FrequencyDomainAllocation 和firstOFDMSymbolInTimeDomain 配置的，如下所示。

![img](https://www.sharetechnote.com/html/5G/image/NR_CSI_RS_kbar_lbar_01.png)

![image-20230717145936850](https://s2.loli.net/2023/07/17/qFEoR5M3KwiWjZn.png)

<font color=" #3CB371">**startingRB** </font>：此 CSI 资源相对于公共资源块网格上的公共资源块 #0 (CRB#0) 开始的 PRB。<font color='red'>只允许使用 4 的倍数（0, 4, ...）</font>。

<font color=" #3CB371">**nrofRBs** </font>：该CSI资源跨越的PRB数量。<font color='red'>只允许 4 的倍数</font>。最小可配置数是 24 和相关 BWP 宽度的最小值。<font color="#DAA520">如果配置值大于相应 BWP 的宽度，则 UE 应将实际 CSI-RS 带宽设置为等于 BWP 的宽度。</font>

好了，结合上面的两部分内容，我们可以给出一个配置的例子啦：

![img](https://www.sharetechnote.com/html/5G/image/NR_CSI_RS_Resource_Determination_Ex01.png)

#### <font color="#3CB371">网络在使用时怎么知道当前使用的(逻辑)天线配置信息呢?</font>

这是个比较复杂的问题，我们会在[这里](https://www.sharetechnote.com/html/5G/5G_CSI_RS_Codebook.html)讲述。

## CSI-RS资源映射的例子

### Example1：

给定如下RRC配置信息：

```
density = three
nrofPorts = p1
cdm-Type = noCDM
frequencyDomainAllocation.row1 = 0001
firstOFDMSymbolinTimeDomain = 4
```

<center>根据38.211-Table 7.4.1.5.3-1</center>
![img](https://www.sharetechnote.com/html/5G/image/NR_CSI_RS_Table_Header.png)

![img](https://www.sharetechnote.com/html/5G/image/NR_CSI_RS_Table_Row1.png)

```
k prime = {0} ==> k prime[0] = 0
l prime = {0}  ==> l prime[0] = 0
```

通过如上的参数给定，RRC参数为：

```
k0 = 0 (0001，首位1出现在index=0的位置)
l0 = 4(firstOFDMSymbolinTimeDomain = 4给定)
```

再通过查表38.211-Table 7.4.1.5.3-1以及如上的所有信息，可以知道：
$$
\begin{aligned}
(k,l) = &(k0+kprim[0],l0+lprim[0]) ,(k0+4+kprim[0],l0+lprim[0]) ,(k0+8+kprim[0],l0+lprim[0])\\
= &(0+0,4+0) ,(0+4+0,4+0) ,(0+8+0,4+0)\\
= &(0,4) ,(4,4) ,(8,4)
\end{aligned}
$$
![image-20230717152835560](https://s2.loli.net/2023/07/17/NzM3ZmyQHRhnBGe.png)

## CSI-RS传输时间

通过如下式子确定：

![img](https://www.sharetechnote.com/html/5G/image/CSI_RS_TransmissionTiming_01.png)

## 干扰测量的资源（CSI-IM）

## 追踪参考信号（TRS）

## 零功率（ZP）CSI-RS

### 为什么是零功率CSI-RS

#### 专家解答

#### 文章参考

## CSI-RS的RRC参数设置

## RRC配置举例

## 如何避免其他信号？