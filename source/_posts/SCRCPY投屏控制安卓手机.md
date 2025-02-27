---
title: Ubuntu下SCRCPY投屏控制安卓手机
mathjax: false
tags:
- 投屏
- 安卓
categories:
- 杂项
---

Ubuntu下使用投屏开源的SCRCPY投屏控制安卓手机~

<!--more-->

### 软件安装以及依赖

先安装软件以及adb依赖：

```bash
sudo snap install scrcpy

sudo apt-get install android-tools-adb
```

### 连接手机并配置

首先在手机中设置允许USB调试，然后用USB与电脑连接。随后查看待连接设备的识别号：

```bash
lsusb
```

![2023-05-22 11-07-52 的屏幕截图.png](https://s2.loli.net/2023/05/19/KOelZMHGpqSPgX7.png)

使用的是OPPO Reno3这台设备，它对应的id是`22d9`。记录下这个ID，在后面的配置我们会用到，记得把你的id替换调下面内容中所有的`22d9`。

下一步创建配置文件：

```bash
mkdir ~/.android

echo 0x22d9 > ~/.android/adb_usb.ini

sudo touch /etc/udev/rules.d/android.rules

sudo gedit /etc/udev/rules.d/android.rules
```

在文本编辑器中，添加文件内容：

```bash
SUBSYSTEM=="usb", SYSFS{idVendor}=="22d9", MODE="0666"
```

![2023-05-22 11-15-43 的屏幕截图.png](https://s2.loli.net/2023/05/19/pnyjJXxufZI1EHL.png)

因为我之前也添加过别的手机，所以会多一点。保存后，修改文件权限：

```bash
sudo chmod 777 /etc/udev/rules.d/android.rules
```

### 启动scrcpy

首先先启动adb服务：

```bash
service udev restart
adb start-server
adb devices
```

当检测到设备连接时就成功了：

![2023-05-22 11-20-24 的屏幕截图.png](https://s2.loli.net/2023/05/19/tnhTb6xSR9ymUJX.png)

使用`scrcpy`命令打开：

![Final](https://s2.loli.net/2023/05/19/EY3AJLtiV5am1Fh.png)
