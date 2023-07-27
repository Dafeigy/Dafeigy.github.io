---
title: CMA均衡Matlab仿真实验
mathjax: true
tags:
- CMA
- 自适应均衡
- Matlab
categories:
- 通信技术
- 学习记录
---



# CMA均衡算法实验

均衡是为了弥补信号在非理想信道下传播时对波形造成的失真进行“修补”，假如把信道输入到非理想信道再到信号输出的过程看作时域下信号与一个系统的系统函数卷积再加上特定的噪声的结果，那么均衡所做的就是拟合这个系统函数“求逆”的结果，使得经过信道的信号在经过这个“求逆”的操作后恢复原本的信号波形。大部分情况下，信道情况是未知的，在这种情况下对输出信道进行均衡的话有主要两种方法：依赖辅助数据进行信道估计和盲均衡方法。依赖辅助数据的方法就是在正式发送数据前先发送一段训练序列，通过已知的训练序列对比在接收端经过信道后的序列进行对比从而对信道进行估计，这种方法应用很广，在资源充足的情况下可以获得较好的效果，OFDM技术中长短训练序列的使用正是这一思想的体现：

![image-20221025191837355](https://s2.loli.net/2022/11/12/VNzIbeLtRpE9k7W.png)

另一种方法则是不依赖训练序列的均衡方法，它靠接受到的信号进行自适应均衡，这种不依赖训练序列的方法称为盲均衡。下面的CMA（Constant Modulus Algorithm）即恒模算法就是隶属于Bussgang类盲均衡算法中的一种。CMA算法具有计算复杂度低，易于实时实现，收敛性能好等优点，代价函数只与接收序列的幅值有关，而与相位无关，故对载波相位不敏感。

<!-- more -->

## CMA均衡原理

一般的均衡过程的原理框图如下所示：

![image-20221026101739233](https://s2.loli.net/2022/10/26/JX9G6uVkP4jMxvq.png)

<center><small>图1. 均衡流程</small></center>

均衡器的输入信号可表示为：
$$
y(n) = x(n)*h(n) + noise(n)
$$
其中，$x(n)$是信源信号，$h(n)$是信道的系统函数，$noise(n)$为高斯信道白噪声。均衡的输出为：
$$
\hat x(n) = w(n)*y(n)\\
$$
均衡的目标即为通过改变滤波器参数组让输出的序列$\hat x(n)$无限逼近原始的信息序列$x(n)$。一般来说如果要估计信道的系统函数，我们可以通过给定训练序列作为$x(n)$输入，观察最终的$\hat x(n)$即可通过一系列的方法对$h(n)$进行估计$\hat h(n)$，从而设计恰当的FIR滤波器充当均衡器。

<img src="https://s2.loli.net/2022/10/26/m9nIiloRDadPq4p.png" alt="image-20221026101835743" style="zoom: 50%;" />

<center><small>图2. 均衡器等价构造：一个有限长的FIR滤波器</small></center>

然而CMA算法不需要训练数据作为依赖, 它的本质是通过误差函数来迭代FIR滤波器的参数组$W_n$。滤波器一般用长度为$L$的FIR滤波器实现，其构成参数（抽头系数）$w(n)$为：
$$
W(n) = [w_0(n),	w_1(n),\dots,w_{L-1}(n)]
$$
输入均衡其的符号序列表示为：
$$
Y(n) = [y(n),y(n-1),\dots,y(n-L+1)]
$$
定义CMA算法的代价函数：
$$
J(W_n)=[|(\hat x(n)|^2-R_2)^2]
$$
其中$R_2$代表了期望的收敛半径，是一个依赖于信源序列高阶统计量的一个实常数，定义为：
$$
R_2 = \frac{E[|\hat x(n)|^{2p}]}{E[|\hat x(n)|^p]}
$$
其中，$p$由信源决定，对于在复平面对称的4QAM信号p取值为2。我们的目标是找到最适合的$W_n^\star$使得代价函数的值最小，即：
$$
\begin {align}
W_n^{\star}&= \arg \min_{\{W_n\}} (J(W_n))\\
&=\arg \min_{W_n}(E[(|\hat x(n)|^2-R_2)^2])
\end{align}
$$

求解上式可以使用梯度下降法，即对$J(W_n)$计算其关于$W_n(n)$的偏导，即可得知$W_n(n)$中的每一组分对$J(W_n)$大小的影响。通过计算当前损失函数的偏导与设定的$\mu$值相乘的结果，再用当前的$W_n(n)$与该值相减即可得到下一时刻梯度变化最大的$W_n(n)$，通过不断地迭代即可得到最优的$W_n$，此时偏导数部分几乎不再变化，参数组也随之收敛：
$$
W(n+1) = W(n)-\mu\frac{\partial J(W(n))}{\partial W(n)}
$$
将损失函数表达式代入上式，即可得到$W(n)$的迭代更新式：
$$
W(n+1)=W(n)+\mu \hat x(n)[R_2-|\hat x(n)|^2]Y^\star(n)
$$

## CMA仿真实验

### 使用CMA算法均衡4QAM信号

采用4QAM对原始的`3000`个比特信息流进行调制，将调制后的信号经过参数矩阵为$H$的滤波器以拟合信号在非理想信道下传播时的码间串扰，随后将输出与信噪比为`snr`的高斯加性白噪声叠加作为均衡器输入的信号。对均衡器而言，设定三个不同的参数$\mu_1=0.01,\mu_2=0.005,\mu_3=0.001$，并迭代5000轮观察该参数对均衡结果的影响。在4QAM调制中，采用MSE观察均衡结果。在该实验下，MSE定义为损失函数的平方：
$$
MSE = (R_2-|\hat x(n)^2|)^2
$$
原始的比特流经由4QAM调制后的星座图与经过信道与高斯加性白噪声（snr=10）叠加后的星座图：

<center><img src="https://s2.loli.net/2022/10/26/1l2FQMU4WsTLqyk.png" style="zoom:40%;" /><img src="https://s2.loli.net/2022/10/26/8m17WDoZJGRwVdQ.png" style="zoom:40%;" /></center>

经过5000次迭代后，不同$\mu$值下的星座图：

<center><img src="https://s2.loli.net/2022/10/26/qMN3KhyAQaWJpT1.png" alt="Equ-0.01" style="zoom:40%;" /><img src="https://s2.loli.net/2022/10/26/3fAxmcs78lIB5rn.png" alt="equ 0.005" style="zoom:40%;" /><img src="https://s2.loli.net/2022/10/26/2XWxVnmLbcQSioF.png" alt="equ 0.001" style="zoom:40%;" /></center>

<center>SNR=20dB下均衡后信号图，对应μ值分别为0.01，0.005和0.001</center>

可以看到原本分散的符号聚拢在原始星座图的四个点附近。在信噪比较差(SNR=5dB)时，结果如图所示：

<center><img src="https://s2.loli.net/2022/10/26/1l2FQMU4WsTLqyk.png" style="zoom:40%;" /><img src="https://s2.loli.net/2022/10/26/Xuktc2N4jAmDfJ9.png"  style=zoom:40%;/></center>

均衡后的星座图如下所示：

<center><img src="https://s2.loli.net/2022/10/26/t3fnX6slwvGE5LF.png" alt="Equ-0.01" style="zoom:40%;" /><img src="https://s2.loli.net/2022/10/26/Sn4VjB73wkQL9Ns.png" alt="equ 0.005" style="zoom:40%;" /><img src="https://s2.loli.net/2022/10/26/dCPSFRunwfiNshe.png" alt="equ 0.001" style="zoom:40%;" /></center>

<center>SNR=5dB下均衡后信号图，对应μ值分别为0.01，0.005和0.001</center>

两种信噪比下，迭代次数与MSE关系对比如图：

<img src="https://s2.loli.net/2022/10/26/QOwoT8fS26HNmCs.png" alt="image-20221026102630396" style="zoom: 67%;" />

<center>SNR=20dB时，不同μ值的MSE情况</center>

<img src="https://s2.loli.net/2022/10/26/bmgzxMkV1N8dSsF.png" alt="image-20221026102648393" style="zoom:67%;" />

<center>SNR=5dB时，不同μ值的MSE情况</center>

细节图：

<center><img src="C:\Users\cybercolyce\AppData\Roaming\Typora\typora-user-images\image-20221025205942835.png" style="zoom: 30%;" /><img src="C:\Users\cybercolyce\AppData\Roaming\Typora\typora-user-images\image-20221025210130389.png" style="zoom:30%;" /></center>

<center>左：SNR 20dB；右：SNR 5dB</center>

可知$\mu$的选择对MSE影响有重要影响。图中黄色曲线代表$\mu_2 = 0.05$的迭代情况，绿色曲线代表$\mu_3=0.001$的迭代，红色曲线为$\mu_1=0.01$，尽管三条曲线都在迭代初期已有下降趋势，但可以从细节图中看出，MSE下降最快的时$\mu_2=0.005$的黄色曲线，绿色曲线尽管也能收敛，但其收敛速度不如黄色曲线，红色曲线的来回震荡则说明参数设置的不合理，灵敏度过大以至于不能落入梯度为0的位置；同时也可从图中看出均衡对信号的修复能力也是有限的，在低信噪比情况下尽管恰到的学习参数可以让参数收敛，但其收敛效果非常有限，如20dB的信噪比下MSE收敛可到0.005，但在5dB信噪比下收敛的MSE只能到达0.6左右。

### CMA算法对原始比特流波形影响

对于信号的原始比特流信息，经过调制后不好直观展示，因此使用OOK调制将比特映射到集合{-1，1}上，设定系统传输环境如下：

* 码元速率：10e9
* 码元抽样点数（过采样次数）：8
* 码元个数：256
* 调制方式：OOK调制
* 判决门限：0.5

让信号序列经过高斯加性白噪声信道并经过均衡，在最终判决得到如下结果：

![image-20221025213953375](C:\Users\cybercolyce\AppData\Roaming\Typora\typora-user-images\image-20221025213953375.png)

可以从图中看出均衡后的序列可还原出原本的信号波形，但会有一定的时滞。

## 附录

### 实验一代码

```matlab
% QAM_main.h
clear all;
clc;
M=4;        % Modulation Scale
n=5000;     % Num bits
m=3000;     % Num Iteration

% Channel params, change value or length whatever you like
H=[0.005 0.009 -0.024 0.854 -0.218 0.049 -0.016];   %That's quite a bad channel
snr = 5;   % SNR of awgn that need to be added after H(n)*x(n)
L=7;       % CMA FIR scale, need to be set as an odd num

% \mu rate
u1=0.01;    
u2=0.005;
u3=0.001;

% mse buffer
mse_av1=zeros(1,n-L+1);
mse_av2=mse_av1;
mse_av3=mse_av2;

for j=1:m
    % Generate Source bit
    s=randi([0 M-1],1,n);
    % QAM modulation
    qam_msg = qammod(s,M);
    % Calculate R2
    R2=(abs(qam_msg).^4)/(abs(qam_msg).^4);

    % Go through FIR to approximate transmit channel's ISI and add AGWN
    s2=filter(H,1,qam_msg);
    x=awgn(s2,snr,'measured');
    
    %Init the W_n
    w1 = zeros(1,L);
    w1((L+1)/2) = 1;
    w2=w1;
    w3=w1;
    for i=1:n-L+1
        % Invert the input series for calculate discrete convolution
        y=x(i+L-1:-1:i);
        z1(i)=w1*y';
        z2(i)=w2*y'; 
        z3(i)=w3*y';

        % Calculate Loss Function
        e1=R2-(abs(z1(i))^2); 
        e2=R2-(abs(z2(i))^2);
        e3=R2-(abs(z3(i))^2);

        % Update FIR params
        w1=w1+u1*e1*y*z1(i);
        w2=w2+u2*e2*y*z2(i);
        w3=w3+u3*e3*y*z3(i);

        % Calculate MSE
        mse1(i)=e1^2; 
        mse2(i)=e2^2;
        mse3(i)=e3^2;
    end;
    mse_avl=mse_av1+mse1; 
    mse_av2=mse_av2+mse2;
    mse_av3=mse_av3+mse3;
end;
mse_av1=mse_av1/m;
mse_av2=mse_av2/m;
mse_av3=mse_av3/m;


plot([1:n-L+1],mse_avl,'r',[1:n-L+1],mse_av2,'y',[1:n-L+1],mse_av3,'g');
legend('u1=0.01','u2=0.005','u3=0.001')
title('MSE vs. Iteration Using CMA');
xlabel('Iteration','fontsize',14);
ylabel('MSE','fontsize',14);
hold on

scatterplot(qam_msg);
title("Origin 4QAM");
hold on

scatterplot(x);
title("Noise 4QAM");
hold on

scatterplot(z1);
title("Equalized 4QAM --u -0.01");
hold on

scatterplot(z2);
title("Equalized 4QAM --u -0.005");
hold on

scatterplot(z3);
title("Equalized 4QAM --u -0.001");
hold on

```

### 实验二代码

```matlab
% CMA.m
function [y,epsilon,H_matrix]=CMA(x,M,mu)
% y: 输出信号
% e: 误差输出 
% w: 最终滤波器系数
 
% 输入参数：
% x: 输入信号
% M：滤波器长度
% mu: 因子

% FIR 系数
H_matrix = zeros(1,M);
H_matrix((M+1)/2) = 1;%H_matrix((M+1)/2) = 1;
% 输入向量长度
N=length(x);
% 执行CMA
m = 1;
for n = M:2:N 
    % 倒序输入
    filter_in = x(n:-1:n-M+1);
    % 计算输出
    y(m) = H_matrix*filter_in;
    % 误差计算
    epsilon(m) = y(m)*(1 - abs(y(m))^2);
    % 滤波器系数更新
    H_matrix = H_matrix + mu*epsilon(m)*filter_in';
    m = m + 1;
end
end

%CMA_OOK.m
% 参数初始化
clear all;
clc;
Ts = 10e-9; % 码元周期，以秒为单位，倒数为传输速率
N_sample = 8; % 单个码元抽样点数
dt = Ts / N_sample; % 抽样时间间隔
N = 2^8; % 码元数
t = 0 : dt : (N * N_sample - 1) * dt; % 序列传输时间
dt1 = Ts; % 抽样时间间隔
t1 = 0 : dt1 : (N-1) * dt1; % 序列传输时间
gt0 = -ones(1, N_sample);% NRZ
gt1 = ones(1, N_sample); 

% 生成随机序列
RAN = randi([0 1],N,1);
figure(1);
subplot(5,1,1);
plot(t1,RAN);
axis([0,1000*dt,-1.5,1.5]);
title('原始随机序列');

% 8倍过采样序列
se1 = [];
for i = 1 : N
   if RAN(i)==1
       se1 = [se1 gt1];
   else
       se1 = [se1 gt0];
   end
end
subplot(5,1,2);
plot(t,se1);
axis([0,1000*dt,-1.5,1.5]);
title('8倍过采样序列');

% 通过高斯信道
noise = 0.2*randn(1,length(se1));
se1_addnoise = se1 + noise;
subplot(5,1,3);
plot(t,se1_addnoise);
axis([0,1000*dt,-1.5,1.5]);
title('加噪声后序列');

% 均衡
resample_data = se1_addnoise(1:4:end); % 数据降为2倍过采样
M = 15;
mu = 0.001;
R = 1;
[y,e,w] = CMA(resample_data',M,mu);
subplot(5,1,4);
plot(t1(1:end-7),y);
axis([0,1000*dt,-1.5,1.5]);
title('均衡后序列');

% 判决
y(y >= 0.5) = 1;
y(y <= -0.5) = 0;
result = [y RAN(1:N-7)'];
subplot(5,1,5);
plot(t1(1:end-7),y);
axis([0,1000*dt,-1.5,1.5]);
title('判决后序列');
```

