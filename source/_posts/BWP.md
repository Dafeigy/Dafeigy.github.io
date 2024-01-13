---
title: 5G中的BWP
categories: 学习记录
tags:
- 5G
- BWP
---

BWP其实是(Bandwidth Part)的缩写，中文译名为部分带宽。这是5G区别于4G的最关键概念之一，可谓无线资源的“切片”，可使5G灵活支持各种场景的各类终端。那么具体是怎样的呢？我们先从它的出现说起，然后再详细的从3GPP标准中一探究竟。

<!-- more -->

## 为什么需要BWP？

BWP是5G引入的新概念，而引入这个新的概念是因为4G中有做得不够完善的地方，所以需要5G补充完善。这个不完善的地方是什么呢？在3GPP R8协议中，规定了4G的载波带宽最大是20MHz；在R10引入了载波聚合，最大支持5个载波，把4G的最大工作带宽扩展到了100MHz。按理说这个带宽已经很大了，轻轻松松就可以支持1Gbps的峰值下载速率，然而3GPP还是不满足，在其LTE Advanced Pro版本里面，又把最大载波聚合数量提升到了32，这下4G一共需要支持640MHz的带宽！

但是我们需要考虑一个问题，基站作为强悍的系统设备，支持这么大的带宽倒是问题不大，可是用户终端UE的发射功率只有0.2瓦，还要控制发热保障续航，非常难支持这么多载波和这么大的带宽。所以，截止现在，即使是高端的4G手机，也就支持到5载波聚合，最大100MHz带宽。随着5G的到来，4G已成为明日黄花，人老珠黄，奔冷宫而去，自然没人愿意再给4G投资升级了，最大支持32载波也只能停留在纸面上。

但是到了5G就不一样了。Sub6G频谱**每载波最大需要支持100MHz**，毫米波甚至**每载波达到了400MHz**，协议还规定5G可以**支持最多16个载波聚合**！说人话就是，**5G中最大需要支持1.6GHz到6.4GHz的超大频谱带宽**！这样一来，别说用户终端受不了了，连基站要支持起来也费劲。

另外一个问题，则是设备升级的问题。我们看看5G的三大应用场景：

<img src="https://s2.loli.net/2023/08/02/GvSsNT6Xk8oexhF.jpg" alt="5G三大应用场景" style="zoom:50%;" />

注意，超大频谱带宽，只使用于`eMBB`场景，而`mMTC`和`uRLLC`这样的物联网垂直行业场景，往往并不需要太高的速率，也就不需要太大的带宽。在这种情况下，强制让所有的终端（注意，手机只是5G的诸多类型的终端之一）都支持大带宽，搞一刀切这样的粗放管理是非常不经济的。那我们该怎么办呢？轮到我们的主角出场了：BWP。说白了，BWP就是让频谱资源在不同的用户终端上得到更好的分配。

## BWP有什么用？

BWP相当于把5G的频谱在一定的时间内划分成了很多的小块，每个BWP可以使用不同的参数集，其带宽，子载波间隔，以及其他控制参数都可以不同，相当于在5G小区内部又划分出了若干个配置不同的子小区，以适应不同类型的终端及业务类型。

<img src="https://pic3.zhimg.com/80/v2-e17ba3e13dd377f8774b973b12417bf2_720w.webp" alt="img" style="zoom: 67%;" />

上图直观地表达了一个BWP分配的例子。其中蓝色的矩形表示BWP1，带宽大，但持续的时间短；绿色的矩形表示BWP2，其带宽小，但持续的时间长；橙色的矩形表示BWP3。

第一个时刻，手机正在下载游戏安装包文件，需要较高的速率，因此系统给手机配置了一个大带宽的BWP（BWP1）；

第二时刻，手机在进行在线游戏，需要的流量较小，因此系统给手机配置了一个小带宽的BWP（BWP2），满足其游戏的上下行速率需求即可；

第三时刻，系统发现BWP1所在带宽内遭遇了很强的突发性外部干扰，信号质量急剧恶化，无法满足手机的上网需求了，于是紧急给UE配置了一段新的带宽（BWP3）。

一个终端最大可以支持4个BWP，但同时只能有一个处于激活状态。这样一来就实现了频谱资源的按需使用，不必像4G一样，手机必须在整个载波带宽上工作，更加灵活，也更加省电。

BWP本质上属于5G在空口资源的切片，上面例子中是一个终端内部的BWP的切换，实际上，由于不同BWP的参数集可以不同，它们可以用于完全不同的场景，如下图所示。

<img src="https://pic1.zhimg.com/80/v2-033effa9449f12290829d2bbb92bc994_720w.webp" alt="img" style="zoom: 67%;" />

看起来铁板一块的5G载波，实际在内部可以进行灵活的BWP分配来实现切片，灵活支持eMBB，mMTC和uRLLC这三种最为典型的5G业务。如果看到这你已经了解了BWP的相关概念的话，接下来的内容就是枯燥乏味的标准文件的阅读了。

## 在BWP前，学习一下PointA

PointA的概念是在38.211里面定义的。它是公共物理资源块0 (Common Reference Resource Block 0) 的子载波起始位置，即CRB0的子载波0的中心频点。PointA在TDD模式下上下行是一样的，但对于FDD或SUL的场景则是分别对上下行进行定义。它其实就是一个基准参考点。 我们举个例子：

<img src="https://s2.loli.net/2023/08/03/VGwfnUdACLJqXOg.png" alt="image-20230802190245392" style="zoom: 50%;" />

我们可以看到，SSB和CRB他们是没有对齐的，这是因为SSB遵循的是广播信道栅格，而CRB遵循的是信道栅格。所以他们之间有一定的差别，这个差值就是所谓的$k_{SSB}$，另外CRB之间的间隔是在MIB中的一个名为`subCarrierSpacingCommon`定义的。`offsetToPointA`是和SSB重合的CRB下边缘到PointA的距离，以RB作为衡量，可用于配置PointA的位置，该项放在`SIB`内。

另一种定义PointA的方式则是通过`absoluteFrequencyPointA`进行设定，视情况在FrequencyInfoDL或（与）FrequencyInfoUL中的absoluteFrequencyPointA配置。这种情况是载波聚合的情况，这个时候配置了CA的SCell，我们需要告知UE其中SCell的位置。

注意：PointA的起始位置并不是小区配置的带宽的起始位置，小区载波的下沿相对于PointA的位置和小区总带宽还需要通过SIB1的`SCS-SpecificCarrier`确定。我们举个例子：

![image-20230803164914570](https://s2.loli.net/2023/08/03/mKO49osAXbcjp8i.png)

在这里我们定义两个Carrier，其中`Carrier0`是100MHz-273PRB，子载波间隔为30kHz。它的起始位置直接和CRB0的子载波0的中心位置重合（注意，并不是子载波0的起点位置，MJ的图有问题），也就是PointA所在的位置，此时`offsetToCarrier`为0；另外一个载波则是50MHz-270PRB，子载波间隔为15kHz，此时offsetToCarrier的长度为这一段的RB个数乘以`Carrier1`的子载波间隔15kHz然后再乘以12（对应的是一个RB中的子载波个数）。

好了前面铺垫了这么多，我们做个总结吧！再次回到这张图：

<img src="C:\Users\10706\AppData\Roaming\Typora\typora-user-images\image-20230803170211711.png" alt="image-20230803170211711" style="zoom: 33%;" />

NR中的SSB可以在传输载波的任何位置，SSB的位置服从的是同步栅格，而PDCCH/PDSCH的载波的中心频率服从信道栅格，SSB子载波0的位置甚至不与物理资源块对齐。就如上图这样子，SSB的子载波0与CRB0的偏移等于offsetToPointA (单位是RB) + $k_{SSB}$。

> OAI中pointA的相关配置
>
> 在OAI中，pointA是在基站的配置文件中`dl_absoluteFrequencyPointA`配置的，代表的是pointA的绝对频点。

## 正式开始BWP啦！

BWP其实是在整个载波里面划分出一部分的区域作为一段这段频谱中的子集，而UE只会在划分的这一段频谱中进行业务，而不会在其它地方进行业务，如图，$RB_{start}$和 $L_BWP$定义了载波中的BWP位置和长度信息。

<img src="https://s2.loli.net/2023/08/03/OeNusRyB69mbGY2.png" alt="image-20230803171459518" style="zoom: 50%;" />

在前面我们说过了，BWP的提出就是为了**节能**，这是为了照顾一些功率有限的终端的资源消耗

### BWP的使用场景

BWP一共有四种使用场景，如图所示：

<img src="https://s2.loli.net/2023/08/03/s7WXiqxU3lMmNdp.png" alt="image-20230803174449518" style="zoom:67%;" />

图1非常好理解，这里不再多赘述了；图2就是分配两段BWP，并且只有在进行大业务的时候才使用BWP#1以降低手机的功耗；图3就是支持不同子载波间隔的BWP，图4就是抗干扰，蓝色的部分可能性能比较差，那么在分配BWP的时候就可以将其避开，让UE在稍微好一点的频段中工作。

### BWP的位置

我们前面讨论了PointA和与之相关的位置的概念，现在轮到BWP的位置了。BWP的起始位置是这样定义的：
$$
N_{BWP,i}^{start}=O_{carrier}+RB_{start}
$$
其中 $O_{carrier}$指的是`offsetCarrier`。在实际中，为了节约存储使用了一个RIV（resource indicator value）进行指示RB的起始位置与长度。计算公式如下：
$$
RIV=\begin{cases}
N_{BWP}^{size}(L_{RBs}-1)+RB_{start}\qquad \quad if\,L_{RBs}-1\leq[N_{BWP}/2]\\
N_{BWP} \cdot(N_{BWP}-L_{RBs} +1) + (N_{BWP} -1 -RB_{start})\quad else
\end{cases}
$$
其中，$N_{BWP}$是一个常数=275，表示的是BWP支持的最大长度。

### BWP的配置

#### 下行

- 可以**分配最多4个BWP，但不包括初始BWP**。
- BWP的带宽要大于等于SSB的带宽，但可以选择包含或不包含SSB。
- 在同一时刻，只有一个BWP可以被激活
- UE不希望在激活的BWP外收到一些下行的数据，比如说PDSCH，PDCCH，CSI-RS或者TRS。
- 每一个下行的BWP至少要有一个用户搜索空间的对应的CORESET
- 在一个主载波内，下行的BWP需要包含至少一个公共的搜索空间（CSS）的CORESET

#### 上行

- 可以**分配最多4个BWP，但不包括初始BWP**。
- 在同一时刻，只有一个BWP可以被激活
- 对于纯上行  (Supplementary uplink) 的：
  - 可以再额外配置多4个BWP
  - 在同一时刻，只有一个BWP可以被激活
- UE不能在一个激活的BWP外传输PUSCH或者PUCCH

## BWP的类型

- 初始BWP。这是用于在初始的随机接入时比如说解调SIB时，它包含了像RMSI (Requested Minimum System Information) ，CORESET（某一段PDCCH的时频资源范围）和RMSI Frequency location/Bandwith/SCS的参数信息。它可以配置为24~96个PRB的大小。
- 激活的BWP。这就是手机进行业务的BWP，一般通过RRC 重配或者RRC建立配置。手机得到了这些信息后可以不定时的激活某一个BWP。
- 默认BWP。UE不进行业务的时候要回到的状态。它同样通过RRC重配或者RRC建立及逆行配置，如果不进行配置的话，则初始BWP就是默认的BWP。

BWP有一个定时器，叫做`BWP Timer`，当定时器超时，手机没有业务，那么则会回到默认的BWP，所以我们会至少配备一个初始的BWP用于小区接入。