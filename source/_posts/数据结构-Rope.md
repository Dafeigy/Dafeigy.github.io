---
title: 字符串结构的高效实现——Rope
mathjax: true
tags:
- 数据结构
categories:
- 学习记录
---


Rope作为一种高效处理字符串的数据结构广泛应用于长文本的字符串操作中。下文给出Rope数据结构的基本结构以及Python的实现。

<!-- more -->

## Rope简介

字符串连接是一种十分常见的操作。当字符串以传统的方式(比如字符数组)存储的时候，连接操作需要花费$O(n)$的时间；而`Rope`可以高效处理字符串的拼接、查询、删除、及随机访问。Rope的一个典型应用场景是：**在一个文本编辑程序里，用来保存较长的文本字符串**。图1给出了Rope结构存储的字符串“Hello_my_name_is_Simon”。

![Rope Structure](https://media.geeksforgeeks.org/wp-content/uploads/ropeDataStructure-1.jpg)

<center><small></small>ropeDataStructure</small></center>

从图中可看出，Rope是一个二叉树，叶节点包含的是字符串的子串，非叶节点是权重，其值等于左子树叶节点的所有字符之和。一个字符串被分割为两部分，左子树包含了字符串左部分，右子树包含了字符串的右部分，这些部分都是由用户自己确定的。

## Rope的Python实现

一个Rope的数据结构有如下几种操作：

* 索引`index()`。它返回位置为`i`的字符。我使用`__getitem__()`方法实现使它更符合`Python`语法。
* 拼接`concat()`。它连接两个`Rope`结构的字符串一个`Rope`中。我使用`__add__()`方法重载使得操作更加方便。
* 分割`split()`。它分离字符串`s`在位置`i`处的两个字符串，返回`s1`和`s2`。

下面给出我的实现思路。把这个二叉树的所有节点都看成是一个`Rope`对象，任意的Rope对象之间都可以拼接，这种结构很递归，因此我们可以使用一个递归的结构进行实现，具体来说就是可以通过一个`List`存储所有的Rope的字符串`data`，叶节点的数据采用`str`，其左右子树指向`None`，节点大小定义为字符串大小。递归的终点就是Rope（对应如中的节点）的内容是`Str`，而不是若干被切分的`List`。

```python

# 叶节点用Str表示，其左右子树指向None代表已经到底了
# 也可以用[Str]进行Rope构建
# 递归实现，任意组合都可以是一个Rope

class Rope(object):
    def __init__(self, data='') -> None:
        if isinstance(data,list):
            if len(data) == 0:
                self.__init__()
            elif len(data) == 1:
                self.__init__(data[0])
            else:
                idiv = len(data)//2 + (len(data)%2>0)

                self.left = Rope(data[:idiv])
                self.right = Rope(data[idiv:])
                self.data = ''
                self.length = self.left.length + self.right.length
        
        elif isinstance(data,str):
            self.left = None
            self.right = None
            self.data = data
            self.length = len(data)

        else:
            raise TypeError('只能使用`Str`或`List:[Str]`作为Rope的传入数据')
        
        self.current = self
    
    def __add__(self, inputdata):
        if isinstance(inputdata, str):
            inputdata = Rope(inputdata)
        
        tmp = Rope()
        tmp.left = self
        tmp.right = inputdata
        tmp.length = tmp.left.length + tmp.right.length
        tmp.current = self
        return tmp
    
    def __len__(self):
        # 非叶节点长度
        if self.left and self.right:
            return len(self.left) + len(self.right)
        # 叶节点长度
        else:
            return(len(self.data))

    def __getitem__(self, index):
        if isinstance(index, int):
            if self.left and self.right:
                if index < -self.right.length:
                    subindex = index + self.right.length
                elif index >= self.left.length:
                    subindex = index - self.left.length
                else:
                    subindex = index

                if index < -self.right.length or 0 <= index < self.left.length:
                    return self.left[subindex]
                else:
                    return self.right[subindex]
            else:
                return Rope(self.data[index])

        elif isinstance(index, slice):
            if self.left and self.right:
                start = index.start
                if index.start is None:
                    if index.step is None or index.step > 0:
                        head = self.left
                    else:
                        head = self.right
                elif (index.start < -self.right.length or
                        0 <= index.start < self.left.length):
                    head = self.left
                    if index.start and index.start < -self.right.length:
                        start += self.right.length
                else:
                    head = self.right
                    if index.start and index.start >= self.left.length:
                        start -= self.left.length

                # TODO: stop = -right.length could be on either subrope.
                # There are two options:
                #   1. tail = left and stop = None (or left.length)
                #   2. tail = right as a '' string, which is removed
                # Currently doing method 2, but I'm on the fence here.
                stop = index.stop
                if index.step is None or index.step > 0:
                    if (index.stop is None or
                            -self.right.length <= index.stop < 0 or
                            index.stop > self.left.length):
                        tail = self.right
                        if index.stop and index.stop > self.left.length:
                            stop -= self.left.length
                    else:
                        if head == self.right:
                            tail = self.right
                            stop = 0
                        else:
                            tail = self.left
                            if index.stop < -self.right.length:
                                stop += self.right.length
                else:
                    if (index.stop is None or
                            index.stop < (-self.right.length - 1) or
                            0 <= index.stop < self.left.length):
                        tail = self.left
                        if index.stop and index.stop < (-self.right.length - 1):
                            stop += self.right.length
                    else:
                        if head == self.left:
                            tail = self.left
                            stop = -1   # Or self.left.length - 1 ?
                        else:
                            tail = self.right
                            if index.stop >= self.left.length:
                                stop -= self.left.length

                # Construct the rope
                if head == tail:
                    return head[start:stop:index.step]
                else:
                    if not index.step:
                        offset = None
                    elif index.step > 0:
                        if start is None:
                            delta = -head.length
                        elif start >= 0:
                            delta = start - head.length
                        else:
                            delta = max(index.start, -self.length) + tail.length

                        offset = delta % index.step
                        if offset == 0:
                            offset = None
                    else:
                        if start is None:
                            offset = index.step + (head.length - 1) % (-index.step)
                        elif start >= 0:
                            offset = index.step + min(start, head.length - 1) % (-index.step)
                        else:
                            offset = index.step + (start + head.length) % (-index.step)

                    if not tail[offset:stop:index.step]:
                        return head[start::index.step]
                    else:
                        return head[start::index.step] + tail[offset:stop:index.step]
            else:
                return Rope(self.data[index])

    def __repr__(self):
        if self.left and self.right:
            return '{}{} + {}{}'.format('(' if self.left else '',
                                        self.left.__repr__(),
                                        self.right.__repr__(),
                                        ')' if self.right else '')
        else:
            return "\033[1;37;42mRope ['{}']\033[0m".format(self.data)

    def split(self, index):
        if isinstance(index, int):
            if self.left and self.right:
                    if index < -self.right.length:
                        subindex = index + self.right.length
                    elif index >= self.left.length:
                        subindex = index - self.left.length
                    else:
                        subindex = index

                    if index < -self.right.length or 0 <= index < self.left.length:
                        return self.left[:subindex], self.left[subindex:]
                    else:
                        return self.right[:subindex], self.right[subindex:]
            else:
                return Rope(self.data[:index]), Rope(self.data[index:])
        else:
            raise TypeError("`index`必须为`Int`类型")

if __name__ == "__main__":
    a = Rope('hello')
    b = Rope('world')
    c = Rope(['I ', "don't", "know"])

    r = a + b
    print(c)
    print(r)
    print(r[6])
    print(r.split(3))
    
# ((Rope ['I '] + Rope ['don't']) + Rope ['know'])
# (Rope ['hello'] + Rope ['world'])
# Rope ['o']
# (Rope ['hel'], Rope ['lo'])

```

## 性能比较

|      | String | 块状链表      | Rope        |
| ---- | ------ | ------------- | ----------- |
| 索引 | $O(1)$ | $O(\sqrt{N})$ | $O(\log N)$ |
| 插入 | $O(N)$ | $O(\sqrt N)$  | $O(1)$      |
| 删除 | $O(N)$ | $O(\sqrt N)$  | $O(\log N)$ |

