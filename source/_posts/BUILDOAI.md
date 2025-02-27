---
title: OAI gNB构建
mathjax: true
tags:
- OAI
categories:
- 学习记录
---


在Ubuntu 20.04中搭建OAI平台。

<!-- more -->

## 前置条件

先安装依赖包：

```bash
sudo apt install net-tools
sudo apt install git
sudo apt install -y libboost-all-dev libusb-1.0-0-dev doxygen python3-docutils python3-mako python3-numpy python3-requests python3-ruamel.yaml python3-setuptools cmake build-essential
```

随后下载UHD镜像：

```bash
git clone https://github.com/EttusResearch/uhd.git ~/uhd
cd ~/uhd
git checkout v4.4.0.0
cd host
mkdir build
cd build
cmake ../
make -j $(nproc)
make test # This step is optional
sudo make install
sudo ldconfig
sudo uhd_images_downloader
```

下载完成后，接入USRP设备后可通过`uhd_find_devices`以及`uhd_usrp_probe`验证。

## 构建OAI gNB

```bash
# 获取源代码
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g
git checkout develop

# 安装OAI依赖
cd ~/openairinterface5g/cmake_targets
./build_oai -I

# 构建OAI
cd ~/openairinterface5g
source oaienv
cd cmake_targets
./build_oai -w USRP --ninja --gNB --build-lib "nrscope" -c
```

## 运行OAI gNB

根据USRP的板型号的差异，选择不同的配置。

### USRP B210

```bash
cd ~/openairinterface5g
source oaienv
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --sa -E --continuous-tx
```

### USRP N300

```bash
cd ~/openairinterface5g
source oaienv
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band77.fr1.273PRB.2x2.usrpn300.conf --sa --usrp-tx-thread-config 1
```

### USRP X300

```bash
cd ~/openairinterface5g
source oaienv
cd cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band77.fr1.273PRB.2x2.usrpn300.conf --sa --usrp-tx-thread-config 1 -E --continuous-tx
```

# 

