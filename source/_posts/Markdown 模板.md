---
title: Typora新建文件时使用模板
mathjax: false
tags:
- Typora
categories:
- 学习记录
---

<p align="center">
    <img src="https://s2.loli.net/2022/08/20/oxvPZeT8G7ydzct.png" alt="banner" width=200 height=200 />
</p>
<h1 align="center">这是文章模板的标题</h1>
<p align="center">
    <em>这是文章模板的副标题</em>
</p>




经常写日记/博客/ToDo的朋友应该会对每次新建md文件时都要从以前的文档里复制粘贴一份修改感到厌倦，但是我们完全可以使用Typora在新建文档的时候直接搬用模板！>>>

<!--more-->

## 模板文件参考

你可以参考我的模板文件，我常用的Typora写作场景是博客博文写作(Next主题)和项目说明书，所以我的模板就如上所示，你可以根据自己的需求更改。

```markdown
---
title: Typora新建文件时使用模板
mathjax: false
tags:
- tag1
categories:
- category1
---
<p align="center">
    <img src="https://s2.loli.net/2022/08/20/oxvPZeT8G7ydzct.png" alt="pyecharts logo" width=200 height=200 />
</p>
<h1 align="center">Type Your Title here</h1>
<p align="center">
    <em>type your subtitle here</em>
</p>

经常写日记/博客/ToDo的朋友应该会对每次新建md文件时都要从以前的文档里复制粘贴一份修改感到厌倦，但是我们完全可以使用Typora在新建文档的时候直接搬用模板！看看我是怎么操作的>>>

<!--more-->
```

将以上内容或者你的模板内容保存为一个`.md`文件，我的是`Template.md`，然后就需要管理员权限来到`C:\Windows\ShellNew`的目录，如果没有的就新建一个，把我们的模板文件复制到这里：

![image-20220828174923586](https://s2.loli.net/2022/08/28/adlH7VxifLpM3cX.png)

## 新建文件行为写入注册表

按下`win+R`进入运行窗口，输入`regedit`后回车进入注册表。然后在`\HKEY_CLASSES_ROOT\.md\ShellNew`的目录新建项：

![image-20220828174613056](https://s2.loli.net/2022/08/28/HMKxZzArsbhUv6S.png)

新建的项将其命名为`FileName`（注意大小写），然后修改它的值为我们的模板文件的目录：

![image-20220828174728373](C:\Users\cybercolyce\AppData\Roaming\Typora\typora-user-images\image-20220828174728373.png)

然后重启下Typora就可以啦！

## 右键菜单新建`.md`文件(可选)

为了更方便地不打开Typora而是直接右键新建`.md`文件，可以在注册表注册一个右键的新建选项。打开笔记本，然后将以下内容复制粘贴进去：

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\.md]
@="Typora.md"
"Content Type"="text/markdown"
"PerceivedType"="text"

[HKEY_CLASSES_ROOT\.md\ShellNew]
"NullFile"=""
```

然后将它保存为`.reg`文件，直接运行就好。这个需要重启计算机才能在右键看到新增了Markdown文件选项。
