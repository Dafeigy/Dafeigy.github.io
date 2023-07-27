---
title: CSI-RS 作用
---



CSI是信道状态信息Channel State Information的缩写，对于分析无线通信系统的性能非常重要。

<!--more-->

## CSI-RS作用

* 3GPP定义了CQI（Channel Quality Indication，信道质量指示）这个概念，CQI用于UE 想网络侧指示，通过下行测量，UE所感知到的下行链路的信干扰噪声比 (SINR)。
* 通过UE所报告的SINR,网络决定UE在下行的频谱效率，而频谱效率可以通过通过香农定理得到。
* 当频谱效率被决定了以后，那么下行的码率就决定了，因为频谱效率的本意就是成功发送每个bit所需要的赫兹。
* 如果码率决定了，那么MCS（Modulation and Coding Scheme，调试与编码策略：调制方式和码率）也就确定了。
* 此外还有**波束管理**（主要包括发端波束测量、收端波束测量以及收发端同时波束测量），**时频跟踪**（精确测量时偏和频偏，通过TRS信号实现），**移动性管理**（通过对本区以及邻区的CSI-RS跟踪测量），**速率匹配**（主要ZP CSI-RS实现对PDSCH的RE级速率匹配）等。

从原理上，UE需要一个参考信号，来测得下行的SINR，并且进行上报，那么CSI-RS的基本功能就是提供这个参考信号。

CSI-RS分为非零功率NZP CSI-RS和零功率ZP CSI-RS两种：

* Zero power CSI-RS（不需要产生并映射到RE上，用于PDSCH的速率匹配，不发东西）
* Non zero power CSI-RS（需要实际产生并映射到RE上）
  1. 主要的CSI report 
  2. TRS
  3. LI-RSRP computation:参考信号接收功率（参数Repetition=on/off）
  4. Mobility Management

## CSI-RS的配置

CSI-RS resource的port数量可以是单个port，也可以是multi-port，最多到32个port。在multi-port映射的时候会用到CDM的这个概念，即多个CSI-RS port可以在相同时频资源上通过CDM的方式加以区分和映射。协议中规定的CDM类型有4种，种类可根据RRC中参数cdm-Type的值，有{noCDM,fd-CDM2,cdm4-FD2-TD2,cdm8-FD2-TD4}，其中其中noCDM就是将CSI-RS只映射在一个RE上，没有码分的概念，其余3种如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713140743885.png)

* FD-CDM2： 在频域2载波，时域1符号的2个RE上实现2个port的复用
* CDM4-FD2-TD2：在频域2载波，时域2符号的4个RE上实现4个port的复用
* CDM8-FD2-TD4：在频域2载波，时域4符号的8个RE上实现8个port的复用

## CSI-RS时频资源映射

![img](https://img-blog.csdnimg.cn/2020030810193692.png)

UE通过RRC中下发的CSI-RS-ResourceMapping，可以知道下面的参数：
1.时域上的相关的参数 :l=l_+l’,其中l_有两个取值，l0和l1，值会通过RRC参数firstOFDMSymbolInTimeDomain和firstOFDMSymbolInTimeDomain2得到，l0和l1的区别在于不同的场景中使用，表示CSI-RS的时域起始符号。

2.频域上的相关参数：k，主要依赖于k_和k’，k_取值可能为k0,k1,k2,k3，这些值会通过RRC参数frequencyDomainAllocation计算得出。

3.(k_,l_)表示在不同的密度和CDM Type时的时频资源组合，其值由表7.4.1.5.3-1得出。

4.Wf(k’)和Wt(l’)表示不同CDM Type对应的正交码权值，由表7.4.1.5.3-1得出。

## 参考链接

https://blog.csdn.net/GiveMe5G/article/details/104727682