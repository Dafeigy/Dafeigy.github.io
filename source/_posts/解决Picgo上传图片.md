---
title: 关于SMMS更换国内域名导致Picgo不能上传图片的解决方法
tags:
- PicGo
categories:
- 学习记录
---



最近（2022-8-20）在写文档，突然发现用`PicGo`上传图片一直报错，主要是`Error: connect ETIMEDOUT`和`Error: read ECONNRESET`这俩，但是我挂了梯子后用SMMS是可以直接上传图片的，所以问题出在`PicGo`上面了。查了下，估计是和SMMS更换了国内域名为`.app`的原因，这里给出我的解决方法:......

<!-- more -->



解决方法也很简单，打开`PicGo`然后点`PicGo设置`，设置下代理和镜像源就好：

<img src="https://s2.loli.net/2022/08/21/YOnJ9T2FG3A8sEb.png" alt="image-20220821001419713" style="zoom:80%;" />

<center><small>我的PicGo设置</small></center>

```yaml
上传代理: http://127.0.0.1:7890
插件安装代理: http://127.0.0.1:7890
插件镜像地址: https://registry.npm.taobao.org/
```



然后就点确定重启一下`PicGo`就好了~

