---
title: Vue3学习记录
mathjax: true
tags:
- vue3
categories:
- 学习记录
---

![image-20230528212216596](https://s2.loli.net/2023/05/28/TdPjxktvKYD9rJC.png)


Vue3学习记录。本篇为第一篇，记录配置与创建Vue3项目。

<!-- more -->

## 两种API

组合式API和选项是API：都能覆盖大部分的应用场景。选项式是在组合式API基础上实现的。

**在生产生项目中，**

* 当不需要使用构建工具，或打算在低复杂度场景中使用Vue，例如渐进增强的应用场景推荐使用选项式API；
* 当打算用Vue构建完整的单页应用，推荐采用组合式API+单文件组件**（一般推荐使用这种）**

## 开发前准备

`npm`版本大于15.0即可。使用`node -V`即可产看版本。随后就是下载并初始化脚手架：

```bash
npm init vue@latest
```

![image-20230528210847282](https://s2.loli.net/2023/05/28/x2zDR1bt5UHSZeE.png)

在创建项目名称时**不要有大写！**随后一路回车即可。项目创建完成后，开始运行项目：

```bash
cd vue-project
npm install
```

通过`npm run dev`即可运行：

![image-20230528211200582](https://s2.loli.net/2023/05/28/TMCsbl5EearzBdq.png)

![image-20230528211221020](https://s2.loli.net/2023/05/28/uL5shn8wRpDMYmN.png)

## VScode中的开发配置

需要添加一个插件`volar`:

![image-20230528211443953](https://s2.loli.net/2023/05/28/aq1Seb4oHvtEn6Y.png)



## Vue项目目录结构

```
vscode								--- vSCode工具的配置文件
node_modules						--- Vue项目的运行依赖文件夹
public								--- 资源文件夹(浏览器图标)
src									--- 源码文件夹，我们主要操作的对象
.gitignore							--- git忽略文件
index.html							--- 入口html文件
package.json						--- 信息描述文件
README.md							--- 说明文件
vite.config.js						--- Vue配置文件，跨域，打包之类的
```

