---
title: 5GNR SSB
categories: 学习记录
tags:
- 5GNR
- SSB
- OAI
- 同步
---



SSB（Synchronization Signal/PBCH, 同步广播块）是5G中使用的最重要的导频信道之一，其作用关系到UE接入小区的很多方面，如小区搜索、波束测量、波束选择、波束恢复等。SSB是由基站发出的，在进行一定的数量配置后，基站会使用不同方向的波束发送SSB，这其实就包含了一定的UE到gNB的角度信息。下面将介绍SSB的基本组成以及配置，以及OAI中nrUE中的`--ssb`参数的配置。

<!-- mroe -->

## SSB时频结构

在5G中，SSB包括同步信号和广播信号，具体同步信号包括**PSS**（Primary Synchronization Signal，主同步信号）和**SSS**（Secondary Synchronization Signal，辅同步信号）；广播信号包括**PBCH Data**和**PBCH DMRS**信号。具体SSB的时频域结构如下图所示，SSB在时域占据4个OFDM符号，在频域占据20个RB。

<img src="https://s2.loli.net/2023/09/01/IPd8MzG4F1C7mwq.png" alt="image-20230901103453540" style="zoom: 33%;" />

所有有颜色的色块构成的就是SSB。如上图所示，PSS和SSS分别位于SSB的sym0和sym2，频域上均占用127个RE，而对于sym0和sym2上20RB以内的**其他空闲RE，则不能调度其他信道信号**。PBCH数据和DMRS信号均位于SSB后三个符号，其中在sym2时，SSS的上下两端与PBCH分别间隔9个和8个RE，**这样设计是为了在SSS和PBCH信号间留有一定的保护间隔，抑制子载波间干扰。**注意，上图只是为了说明时频结构，SSB的具体位置还需要参考38.211的如下表格：

![image-20230901104848894](https://s2.loli.net/2023/09/01/4Poj3ZOVmIry8tL.png)

## SSB时域传输

我们在上面已经大致了解了SSB所处的位置，以及它的组成结构。下面我们将应用SSB，将其在无线NR中传输。

### SSB周期

与LTE中PSS/SSS的传输周期固定为5 ms不同，**NR中SSB的传输周期从5 ms到160 ms不等**。NR中，SSB的周期可以配置为5 ms、10 ms、20 ms、40 ms、80 ms和160 ms，由高层参数`ssb-periodicityServingCell`给出。但是，当U**E在进行初始小区搜索，以及在空闲状态下作小区搜索以作移动时，UE可以假设SSB的传输周期为20 ms**。这样，UE就可以知道在某个频率上搜索SSB需要停留的时间。如果UE在这段时间内没有搜索到PSS/SSS，则UE会转换到同步栅格上的下一个频率上继续搜索。

> 关于同步栅格的概念，可参考5G频段相关术语一文。

较长的SSB周期可以使基站处于深度睡眠状态，从而可以达到降低基站功耗以节能的目的，也有利于节约OFDM符号等系统开销。缺点是会导致UE长时间停留在某个频率上以搜索SSB，即会增加UE开机后的搜索复杂度及搜索时间。

但是，SSB周期的增加不一定会影响用户的体验，一是因为现在终端的开关机频次较低，开机搜索复杂度和时间的适当增加并不一定会严重影响用户的体验；二是NR使用了比LTE更稀疏的同步栅格，这在一定程度上抵消了由于SSB周期增加所导致的搜索复杂度的增加。

在实际网络部署的时候，可以根据基站类型、业务类型等设置SSB周期。例如，宏基站覆盖范围大，接入用户多，因此可以设置较短的SSB周期以便快速同步和接入；而微基站覆盖范围小，接入用户少，因此可以设置较长的SSB周期以节约系统开销和基站功耗。再例如，对于时延要求高的uRLLC业务，可以设置较短的SSB周期；而对于时延要求不高的mMTC业务，则可以设置较长的SSB周期

### SSB突发集合概念

> *SSB突发集合*这一术语源于3GPP早期的讨论。在3GPP早期讨论中，假设SSB组成SSB突发，而SSB突发又组成SSB突发集合。但是，中间的*SSB突发*这一术语最终并未被使用，而*SSB突发集合*这一术语被保留了下来

NR中，SSB通过波束扫描的方式传输，即通过时分复用的方式在不同的波束上传输SSB。**一个波束扫描内SSB的集合就称作SSB突发集合（SS Burst Set）**。需要指出的是，上面一节所说的**SSB周期其实是指SSB突发集合的传输周期**。但在**每个SSB周期内，SSB突发集合总是被限制在5 ms的时间间隔内**，要么在每个帧的前一个半帧内，要么在每个帧的后一个半帧内，如下图所示。如果对帧、子帧、时隙的概念比较模糊的话，可以参考之前的文章。

![在这里插入图片描述](https://s2.loli.net/2023/09/01/76p5OqlNacEQLSX.png)

### SSB图样

每个SSB突发集合内的最大SSB数因频带不同而不同。SSB在每个SSB突发集合（长度为半帧）内可能的位置（因此此处的SSB称作候选SSB），即**SSB图样**（Pattern），具体有如下A、B、C、D、E、F、G共7种情况，38.213中有详细定义。

![image-20230901114124886](https://s2.loli.net/2023/09/01/47cEqVfWnDzUM5Y.png)

> 需要注意，在3GPP演进过程中，CaseC的定义被修改过多次。

这里涉及的情况比较多，我拿OAI的默认配置作为解释，至于其他的Case可参考[这位博主的文章](https://blog.csdn.net/Graduate2015/article/details/118613213?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-118613213-blog-113067577.235^v38^pc_relevant_anti_vip&spm=1001.2101.3001.4242.1&utm_relevant_index=3)。在OAI 中，我们使用的是band78以及子载波间隔为30kHz，对应的是`Case C`的情况。此时我们一个帧有20个时隙，一个时隙是0.5ms，因此有：

![img](https://s2.loli.net/2023/09/01/I96hogxkSWUBYLe.png)

> 在5GNR中，允许SSB与数据和控制信道使用不同的子载波间隔，这样的设计可以保证，无论数据及其相应的控制信道使用的是SCS＝15 kHz还是SCS = 30 kHz，都可以最大程度降低由于SSB的传输导致的对数据传输的影响。

解释一下`OFDM Symbol(s)`那里的公式：`{2，8}`是在每个时隙中的位置，每一个`n`的取值其实是SSB的标号。因此我们可以看到，在Case C中，最多可以配置8个SSB。

### SSB激活

接下来的问题就是，我是如何控制配置的SSB数量呢？SSB图样给出的是在SSB突发集合内，SSB可能的候选位置以及SSB的最大值$L_{\max}$，基站通过系统消息SIB1或UE专用的RRC信令的高层参数`ssb-PositionsInBurst`通知给UE具体是哪些SSB被激活使用这个配置在OAI里：

```bash
# ssb_PositionsInBurs_BitmapPR
# 1=short, 2=medium, 3=long
      ssb_PositionsInBurst_PR                                       = 2;
      ssb_PositionsInBurst_Bitmap                                   = 1;
```

`short`/`medium`/`long`的字段，分别对应的是下方的Bitmap的长短。`Short`表示Bitmap只按照4位比特读取，比如说`1010`,`0101`,`0001`等等，开启对应位置的SSB。不可以不配置SSB的哈~如果是`Medium`的话，最多是8位比特；如果是`long`那么则是64位比特，一般会以十六进制数进行保存，比如说`0xff00000000000000`。

> SSB突发集合内的SSB数量与天线的波束宽度密切相关。天线发射的波束越窄，需要配置的SSB数量越多，而波束的宽度与载波频率和天线增益有关。对于定向天线，频率越高、增益越大、波束越窄，因此需要配置的SSB数量越多。宏基站需要通过较大的天线增益和较窄的波束实现较大的覆盖范围，因此需要配置的SSB数量较多；而微基站由于覆盖范围较小，波束较窄、波束较少，因此配置的SSB数量也较少。波束数量较多的优点是可以通过波束扫描获得较大的覆盖增益，缺点是增加了系统复杂性和开销；波束较少的情况则相反。
>
> 对于配置了载波聚合的小区，SSB的数量还与小区类型有关。由于UE是在主服务小区（PCell）上进行小区搜索和随机接入，为了较少系统开销，辅小区可以不配置SSB，UE通过同一组小区内的主服务小区或主辅服务小区（PSCell）的SSB获得时间和频率同步。

## OAI中nrUE的`--ssb`参数计算

这部分要感谢我的好Bro 冰老师。UE里面涉及到频点配置的有2个参数：

- `-C` ：中心频点
- `--ssb`：ssb的最底的子载波到pointA的**距离**（子载波个数）

冰老师给了个例子，我们来看看：

### 例子1

**已知**：

1. 带宽为106 RB 
2. `dl_absoluteFrequencyPointA = 620040 `

pointA的ARFCN是620040对应的频率是3300.6MHz。这个计算如果你偷懒的话，可以使用[这个工具](https://5g-tools.com/5g-nr-gscn-calculator/)进行换算：

<img src="https://s2.loli.net/2023/09/01/82OasI4gfzM3E7G.png" alt="image-20230901143034289" style="zoom:67%;" />

带宽是106RB ，中心频点 = 53RB + pointA的频率 = 53\*12\*30k + 3300.6M = `3319.68e6`，因此在输入`-C`的时候需要输入`3319680000`。然后，我们将中心频点转换到ARFCN：

<img src="https://s2.loli.net/2023/09/01/VH3XflIhZWJ5iyK.png" alt="image-20230901143300463" style="zoom:67%;" />

当前SSB的绝对频点也就是621312；（这里走了狗屎运，正好带宽中心频点可以配置成SSB的中心频点）此时，计算`--ssb`参数要计算SSB的最底子载波到pointA的子载波个数，因为SSB的中心频点到最底子载波还有10个RB，所以就是：
$$
\begin{aligned}
&((3319.68-3300.6)\times 10^6)/(30*10^3)-10\times12\\
=&636子载波-120子载波\\
=&516
\end{aligned}
$$

### 例子2

冰老师截的图咯：

<img src="https://s2.loli.net/2023/09/01/mZPi5y7nYcIavFJ.jpg" alt="f3f89838c01c81a47b044aaa7c3ecd5" style="zoom:33%;" />

SSB的频率为3408.96，pointA的ARFCN是626724。使用工具可计算PointA的频率为3400.86。因此，`--ssb`计算为：
$$
\begin{aligned}
&((3408.96-3400.86)\times 10^6)/(30*10^3)-10\times12\\
=&270-120\\
=&150
\end{aligned}
$$
