---
title: 在Ubuntu20.04 LTS中配置USRP B210
mathjax: true
tags:
- SDR
categories:
- 学习记录
---




从实验室白嫖了SDR，是USRP B210，准备用作UE放在Ubuntu搭建的核心网里，记录一下配置的过程。

<!-- more -->

## 软硬件平台

![image-20220920160939401](https://s2.loli.net/2022/09/20/Q1amF49NGo5n76H.png)

经测试本文内容在：

* Ubuntu 20.04 LTS
* USRP B210

可以成功运行。

## UHD依赖安装

先更新软件索引：

```bash
sudo apt-get update
```

随后安装依赖：

```bash
sudo apt-get -y install git swig cmake doxygen build-essential libboost-all-dev 
libtool libusb-1.0-0 libusb-1.0-0-dev libudev-dev libncurses5-dev libfftw3-bin 
libfftw3-dev libfftw3-doc libcppunit-1.15-0 libcppunit-dev libcppunit-doc ncurses-bin cpufrequtils python-numpy python-numpy-doc python-numpy-dbg python3-scipy python-docutils qt4-bin-dbg qt4-default qt4-doc libqt4-dev libqt4-dev-bin python-qt4 python-qt4-dbg python-qt4-dev python-qt4-doc python-qt4-doc libqwt6abi1 libfftw3-bin libfftw3-dev libfftw3-doc ncurses-bin libncurses5 libncurses5-dev libncurses5-dbg libfontconfig1-dev libxrender-dev libpulse-dev swig g++ automake autoconf libtool python-dev libfftw3-dev libcppunit-dev libboost-all-dev libusb-dev libusb-1.0-0-dev fort77 libsdl1.2-dev python-wxgtk3.0 git libqt4-dev python-numpy ccache python-opengl libgsl-dev python-cheetah python-mako python-lxml doxygen qt4-default qt4-dev-tools libusb-1.0-0-dev libqwtplot3d-qt5-dev pyqt4-dev-tools python-qwt5-qt4 cmake git wget libxi-dev gtk2-engines-pixbuf r-base-dev python-tk liborc-0.4-0 liborc-0.4-dev libasound2-dev python-gtk2 libzmq3-dev libzmq5 python-requests python-sphinx libcomedi-dev python-zmq libqwt-dev libqwt6abi1 python-six libgps-dev libgps23 gpsd gpsd-clients python-gps python-setuptools
```

## 下载编译UHD v3.15

### 下载源码

先创建`UHD`文件夹作为UHD源码存放地：

```bash
cd
mkdir UHD
cd UHD
git clone https://github.com/EttusResearch/uhd
```

随后进入文件夹并检出`v3.15`版本的UHD：

```bash
cd uhd
git tag -l	#查看标签
git checkout v3.15.0.0	#检出标签
```

### 编译源码

```bash
cd host #该指令执行前目录应该为 UHD/uhd
mkdir build
cd build
cmake ../
make	
```

### 安装并验证UHD

验证编译结果可使用如下命令：

```bash
make test
```

输出内容大致如下：

```bash
Running tests...
Test project /home/zxhui/workspace/uhd/host/build
      Start  1: addr_test
 1/50 Test  #1: addr_test ........................   Passed    0.05 sec
      Start  2: buffer_test
 2/50 Test  #2: buffer_test ......................   Passed    0.03 sec
      Start  3: byteswap_test
 3/50 Test  #3: byteswap_test ....................   Passed    0.01 sec
      Start  4: cast_test
									... 
									...
                                    ...
      Start 50: paths_test
50/50 Test #50: paths_test .......................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 50

Total Test time (real) =   0.20 sec
```

如果测试全通过了则编译成功。下面进行安装：

```bash
sudo make install
sudo ldconfig
```

随后修改`$HOME/.bashrc`文件，在最后一行添加环境变量：

```bash
export LD_LIBRARY_PATH=/usr/local/lib
```

然后下载UHD FPGA的镜像文件：

```bash
sudo uhd_images_downloader
```

随后将USRP通过数据线与Ubuntu的USB接口连接，并在当前终端输入：

```bash
sudo uhd_find_devices
```

如果安装成功的话应该会有设备信息的显示：

![image-20220920152330263](https://s2.loli.net/2022/09/20/eIKa1xmqSO8G6ct.png)
