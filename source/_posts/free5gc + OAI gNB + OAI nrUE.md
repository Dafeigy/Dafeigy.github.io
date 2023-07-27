---
title: free5gc+OAI gNB + OAI nrUE
mathjax: false
tags:
- OAI
- 5G
- 核心网
categories:
- 学习记录
---

一台Ubuntu运行free5gc充当核心网，另一台Ubuntu主机运行OAI gNB和模拟UE。在Ubuntu 20.04.6LTS版本可行。

<!-- more -->

## Free5gc

### Free5gc安装

Free5gc的参考文档:[Home · free5gc/free5gc Wiki (github.com)](https://github.com/free5gc/free5gc/wiki)

#### 前置条件

##### Go 安装

Free5gc使用Go 1.17.8构建，因此需要先安装该版本的Golang：

```bash
wget https://dl.google.com/go/go1.17.8.linux-amd64.tar.gz
sudo tar -C /usr/local -zxvf go1.17.8.linux-amd64.tar.gz
mkdir -p ~/go/{bin,pkg,src}
# The following assume that your shell is bash
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
echo 'export GO111MODULE=auto' >> ~/.bashrc
source ~/.bashrc
```

##### 用户面依赖

```bash
sudo apt -y update
sudo apt -y install mongodb wget git
sudo systemctl start mongodb
```

##### 控制面依赖

```bash
sudo apt -y update
sudo apt -y install git gcc g++ cmake autoconf libtool pkg-config libmnl-dev libyaml-dev
```

#### 源码下载与编译

##### 源码下载

```bash
cd ~
git clone --recursive -b v3.2.1 -j `nproc` https://github.com/free5gc/free5gc.git
cd free5gc
```

##### 编译源码

```bash
cd ~/free5gc
make
```

注意如果出现了编译失败，有可能是在上一步的源码下载部分出现问题。使用如下命令删除后重新下载：

```bash
cd ~
sudo rm -rf free5gc
```

#### UPF功能安装

```bash
cd ~
git clone -b v0.6.8 https://github.com/free5gc/gtp5g.git
cd gtp5g
make
sudo make install
```

#### WebConsole安装

Webconsole可用于观察UE注册信息。

##### 安装依赖

```bash
sudo apt install curl
sudo apt remove cmdtest
sudo apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get update
sudo apt-get install -y nodejs yarn
```

##### 下载编译源码

```bash
cd ~/free5gc
make webconsole
```


### 运行WebConsle
```bash

cd ~/free5gc/webconsole
./bin/webconsole
```
### 运行Free5gc核心网
新建一个终端，输入如下指令：
```bash
cd ~/free5gc
./run.sh
```
### 退出Free5gc核心网
如果在启动Free5gc核心网时遇到问题，在按下`ctrl+c`后还需关闭掉其他的一些已启动的进程：
```bash
cd ~/free5gc
./force_kill.sh
```

### Free5gc配置

配置`/config`下面的`amfcfg.yaml`,`smfcfg.yaml`以及`upfcfg.yaml`。

修改`amfcfg.yaml`中`plmnd`字段下所有的`mnc`和`mcc`，同时修改`ngapIpList`中的IP地址为运行核心网的主机上网用的IP地址。如：

#### amfcfg.yaml

```yaml
ngapIpList:  # the IP list of N2 interfaces on this AMF
    - 10.25.18.205
...
servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
    # <GUAMI> = <MCC><MNC><AMF ID>
    - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
        mcc: 466 # Mobile Country Code (3 digits string, digit: 0~9)
        mnc: 92 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
      amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
  supportTaiList:  # the TAI (Tracking Area Identifier) list supported by this AMF
    - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
        mcc: 466 # Mobile Country Code (3 digits string, digit: 0~9)
        mnc: 92 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
      tac: 1 # Tracking Area Code (uinteger, range: 0~16777215)
  plmnSupportList: # the PLMNs (Public land mobile network) list supported by this AMF
    - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
        mcc: 466 # Mobile Country Code (3 digits string, digit: 0~9)
        mnc: 92 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
      snssaiList: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
        - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
          sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
        - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
          sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
```

#### smfcfg.yaml

修改N3接口的IP地址：

```yaml
interfaces: # Interface list for this UPF
          - interfaceType: N3 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - 10.25.18.205
```

#### upfcfg.yaml

修改gtpude的N3接口的IP地址：

```yaml
gtpu:
  forwarder: gtp5g
  # The IP list of the N3/N9 interfaces on this UPF
  # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
  ifList:
    - addr: 10.25.18.205
      type: N3
      # name: upf.5gc.nctu.me
      # ifname: gtpif
```




## OAI基站与射频仿真的安装与使用

### 下载编译源码

```bash
cd ~
git clone http://git.icfn.top/icfn-ai/oxg-ran.git
cd oxg-ran
cd cmake_targets
./build_oai -I --gNB --nrUE -w SIMU --ninja

```

### 创建Nets名称空间命令

```bash
sudo ip netns delete ueNameSpace2
sudo ip link delete v-eth2
sudo ip netns add ueNameSpace2 
sudo ip link add v-eth2 type veth peer name v-ue2
sudo ip link set v-ue2 netns ueNameSpace2
sudo ip addr add 10.201.1.1/24 dev v-eth2 
sudo ip link set v-eth2 up 
sudo iptables -t nat -A POSTROUTING -s 10.201.1.0/255.255.255.0 -o {你上网的网卡名称} -j MASQUERADE 
sudo iptables -A FORWARD -i {你上网的网卡名称} -o v-eth2 -j ACCEPT 
sudo iptables -A FORWARD -o {你上网的网卡名称} -i v-eth2 -j ACCEPT 
sudo ip netns exec ueNameSpace2 ip link set dev lo up 
sudo ip netns exec ueNameSpace2 ip addr add 10.201.1.2/24 dev v-ue2 
sudo ip netns exec ueNameSpace2 ip link set v-ue2 up 

```

注意需要修改`{你上网的网卡名称}`的内容（包括花括号）。使用`ifconfig`命令查看网络连接情况并找到自己上网的网卡。

### 配置ue和gNB

gNB的一般配置文件格式参照`targets/PROJECTS/GENERIC-NR-5GC/CONF/*.conf`中的格式修改，修改内容有如下几个关键点：`plmn`中的`mmc`和`mnc`；以及`amf`对应的ip地址和gNB和amf之间的`接口地址`和`网卡名称`。

```yaml	
plmn_list = ({ mcc = 466; mnc = 92; mnc_length = 2; snssaiList = ({ sst = 1 }) });
...
mf_ip_address      = ( { ipv4       = "10.25.18.205"; //核心网的ip地址
                                      ipv6       = "192:168:30::17";
                                      active     = "yes";	//这个不用管
                                      preference = "ipv4";
                                    }
                                  );

         NETWORK_INTERFACES :
            {
                GNB_INTERFACE_NAME_FOR_NG_AMF            = "eno2";				//gNB主机上网的网卡名称
                GNB_IPV4_ADDRESS_FOR_NG_AMF              = "10.25.18.142/24";	//gNB主机上网的IP地址
                GNB_INTERFACE_NAME_FOR_NGU               = "eno2";				//gNB主机上网的网卡名称
                GNB_IPV4_ADDRESS_FOR_NGU                 = "10.25.18.142/24";	/gNB主机上网的IP地址
                GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
            };
```

UE的配置比较简单，只要和WebConsole中对应起来即可：

```yaml	
uicc0 = {
imsi = "466920000000003";
key = "8baf473f2f8fd09487cccbd7097c6862";
opc= "8e27b6af0e692e750f32667a3b14605d";
dnn= "internet";
nssai_sst=1;
nssai_sd=0x010203;
}

```

需要注意的是`nssai_sd`使用的是十六进制数。

### 运行基站

```bash
sudo RFSIMULATOR=server ./ran_build/build/nr-softmodem --rfsim --sa -E  -O ../sa.conf
```

### 运行RFSIM仿真

```bash
sudo ip netns exec ueNameSpace2 bash //进入ueNameSpace2
cd oxg-ran
cd cmake_targets
sudo RFSIMULATOR=10.201.1.1 ./ran_build/build/nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --sa --nokrnmod 1 --non-stop -E   -O ../ue.conf
```


## SONY & LAB
- SONY：Redmi K40
  * `imsi`：466920000000001
  * `K*`：00112233445566778899aabbccddeeff
  * `OPc`：000102030405060708090a0b0c0d0e0f
- ICFN：OPPO Reno3 PRO 5G
  * `imsi`： 466920000000003
  * `K*`：465B5CE8B199B49FAA5F0A2EE238A6BC
  * `OPc`：E8ED289DEBA952E4283B54E88E6183CA

* Free5gc 默认
  * `imsi`： 208930000000003
  * `K*`：8baf473f2f8fd09487cccbd7097c6862
  * `OPc`：8e27b6af0e692e750f32667a3b14605d