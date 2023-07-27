## Free5GC+OAI gNB+COTS UE

Free5GC和OAI gNB的安装见前文。下面主要是配置：

### 核心网配置

在执行`./run.sh`，命令前需要设置转发：

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o <上网网卡名称> -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
```

上网网卡名称通过`ifconfig`命令查看。

### gNB侧配置

在运行前需要添加网络路由：

```bash
route -n # 查看路由
sudo route add -net 10.60.0.0 netmask 255.255.255.0 gw 192.168.0.131
						^									^
				 核心网分配UE的IP网段					运行核心网主机IP地址
sudo route add -net 10.61.0.0 netmask 255.255.255.0 gw 192.168.0.131
						^									^
				 核心网分配UE的IP网段					运行核心网主机IP地址
```

gNB在运行前需要修改基站配置信息中的`PLMN`相关信息和`amf`地址的信息，如下：

```yaml
amf_ip_address      = ( { ipv4       = "192.168.0.133";	# 修改成运行核心网主机的IP地址
                              ipv6       = "192:168:30::17";
                              active     = "yes";
                              preference = "ipv4";
                            }
                          );
NETWORK_INTERFACES :
    {
        GNB_INTERFACE_NAME_FOR_NG_AMF            = "eno1";				# 修改成运行gNB的主机上网的网卡名称
        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.0.140/24";	# 修改成运行gNB的主机的IP
        GNB_INTERFACE_NAME_FOR_NGU               = "eno1";				# 修改成运行gNB的主机上网的网卡名称
        GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.0.140/24";  # 修改成运行gNB的主机的IP
        GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
    };
```

### 手机侧

手机首先需要先锁网，然后打开sim卡设置，点开烧录好的卡，在接入点进行设置：

<center><img src="C:\Users\10706\AppData\Roaming\Typora\typora-user-images\image-20230531130734178.png" alt="image-20230531130734178" style="zoom:50%;" /><img src="C:\Users\10706\AppData\Roaming\Typora\typora-user-images\image-20230531130955075.png" alt="image-20230531130955075" style="zoom:50%;" /></center>

核心网用Free5GC的话，APN是`internet`，如果是OAI的核心网则是`ctnet`。连上就可以进行业务了，比如说抖音和测速啥的：

<center><img src="C:\Users\10706\AppData\Roaming\Typora\typora-user-images\image-20230531131527149.png" alt="image-20230531131527149" style="zoom:50%;" /><img src="C:\Users\10706\AppData\Roaming\Typora\typora-user-images\image-20230531131636260.png" alt="image-20230531131636260" style="zoom:50%;" /></center>



