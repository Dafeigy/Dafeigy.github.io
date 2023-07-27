---
title: 【文献翻译】Delay-Optimal Virtualized Radio Resource Scheduling in Software-Defined Vehicular Networks via Stochastic Learning
mathjax: true
tags:
- 车联网 
- 资源分配 
categories:
- 学习记录 
- 文献翻译 
---



<h1><center>摘要</center></h1>

由于现有的车辆高密度和多样的车载服务需求，在当前的LTE网络使用效益最高的方法保证车载服务质量充满了挑战。很幸运的是，随着5G技术的发展，大量的蜂窝小区的建立被视为可达到车联网低延迟需求的场景下的一个可行方案。然而，这可能会导致大量的操作开销的产生以及由受限的回传容量以及爆发式增长的信号发射造成的移动网络操作的成本。本文提出一种把软件定义的网络和无线资源虚拟化整合到基于LTE系统的车联网络，比如说

，**SoftdEfined heteRogeneous VehICular nEtwork**。基于此系统框架，我们提出了基于stochastic Learning的时延最优虚拟无线电资源规划方法。延迟优化问题可以被建模成一个无限维度的平均损耗的部分可观测马尔可夫决策链的过程。然后，使用一个等价的贝尔曼方程来解决这个问题。我们提出的方法可以分成两部分，宏观虚拟资源分配MaVRA以及微观虚拟资源分配MiVRA。前者基于大尺度的时间段单元（交通密度的）而使用，后者通过小尺度的时间段单元履行职能（信道状态 队列状态）。仿真的结果显示了我们提出的方法比传统方法更有效。

<!--more-->



## 引言

“链接的车辆”被视为能提高交通运势效率，保障交通安全，减少交通事故并降低交通拥塞影响的方法。智能交通系统通过V2V以及V2I的通信为交通安全与效率提供了支持，车辆的无线连接已经称为多方追逐社会与经济效益的目标。同时，大量的服务被设计来保证交通安全，提高交通质量并满足乘客娱乐需求（？），要做到这些就需要达到严格的服务质量QoS需求。通常，安全服务的最大容许延迟时间是需要小于100ms的，对于特殊服务，比如说碰撞预测警报，最大允许的时延是50ms，其他的非安全相关服务的最大允许时延时间是500ms。



多样的无线通信技术已经被考虑纳入支持ITS服务。1999年DSRC技术取得发展，它最广为人知的叫法是IEEE 802

.11p。DSRC从它的特性中受益许多，比如说易于开发，低成本，以及传播支持的原生信息的能力。然而扩展性问题、无边界的延迟以及缺少确定性QoS保障都极大地阻碍了其发展和DSRC系统的市场化。比如说，在[4]中，分析指出一种增强的分布信道接入EDCA在高密度的车联网场景下会受到严重影响。**[5]研究了切换CCH和SCH的方法的性能，并得出aware的应用如果使在UTC时间上是对齐的，那么出于改变信道的切换的原因，这些应用必须妥善处理。（？）**同时 由与DSRC 一开始设计之时是针对短距离通信的，并没由考虑到路边设施的需要大量的接入问题，因此它并不能提供广阔的通信覆盖范围和长久的V2I连接。再者，DSRC的部署是一个耗时耗财的工作，并且面临了鸡蛋和鸡的问题。一方面汽车制造商并不愿意投资开发安装车载设备（DSRC无线设备），因为没有任何东西保证他们制造的东西以后可以去连接。另一方面，当局也不想在毫无车载服务保证的情况下投入资金成本来建立供DSRC接入的设备。



同时，一些汽车制造商和通信运营商在考虑基于蜂窝网络的解决方法。比如说，LTE是最具潜力的无线技术，它可以对车载用户提供高速率的低延时的传输，同时他也满足了QoS对宽带的高要求的要给子类——信息娱乐，比方说社交网络。然而，LTE对车载安全应用的指出非常有限，并且网络几百年实在理想情况下也非常容易过载。因此，未来的车联网趋势已经从关注单一技术转向设计异构网络，比如说HetVNETs。



HetVNETS当前正处基础信息平台建设过程中，最终他将成为适配车联网的网络。未来的HetVNETS不仅仅可以支持大面积覆盖、高宽带以及非强时效性的车联网服务，同时也能保障安全服务的QoS需求。然而，HevNETS用于车联网服务依然面临几个问题。HetVNETs存在多种不同的无线技术，在传统的网络架构下这些技术很难相互连通，在设备中集成大量的无线技术是一个难题；再者，大量的无线网络设施和频谱资源会被浪费并且会导致车联网用户的糟糕的用户体验。因此，HetVNETs在车联网领域的发展需要一个新的架构。就如前文所提及的，一个成功的HetVNETs的架构必须达成以下严格的标准：

* 架构能扩展、改进，并且与新发展的技术能有效融合
* 在掌握架构配置和交互面的定义后，第三方公司可以开发新的尚未出现充满前景的应用



最近，大量的解决车联网面临的挑战创新想法被提出。一种紧急信息传播的时/地严格的框架被提出，且很好地适配了即时碰撞避让与路径规划功能。车联网使用的协同通信可以提高车联网的性能，V2V通信被用于中继V2I通信的信息。在[16]中，车联传感网络被用于监测城市交通，同时巡逻控制算法也被提出以提升检测性能，并为传感器网络设计了一种基于动态感知和路由的数据采集优化算法。然而，大多数的现有成果只能在特定场景下发挥作用。再者，随着5G技术的提升，软件定义网络可以被用于设计高效和拓展网络。因此为了满足需求，我们将SDN与虚拟无线资源集成到HevNETs中，在先前的论文中我们将它称为SERVICVE。在[12]中提到，大量的虚拟无线资源不能被充分地被利用以及大量的信号开销引入和浪费，导致严格的延迟要求不能被保障。为了解决这些问题，我们再者提出了一种时延最优的虚拟无线电资源规划方法，它分为两个部分：MaVRA和MiVRA。两者分别在大的时间段单元（比如说交通密度）和小的时间段单元（队列状态、信道状态）履行职能。他们的特点总结如下：

* 提出一个用于SERVICE的两步的延迟优化动态虚拟无线电资源优化方法，将大的时间单元以及小的时间单元作为变量进行考察分析
* 将时延最优的动态虚拟化无线电资源分配问题建模成受限于部分可观马尔可夫决策过程的时域无穷的的平均开销，并且通过使用贝尔曼方程解决该问题，同时：
* 通过对每一车辆势函数求和来拟合MaVRA选择Q-因子，和分布的在线随机学习算法评估每辆车的势函数，这样能减少大量的控制面上的信号收发。



马尔可夫决策过程是一种解决延迟优化的资源分配问题的系统建模理论，唯一需要考虑的问题就是由于维度的问题解决MDP没有简单的方法。最近MDP被用于解决各个领域的延迟优化无线电资源分配问题。比如说，在[21]中OFDMA中使用了一种时延最优的功率和子载波分配方法，将问题转为时域有限的马尔可夫决策过程的平均奖励问题。在MIMO系统网络中，双时间尺度的时延优化的动态集束和资源分配方法被应用在下行链路的传输过程中。在[23]中，端到端通信的时延最优模式选择和资源分配同样使用了MDP的理论。更有甚者，移动云计算的优化同样可以建模成MDP问题。在[24]中，提出了基于半MDP的移动云信息娱乐服务模型，旨在通过最小化服务概率来最大化云系统和用户的奖励。据我们所知，这是首次将交通行为纳入资源分配的马尔可夫决策过程中。



本文剩下部分结构如下。第二部分将给出SERVCICE架构的简要介绍，第三部分展示了时延优化问题的系统模型，第四部分展示了问题的建模。第五部分给出了一种低复杂度的时延优化问题的解决方案，第六部分则提供了两条baseline并与提出的MDP方法进行了性能对比，同时还有数值和模拟结果的讨论，最后则是结论部分。



## 软件定义网络

SDN已成为提升网络资源管理的网络性能的最有前景的一种方法。SDN的主要特征包括控制层和信息层的分离，控制层和数据层的开放接口以及集中的控制功能、可编程网络。基于可编程的SDN控制器，网络操作者可以轻松地定义新网络的设备配置并快速部署新应用。在这部分，我们给出SOTA的SDN，也就是我们之前的工作：SERVICE。



### SOTA

SDN的概念原先是基于UCB、斯坦福、CMU、普林斯顿大学的研究提出的Nicira Network。曾有人在\[25\][26]做过OpenFlow/SDN的全面调查，包括基础的概念，应用，虚拟化，QoS，安全性等等，展示了SDN/OpenFlow的最新成果。ONF主导SDN改进与SDN框架的基本元素（如OpenFlow协议）的标准制定。另外，基于OpenFlow的SDN被用于解决高带宽、动态特性，使网络适应不断变化的业务需求，并显著降低了运营和管理的繁琐复杂。在[28]中作者提出了融合了SDN和NFV技术的SDNFV以到达更深层次的SDN。大量新兴的通信范式，比如说社交网络、移动云计算、物联网、车联网给当前的LTE网络带来了挑战。我们受这些启发，面向5G的软件定义移动网络去中心化架构得以提出，它能提升系统性能以达到更加性能与扩展性。



### SERVICE的结构

![image-20220921162447674](https://s2.loli.net/2022/09/21/hfYW52qV1MEF3G9.png)

在这个部分我们将简述SERVICE的框架（详细的介绍可以看文献[12]）。据我们所知，这是首次给HetVNETs的全面SDN框架。作为一个软件定义的网络，SERVICE可以按逻辑分成三个层：网络设施层，控制层和应用层。各层的细节描述如下：

* 网络设施层：网络设施层位于SERVICE架构的最底层，对应SDN标准架构中的数据面。实际上，这一层主要由许多种不同的资源比如说通信资源，计算资源以及纯村资源。为了使所有的资源连通，需要高速率的回程链路并且可以受控制层的配置。
* 控制层：控制层位于网络架构的中层，它作为服务代理传达用户服务需求并控制，对虚拟资源进行资源控制，并未应用程序提供相关信息。由于车辆高机动性以及网络拓补的不稳定性，不同层间的信息交换和功能数量变得巨大，实时通信的性能难以保障。因此需要一种更有效的方式去保证SERVICE中的QoS服务，它可以对快速移动的车辆请求进行迅速响应。所以我们在SERVICE中提出了一种分层的控制层，它由主控制器（PCon）和次控制器（SCon）组成。
  * 主控制器：PCon位于SERVICE控制层的最顶层。PCON控制全局的SERVICE网络，因此PCON可以做出可行的优化决策。再者，PCon和SCon一样拥有直接和网络设施交互的功能。
  * 次控制器：SCon位于PCon之下，它充当一个局部控制实体，是一个重要的组件。它其中一个重要的功能就是确保低延时的安全服务QoS需求，同时Scon可以通过虚拟资源池控制服务区内（intra-SA）的资源。
* 应用层：应用层位于SERVICE架构的顶层。操作者可以通过设计不同的应用比如说动态资源分配、接入管理等功能配置并控制SERVICE。在这里我们以接入管理作为例子：
  * 接入管理：当接入管理应用运行时，SCon将监测网络负载和链路状态。一旦网络复杂超过了基站给定的某一等级，SCon将新的车辆接入请求转移到另一基站，这样不仅可以平衡负载同时也达到了现存车辆用户的QoS要求。



### Win-Lots机制

基于如上所述的架构，软件定义的虚拟基站（vBSs）可以被用于向车辆用户提供符合要求的服务。通常来说，有两种虚拟基站，宏小区型虚拟基站（MvBS）和微小区型虚拟基站（SvBS）。基于此，我们可以设计更高效的虚拟无线资源分配方法。核心思想就是只有负责维持可靠性与覆盖的MvBS可以提供控制信道的功能，SvBS只负责高速率传输以达到广控制和局部传输的效果，这也是Win-Lots的由来。换句话说，不必去处理一个用户在SA覆盖范围内的移动接入问题，这可能会打破信号覆盖和传输能力之间的瓶颈。低频率的无线带宽更适合MvBS使用，因为他们传播路径损耗低。另一方面，高频段资源更应该被分配用于SvBS。基于这一原则，可以谨慎地设计无线资源管理（RRM）应用以最小化传输延迟。



## 系统模型

在这一部分，我们将阐述SERVICE系统的拓扑结构，包括其物理层模型，突发源模型以及流量模型，然后对SERVICE框架的时延优化控制问题进行建模。



### 系统拓扑

考虑基于包含一个MvBS和$M$个SvBS的SERVICE的无线车联网络，如图2所示。假设每一个SvBS都具备一根天线。设所有的SvBNS节点为集合$\mathcal B=\{1,2,3\dots,M\}$，考虑每一个SvBS节点只能为最大为$N_{max}$台车辆设备（VEs）服务，并且将VEs集合中与第$b$个SvBS相连的VE记作$V_b$。![image-20220921165830086](https://s2.loli.net/2022/09/21/TW7BfgAZrtO4ojJ.png)





如图2所示，SERVICE控制器将$M$个SvBS虚拟化以覆盖整个路边区域。每个SvBS可以本地测量本地的CSI和QSI。SERVICE控制器可以从每个分散的SvBS中获取全局的QSI。然后，每个VE可以报告其信息，比如说速度，状态，通过MvBS传输到控制器，这意味着控制器可以获取每个SvBS的交通密度。在P-GW中，存在许多与在控制器覆盖范围内的全部VE的流通信息序列。控制器将从每个SvBS收集的信息转发到RRM应用中。基于转发的信息，时延优化的无线资源分配方案会反馈到相应的车辆。在这里我们给出了详细的交通和我通信行为模型的假设：



* 交通行为模型： 如图2所示，每个SvBS通信范围覆盖了长度为$d_C$长的子路段。假设车辆都处于畅通状态，在该条件下，根据文献p[30]，以下假设是合理的：

  假设1：每辆车的速度是独立且均服从正态分布的随机变量，该概率分布密度表述为：

  $$
  f_V(v)=\frac{\xi}{\sigma_V\sqrt{2 \pi}}e^{-(\frac{v-\bar V}{\sigma_V\sqrt{2}})^2},
  v\in[V_{min},V_{max}]
  $$

  其中，$\bar V$，$V_{max}$以及$V_{min}$分别为平均、最大、最小的车辆速度，$\sigma_V$是车辆速度的标准差，$\xi$是截断产生的归一化常数。

  假设2：车辆的到达时间间隔构成一系列独立且服从指数分布的随机变量，概率分布密度表示为：

  $$
  f_I(t)=\frac{1}{\mu}e^{-\frac{t}{\mu_v}},t\geq 0
  $$

  

  其中，$\bar V$，$V_{max}$以及$V_{min}$分别为平均、最大、最小的车辆速度，$\sigma_V$是车辆速度的标准差，$\xi$是截断产生的归一化常数：

  

  $$
  \rho_v=\mu_v\bar V^{-1}
  $$

  

  基于前面的假设，可以得到车辆位置的稳定状态服从一般分布【？感觉是均匀分布】：

  

  $$
  g_l(r)=\frac{1}{R},0\leq r\leq R
  $$

  在SvBS中的车辆数分布则服从：

  

  $$
  Pr(N_b=n)=e^{-\phi_b}\frac{\phi_b^n}{n!}
  $$
  
* 通信行为模型：为了简化通信行为模型，假设收端和发端的CSI处于完美状态。因此，车载设备（VE）$n$的最大可达速率可以被计算为：
  
  
  $$
  \mu_{b.n}=\sum_{k=1}^{N_F}\mu_{(b,n),k}=\sum_{k=1}^{N_F}s_{(b,n),k}\log(1+\mathrm{SINR}_{n,k})
  $$
  

  其中$\mathrm{SINR}_{n,k}=\mathrm{SINR}_{(b,n),k},n\in V_b$是信号在第$k$个虚拟无线资源块传输时第$b$个VE的**信号与干扰加噪声比（信噪比）**。
  
  
  
  我们假设每个VE拥有一个长度为$N_Q$的有限长队列用于缓存到达的包数据。当前场景下的所有VE的队列长度可以用$Q(t)=\{Q_{b,n}\}_{n\in V_b,v\in\mathcal B}$表示，并且队列的状态转换按如下公式进行：
  $$
  Q_{b,n}(t+1)=\min\{[Q_{b,n}(t)-\mu_{b,n}(t)\Delta T]^{+}+A_{b,n}(t),N_Q\}
  $$
  其中，$A_{b,n}(t)$是在第t个时间间隔中第$n$个VE与SvBS $b$相连的随机新到达的包，并将其表述为$A(t)=\{A_{b,n}(t):b\in \mathrm V_b,b\in\mathcal B\}$
  
  假设3：包到达过程$A_{b,n}$在时隙规划时是独立同分布的，并且在与SvBS和VE相关时他们之间是独立的。

### 虚拟无线资源管理策略

在第t个时隙开始之时，SERVICE的控制器通过获取得到的交通密度条件决定了所有SvBS的MaVRA。然后每个SvBS对其无线资源进行分配，这个过程被定义为MiVRA在给定观测的CSI和QSI后的操作。MaVRA的定义如下：

* ***定义1** MaVRA*：定义一个可行的MaVRA $V \in \mathcal V$是总的虚拟无线资源集合$\mathcal N_f$的一个子集，它的定义如下：
  $$
  V=\{v_b\subseteq \mathcal N_f:v_b \cap v_{b'}=\O \quad\forall b\neq b',\bigcup_{b\in\mathcal B}v_b=\mathcal N_f\}
  $$
  
  
  其中$\mathcal V$是所有可行的MaVRA的集合，$|\mathcal V| = I_V$，$|\mathcal N_f|=N_F$。
  
* ***定义2** 虚拟无线资源管理策略*：虚拟无线资源管理策略包含了MaVRA和MiVRA，举例来说，$\Omega=(\Omega_v, \Omega_s)$就是系统状态$\chi \in X$到宏观与微观无线资源分配动作的一个映射，其中$\Omega_v(\mathrm T)=\mathrm V\in\mathcal V$，$\Omega_s(\chi)=\{s_{(b,n),k}:b\in\mathcal B,n\in \mathrm V_b,k\in v_b\}$。$\mathrm T$是交通状态，代表了当前的交通密度。



如定义2所示，MaVRA是一个对交通密度的映射，这种映射仅为了如下原因而设立：首先，交通密度在时间尺度上与QSI和CSI相比是一个变化的量；其次，由于MaVRA是从SERVICE网络架构中以全局的视角进行功能执行的，所以需要在一个更大的时间尺度上进行决策，这不仅能降低应用与计算的复杂度，同时能减少大量的信号交互；最后，MiVRA在每个SvBS分布式地部署，他们产生的是局部的CSI和QSI。因此在考虑到过量的信号交互与计算复杂度后，可以短的时间单元内（通常是时隙级别）履行职能。

## 问题建模

### 动态系统状态转移

定义状态空间为$\chi(t)=(\mathrm T(t),\mathrm H(t), \mathrm Q(t))$，其中$\mathrm H(t)=\{|H_{(b,n),k}|:n\in\mathrm V_b,b\in\mathcal B\}$代表了系统每一个节点的CSI，$\mathrm T(t)=\{|T_b(t)|:b\in \mathcal B\}$是每一个节点的交通密度信息（TDI）。因此马尔科夫状态转移可表示为：


$$
\begin{align}
&\mathrm {Pr}[\mathrm Q(t+1)|\chi(t),\Omega(\chi(t))]\\\\
=&\mathrm {Pr}[\mathrm A(t)+\mathrm Q(t+1)-[\mathrm Q(t)-\mathrm R(t)\Delta T]^{+}]\\\\
=&\prod_{b\in\mathcal B} \prod_{n\in\mathrm V_b}\mathrm {Pr}[A_{b,n}(t)=Q_{b,n}(t+1)
-[Q_{b,n}(t)-R_{b.n}(t)\Delta T]^{+}]
\end{align}
$$


注意到$M\times N$的队列是通过控制策略$\Omega()$组合成的。引入的随机过程$\chi(t)=(\mathrm T(t),\mathrm H(t), \mathrm Q(t))$是遵循状态转移矩阵（见公式10）的马尔可夫链。



在公式10中，根据$\mathrm {Pr}[\mathrm H(t)]$和TDI的状态转移概率：


$$
\begin{align}
&\mathrm {Pr}[\mathrm T(t+1)|\mathrm T(t)]\\
=&\prod_{b\in \mathcal B}\mathrm {Pr}[T_b(t+1)|T_b(t)]
\end{align}
$$


我们假设信道状态服从一个离散的均匀分布。其中:


$$
\mathrm {Pr}[\mathrm T(t+1)|\mathrm T(t)]
=\begin{cases}&
\begin{align}
p_1=\frac{\Delta T}{\mu_v}(1-T_b(t)\frac{\Delta T \bar V}{R})\quad &T_b(t+1)=T_b(t)+1\\	
p_2=T_b(t)\frac{\Delta T \bar V}{R}(1-\frac{\Delta T}{\mu_v})\quad &T_b(t+1)=T_b(t)-1\\
p_3 = 1-p_1-p_2 \quad &T_b(t+1)=T_b(t)
\end{align}
\end{cases}
$$



### 问题重述

给定一个单链策略组$\Omega$，马尔可夫链是变脸的且拥有唯一的稳定状态分布$\pi_\chi$。因此，第b个SvBNS的平均成本为：


$$
\begin{align}
\bar D_b (\Omega)&=\lim_{T\rightarrow \infty}\frac{1}{T}\sum_{t=1}^{T}\mathbb E^{\Omega}[\sum_{n\in \mathrm V_b}f(Q_{b,n}(t))]\\
&=\mathbb E_{\pi_\chi}[\sum_{n\in\mathrm V_b}f(Q_{b,n})]
\end{align}
$$


在本文中，主要的目标是最小化传输延迟，因此我们设$f(Q_{b,k})=Q_{b,k}/\lambda_{b,k}$。现在，我们有：

* ***问题一 SERVICE网络的时延最优控制*** ：为区分不同的服务优先级，我们给每一个队列设定了权重因子$\beta = \{\beta_{b,n}>0:b\in\mathcal B,n\in \mathrm V_b\}$。时延最优的问题可以建模如下:

  
  $$
  \begin{align}
  \min_{\Omega}J_{\beta}(\Omega)&=\sum_{b\in \mathcal B}\bar D_b(\Omega)\\
  &=\lim_{T\rightarrow \infty}\frac{1}{T}\sum_{t=1}^{T}\mathbb E^{\Omega}[g(\chi(t),\Omega(\chi(t)))]
  \end{align}
  $$
  其中:
  $$
  g(\chi(t),\Omega(\chi(t)))=\sum_{b\in \mathcal B}\sum_{n\in \mathrm V_b}\beta_{b,n}f(Q_{b,n}(t))
  $$
  

### POMDP

根据公式12， 问题一是一个有限域中不受限的PODMP的平均成本问题。如前文所提，为了减少时间和空间复杂度，问题一的解决方法被分成了两部分：MaVRA和MiVRA。MaVRA基于系统的部分状态$\mathrm T$，后者则是在整个系统的状态$\chi=(\mathrm {T,H,Q})$上定义的。因此，POMDP的建模定义如下：

* 状态空间：$\{(\mathrm {T,H,Q}:\forall\mathrm T\in\mathcal T,\mathrm H\in\mathcal H,\mathrm Q\in\mathcal Q)\}$。

* 动作空间：$\{\Omega(\chi)=(\Omega_v,\Omega_s)\}$，是定义2中给出的一个单链可达策略。

* 状态转移矩阵：$\mathrm {Pr}[\chi'|\chi,\Omega(\chi)]$，在公式10中定义。

* 每步损失函数：$g(\chi(t),\Omega(\chi(t))) = \sum_{b\in \mathcal B}\sum_{n\in \mathrm V_b}\beta_{b,n}f(Q_{b,n}(t))$。

* 观测：MaVRA的观测是交通密度，比如说$o_v=\mathrm T$，而MiVRA的观测则是整个系统状态，比如$o_s=\chi$。

* 观测函数：对于MaVRA来讲，观测函数$O_v$是：
  $$
  O_v(o_v,\chi,(\Omega_v(\mathrm T'),\Omega_s(\chi')))=
  \begin{cases}
  \begin{align}
  1\quad\quad &o_v=\mathrm T\\
  0\quad\quad &otherwise\\
  \end{align}
  \end{cases}
  $$
  对于MiVRA来说，观测函数$O_s$是：
  $$
  O_s(o_s,\chi,(\Omega_v(\mathrm T'),\Omega_s(\chi')))=
  \begin{cases}
  \begin{align}
  1\quad\quad &o_s=\chi\\
  0\quad\quad &othersie
  \end{align}
  \end{cases}
  $$
  

因此对应问题1中的的无约束的POMDP可以表述为：
$$
J=\min_{\Omega}J_{\beta}(\Omega)
$$
最优的动作$\Omega^*$可以通过解贝尔曼方程得到。很不幸的是，POMDP通常是一个非常难解决的问题，如果直接获得其解析解需要大量的计算。因此，我们使用了一个等价的贝尔曼方程来获取最优的控制策略。



### 等价贝尔曼方程

首先先定义条件MiVRA动作集：

* ***定义3 条件MiVRA动作集*** 给定一个MiVRA的策略$\Omega_s$，我们定义条件无线资源分配动作集为$\Omega_s(\mathrm T)=\{s=\Omega_s(\chi):\chi=\mathrm{(T,H,Q),\forall H,Q}\}$作为给定$\mathrm T$时的所有可能CSI和QSI集合。

基于定义3，POMDP转换成馋鬼的有限域平均成本MDP问题。除此之外，最优控制策略$\Omega^{*}$可通过如下的等价贝尔曼方程求解得到。

* ***引理1 等价贝尔曼方程与MaVRA选择方法*** 问题1的时延最优化问题的最佳控制策略$\Omega^{*} = (\Omega_v^{*},\Omega_s^{*})$可以通过如下等价的贝尔曼方程求解得到：
  $$
  \begin{align}
  &\mathbb Q(\mathrm T,V)+\theta
  \\=&\min_{\Omega_s(\mathrm T)}\{\bar{g}(\mathrm T,V,\Omega_s(\mathrm T))
  \\&+\sum_{\mathrm T'}\mathrm{Pr}[\mathrm T'|\mathrm T,V, \Omega_s(\mathrm T)]\min_{V'}(\mathrm T',V')\},\forall\mathrm T \in \mathcal T, V\in \mathcal V
  \end{align}\\
  $$

​		其中$\mathbb Q(\mathrm T,V)$是MaVRA的选择的Q因子，$\bar g(\mathrm T,V,\Omega_s(\mathrm T))=\mathbb E[g(\chi,V,\Omega_s(\chi))|\mathrm T]$是每步执行的条件期望，$\mathrm {Pr}[\mathrm T'|\mathrm T,V,\Omega_s(\mathrm T)]=\mathbb E[\mathrm{Pr[\mathrm T'|\chi,\Omega_s(\chi)]}\mathrm T]$是条件转移概率的期望。问题1中的最优的平均损耗$\theta=\min_{\Omega}J_\beta(\Omega)$必须满足等式16所指出的贝尔曼方程。通过最小化最小化等式16的右侧并求$\Omega_v^{*}(\mathrm T)=\arg \min \mathbb Q(\mathrm T,V)$即可得到最优的控制策略。



​		等式16提出的等价贝尔曼方程只考虑交通密度信息$\mathrm T$。通过解等式16的方程，可以求得最优的MiVRA策略$\Omega_s^{\star}=\cup_{\mathrm T}\Omega_s^{\star}(\mathrm T)$，其中的$\Omega_s^{*}$已经在定义3中给出。然而这个定义可以对全局的CSI $\mathrm H$以及QSI $\mathrm Q$同样适用，因此，在下一部分，我们将使用线性拟合和随机学习来减少计算的复杂度。



## 一般的低计算复杂度的解

直接的暴力计算需要考虑指数级别的信道状态数和队列状态数的暴增带来的问题。而且由于部署位置位于整个系统的中心位置，因此大量的数据会不断回传。在这一部分，我们使用线性拟合和随机学习的分布式方法来估计的势函数。

### 线性拟合MVRAS选择Q因子

记$T_b\in\mathrm T$为第b个SvBS的交通密度值。为了减少状态空间的维数以及计算复杂度，同时为了把资源分配问题去中心化，我们通过线性累加每个SvBS的函数$q_b(T_b)$，如下所示：
$$
\mathbb Q(\mathrm T,V)\approx\sum_{v_b\in V}q_b(T_b)
$$
第b个SvbS的势函数$q_b(T_b)$满足如下贝尔曼等式：
$$
\begin{align}
&q_b(T_b)+\theta_b\\
=&\min_{\Omega_{s_b}(\mathrm Q_b)}\{\bar g(T_b,\Omega_{s_b}(T_b))\\
&+\sum_{(T_b,\mathrm Q'_b)}\mathrm{Pr}[T_b'|T_b,\Omega_{s_b}(T_b)]q_b(T_b)
\}
\end{align}
$$

其中：
$$
\bar g(T_b,\Omega_{s_b}(T_b))=\mathbb E[\sum_{n\in\mathrm {V_b}}\beta_{b,n}f)Q_{B,n}],
$$

$$
\mathrm {Pr}[T_b'|T_b,\Omega_{s_b}(T_b)]\\
=\mathbb E[\mathrm{Pr}[T_b'|\chi_b,\Omega_{s_b}(\chi_b)]|T_b]
$$

使用如上的线性拟合，SERVICE的控制器即可根据观测到的交通密度信息$\mathrm T$做出最优的MaVRA策略决策$\Omega_v^*$:
$$
\Omega_v^*(\mathrm T)=\arg \min_{V}\sum_{v_b\in V}q_b(T_b)
$$
基于每个SvBS观测得到的CSI和QSI，每个SvBS通过最优决策$\Omega_{s_b}^*(T_b)=\{\Omega_{s_b}(\chi_b):\chi_b \forall(\mathrm{H_b,Q_b})\}$来最小化等式18。随后我们即可得到全局的无线资源分配方法：
$$
\Omega^*(\chi)=(\Omega_v^*(\chi),.\Omega_s^*(\chi))=(\Omega_v^*(\chi),\{\Omega_{s_b}^*(\chi):b\in \mathcal B\})
$$


### 每一SvBS势函数的分解

每个SvBS的势函数$q_b(T_b)$可分解成每一车辆（或队列）势函数$q_b(T_b)=\sum_{n\in\mathrm V_b}q_b^n(T_b,\mathrm H_{b,n},Q_{b,n})$，这些势函数必须满足等式22，其中$\bar g_{b,n}(T_b,\Omega_{s_b},(T_b,Q_{b,n}))=\mathbb E[\beta_{b,n}f(Q_{b,n})]$，系统的状态转移如式23所示。



优化的目标是最小化等式22迭代中的R.H.S，然后让其变成等式24。



***引理2*** 最优MiVRA的闭式解：基于如上的分析，在给定系统状态$\chi_b=(T_b,\mathrm H_b, \mathrm Q_b)$和控制策略（式-18）下的MiVRA最优解如下：
$$
s_{(b,n),k}=
\begin{cases}
\begin{align}
1,&\quad X_{(b,n),k}=\max_n\{X_{(b,n),k}\}\\
0,&\quad otherwise
\end{align}
\end{cases}
$$
其中：
$$
X_{(b,n),k}=\frac{\Delta T}{\bar N_{b,n}}(q^n(T_b,Q_{b,n}-1)-q^n(T_b,Q_{b,n}))\log(1+PH_{(b,n),k})
$$


### 随机拟合的在线分布式学习算法

在这个模块，我们使用了在线学习算法来预估势函数$q_b^n(T_b,Q_{b,n})$。$q_b^n(T_b,Q_{b,n})$的迭代公式在***算法1***中给出。

### 计算复杂度与信号分析

在这个部分， 我们研究了暴力求解的计算复杂度以及信令开销。对于暴力求解的方法，由于是在集中模式进行处理，这就需要获取全局的CSI、全局的QSI和TDI。暴力求解的过程包含了解一个系统的非线性贝尔曼方程$I_V(N_Q+1)^{MN_{\max}}$。然而使用随机近似的方法时，我们可以得到一个低复杂度的解，它的时间复杂度是$\mathcal O(N_F(N_Q+1))$以及$\mathcal O((N_{Q}+1)(MN_{\max}))$的空间复杂度。除此之外，CSI和QSI的信息并不需要上传到SERVICE框架的应用层，因为它运行于分布模式，这能大量地减少信令开销。



## 仿真结果与分析

在这一部分，我们展示了一些仿真的结果，并评估了我们提出的虚拟无线资源分配方法的性能。为了证明我们的方法更加有效，我们还使用了两个参考的方法与我们的方法进行了对比。

### 仿真实验设置

考虑一个长度为400米的高速公路的仿真场景。仿真实验的主要的参数以及配置已经列在了表格1中。在仿真中，有两个覆盖半径为200米的SvBS部署在路段。该仿真场景使用了Free-Flow模型，这个场景下每辆车不之间没有交互。车辆到达时间间隔服从等式2的指数分布，汽车的每秒的到达率$\mu_v$为取值为$[0.2,1.0]$。每辆车的速度服从等式2的截断正态分布，其中均值为$V=30 m/s$，最大速度为$V_{\max}=50m/s$，最小速度为$V_{\min}=10m/s$。我们假设每辆车辆在路段都保持其初始速度。

![image-20220924095830480](https://s2.loli.net/2022/09/24/BdkJlFwh5HrQf6j.png)



车辆和SvBS的通信使用的是LTE-Advanced系统。在仿真中，无线虚拟资源块被设置成10，对应的是5MHZ的带宽。SvBS维持一个缓存对应车辆的到达数据包，其长度为$Q_{\max}=10$。报数据大小均值为50k比特，包达到速率为20个/s~120ge /s。所有的车辆都假设拥有过独立同分布的无线信道，并且信噪比的均值为12dB。为了对比性能，我们使用了如下两个方法作为参照：

* ***基准1 Macro Resource Equipartition & Round Robin (MaRE&RR)方法***： 这个方法中宏观的虚拟无线资源被平均地分配到每一个SvBS。基于这种分配方式，车辆能适应Round Robin 的方法，这意味着每辆车都有均等的使用无线资源的机会。
* ***基准2  Macro Resource Equipartition & Channel Precedence (MaRE&CP)方法***： 宏观的虚拟无限资源分配方法如基准1，二者的唯一区别在于微观无线资源的分配方式，SvBS是通过信道的条件优劣为车辆分配无线资源。

### 结果分析

图3、4给出了主要性能的对比。图3展示了每辆车在不同的到达率时的平均包延迟。随着车辆到达率的增加，所有方法的包延迟都在上升。基准2的方法（MaRE&CP）比基准1（MaRE&RR）的效果好，因为MaRE&CP方法更倾向于选择更好信道条件的车辆。我们提出的方法都比这两种方法效果要好，因为其根据即时的交通情况动态的分给每个SvBS宏观虚拟无线资源。比如说，我们的方法和MaRE&RR相比可以减少接近30%的平均延迟（包速率和车辆到达率分别为60/s和1/s）.最终，性能增益随着车辆到达率的增加而明显获得增幅，这是由于当交通密度较低时，无线资源对传输数据的支持都非常有效。因此，所有方法在车辆到达率较低时都能有较好的性能表现。

![Fig.3  Average delay of total network versus different vehicle arrival rate.](https://s2.loli.net/2022/09/24/9MPw8Y7HIAbzaXD.png)![Fig. 4. Average delay of total network versus different data packet arrival rate.](https://s2.loli.net/2022/09/24/JL8k6MH7CQoyXvI.png)

图4展示了在不同数据包速率情况下不同算法的性能表现对比。每个SvBS的车辆到达率设成了低密度（每秒1辆车），数据到达率为20~120每秒。可以看到与图3近似的曲线，随着数据包到达率的增加，性能增益得益于宏无线资源得到提升。然而，当数据包到达率较低时，可能会有相反的情况出现。

![Fig. 5. Average delay of total network under different data packet arrival rates and vehicle arrival rates.](https://s2.loli.net/2022/09/24/RJxB8aoNlPnVXeZ.png)![ig. 6. Illustration of convergence property of the proposed scheme via stochastic approximation.](https://s2.loli.net/2022/09/24/UZYAVRLPKBkH26j.png)

图5给出了在动态资源达到率和汽车到达率下的三维的平均时延图。可以看出平均延迟的增加受数据包到达率和车辆到达率的增加而增加。而且数据包到达率的增加对平均时延的影响比车辆到达率的影响更大。

图6表示MDP方法打的收敛性，给出了长度为10的队列状态$\{q_b^n(Q)\}$平均的势函数以及时隙槽的关系图。可以看到我们给出的方法收敛速度很快，而且随着队列状态空间越大，能更快收敛。这种现象的原因是和图六中所给出的在仿真条件下小长度序列拥有更多状态空间的队列的更新频率更快。



## 结论

本文提出了一种在SERVICE框架下的两步时延最优虚拟无线资源分配方法。这种方法可以建模成有限域中的平均成本POMDP问题，通过解一个等价的贝尔曼方程即可求解。为了减少计算复杂度以及信令开销，使用了1)分成MaVRA以及MiVRA两个从不同时间层面决策的决策步；2）通过累加每个用于估计系统势函数SvBS的势函数进行线性拟合，大量地减少了系统空间的维数，并且使用了在线分布学习的分布式算法去估计每个车辆的势函数，最终，我们证明了我么牛的方法可以收敛并且达到了一个不错的性能增益效果。
