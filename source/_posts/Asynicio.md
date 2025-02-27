---
title: Python协程：Asyncio入门
mathjax: true
tags:
- Python
- 异步
- 协程
categories:
- 学习记录
- Python
---

<p align="center">
    <img src="https://s2.loli.net/2023/05/11/eNXy1CHzBRkJMw8.jpg" alt="pyecharts logo" />
</p>


本文是对Python协程的学习记录。协程并不能像多线程那样提升程序运行效率，它更适合处理那些需要等待的任务，如网络通信。本文使用的库是`asyncio`，通过asyncio的Event Loop高效完成任务。内容参考[码农高天的协程教学视频](https://www.bilibili.com/video/BV1oa411b7c9/?spm_id_from=333.999.0.0&vd_source=ab34db443b112b108b42c31ac575fd1f)。

<!-- more -->

## 基本概念

asyncio的运算核心是Event Loop，它类似一个大脑，在面对很多个任务的时候决定执行哪一个任务。在Python的asyncio中，同时执行的任务只能有一个，不存在系统级的上下文切换，和线程是不一样的。它需要每一个任务主动向Event Loop报告自身的运行情况，如：这个任务完成了，可以让下一个任务开始了。这种方式有一个好处就是不存在竞争冒险的问题，可以明确知道一个任务什么时候开始什么时候结束。下面我们需要明白两个关键的概念：`Task`和`Coroutine`。

### Coroutine和Task

在python中，Coroutine一般指两个东西：`Coroutine Function `和 `Coroutine Object`。我们看下面代码：

```python
import asyncio

async def main():
    print("hello")
    await asyncio.sleep(1)
    print('world')
    
asynio.run(main())
```

所有以`async def`开头的内容定义的都是Coroutine function，所有Coroutine Function返回的是一个Coroutine Object，进行变量赋值的时候他不会运行这个Coroutine Function的任何代码：

```python
import asyncio

async def main():
    print("hello")
    await asyncio.sleep(1)
    print('world')

coo = main()

#sys:1: RuntimeWarning: coroutine 'main' was never awaited
#RuntimeWarning: Enable tracemalloc to get the object allocation traceback
```

可以看到并没有输出我们预期的结果，同时也还有系统的警告。那怎么运行coroutine的代码呢？首先需要进入async的模式即Event loop控制的状态，第二步则是将Coroutine转换为Task。我们一般书写Python代码的时候对应的是`“Synchronous”`模式，如果要切换到`Asynchronous`模式的话一般来说只用`asyncio.run()`这个入口函数，它的参数是一个Coroutine。

该入口函数做了两件事情：首先建立起Event loop，其次把传入的Coroutine参数转变为第一个Task，Event loop建立完后就会去寻找可执行的Task，由于此时只有一个Task所以他会run给进来的这个Coroutine（此时转换为了Task）

### 多个Coroutine的排队执行

我们看这段代码：

```python
import asyncio
import time


async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    await say_after(1,'hello')
    await say_after(2, 'world')
    
    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())

#started at 10:42:19
#hello
#world
#finished at 10:42:22
```

我们可以使用`await`关键词来将Coroutine变成task。当我们await一个Coroutine的时候发生了如下几件事情：

1. Coroutine 被包装成了一个Task，并且被告诉了Event Loop说这里有一个新的task
2. 告知Event Loop现在这个Task需要等`say_after`的Task执行完后才能继续
3. 告诉Event Loop现在这个Task干不了，yield出去，先让其他Task先执行
4. 最后Event Loop再次安排它运行的时候，他会把say_after这个coroutine里面的真正返回值拿出来并保存

下面是整个事件的流程：

1. asyncio.run把main函数作为一个task放到了Event Loop，Event Loop发现只有一个Task就让这个Task main运行，
2. main在运行时先print一个开始时间然后它运行say_after这个`coroutine function`得到一个`coroutine object`， await这个coroutine object把这个coroutine object 变成一个task放回Event Loop里，同时告诉Event Loop我需要等待他然后把控制权交还给了Event Loop
3. 现在Event Loop里面有两个·task·，一个是main一个是say_after，但是main运行不了，因为main要等say_after运行完，所以Event Loop就让say_after先运行
4. say_after里面有一个asyncio.sleep也是一个coroutine，say_after 也是把它变成一个`task`然后告诉Event Loop我得等这个sleep完成了我才能运行，然后await又把这个控制权交给Event Loop
5. Event Loop现在有三个`task`：一个是main，一个say_after，一个asyncio.sleep，这个sleep告诉Event Loop说我一秒钟就好了，所以Event Loop就等了一秒钟，一秒钟后这个sleep就完成了，此时的`task`变成只有两个：main和say_after
6. 然后Event Loop让say_after运行，所以say_after打印了hello出来。然后控制权交还给Event Loop，Event Loop一看现在只有一个`task`了即main，然后把控制权给main，此时main需要await第二个say_after，重复上面的流程。

以上所有的控制都是显式的，Event Loop无法从一个task那里拿回一个控制权，而是只能经由task主动交还：await交回和函数执行完成交回。

### 但是为什么不能让大家一起等待呢

在上面的例子中我们可以看到在需要等待的任务中，任务是顺序执行的，这和我们的synchronize的编写好像看不出太多区别...这是因为await的原因，await做了太多的事情了。为了解决这个问题呢，asyncio提供了`create_task()`函数：

```python
import asyncio
import time


async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    task1 = asyncio.create_task(
        say_after(1,'hello')
    )

    task2 = asyncio.create_task(
        say_after(2,'world')
    )
    print(f"started at {time.strftime('%X')}")

    await task1
    await task2

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())

#started at 12:36:35
#hello
#world
#finished at 12:36:37
```

此时一共等待了两秒即完成了任务：

1. `create_task()`函数接受的参数是一个coroutine，然后将其变成一个task并将其注册到Event Loop里面。
2. 在执行到task1部分的时候注册到Event Loop的task有main和task1，但是此时Event Loop不能执行task1，因为控制权还在main手上。
3. 所以接下来又注册了一个task2，随后才开始await task1 和await task2。
4. 当await后面是一个task的时候，他只做一件事情：告诉Event Loop将后面的这个task完成，并把当前的控制权交还给Event Loop，控制权回来的时候从task里面提取所需要的返回值。
5. 在await task1的时候Event Loop里面其实已经有三个tasks：`main` ,`task1`和`task2`。 `task1`告诉Event Loop自己需要等一秒钟完成，所以等待的期间Event Loop“闲来无事”的发现`task2`可以执行，`task2`又跟他说要等两秒钟，这样两个task就可以同时执行了。

### 多个task带返回值

```python
import asyncio
import time


async def say_after(delay, what):
    await asyncio.sleep(delay)
    return f"{what} - {delay}"

async def main():
    task1 = asyncio.create_task(
        say_after(1,'hello')
    )

    task2 = asyncio.create_task(
        say_after(2,'world')
    )
    print(f"started at {time.strftime('%X')}")

    ret1 = await task1
    ret2 = await task2

    print(ret1)
    print(ret2)

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
#started at 12:49:25
#hello - 1
#world - 2
```

一定要注意，是`变量= await 带返回值的函数`，如果不用await你是拿不到这个返回值的。但是加入我有很多task呢？难道每一个都要这么写吗？我们可以使用`asyncio.gather()`函数处理这个问题。

```python
import asyncio
import time


async def say_after(delay, what):
    await asyncio.sleep(delay)
    return f"{what} - {delay}"

async def main():
    task1 = asyncio.create_task(
        say_after(1,'hello')
    )

    task2 = asyncio.create_task(
        say_after(2,'world')
    )
    print(f"started at {time.strftime('%X')}")

    ret1 = await asyncio.gather(task1, task2)

    print(ret1)

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
# started at 12:57:20
# ['hello - 1', 'world - 2']
# finished at 12:57:22
```

gather函数不是coroutine，它会返回一个叫做future的东西。future是可以await的。gather函数的参数是若干个coroutine或者task 甚至可以是future，即gather的return值也可以继续gather。对于coroutine，首先将起包装成task并注册到eventloop里面，然后返回一个future值，当你await这个future的时候就相当于告诉eventloop需要等待这里面每一个task都完成才可以继续执行，同时会把这些task的return值放到一个list里面。还有一个好处就是我们可以不用手动去建立task：

```python
import asyncio
import time


async def say_after(delay, what):
    await asyncio.sleep(delay)
    return f"{what} - {delay}"

async def main():

    print(f"started at {time.strftime('%X')}")

    ret1 = await asyncio.gather(say_after(1,'hello'), say_after(2,'world'))

    print(ret1)

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
# started at 12:57:37
# ['hello - 1', 'world - 2']
# finished at 12:57:39
```

## 总结

Event Loop是一个大脑，负责处理若干个task。task是没办法控制Event Loop去执行某一个task的，只能告诉Event Loop我在等这个task，最终是由Event Loop来决定下一个执行的task。而Event Loop一旦开始运行task就必须要要task显式地将控制权交还给Event Loop，交还的方式有await和函数运行完毕。尽管我们会说这种协程的方式是并发的，但是同一时刻只有一段代码在运行，只不过利用了代码中间的等待时间。 如果你的代码没有等待的任务，那么协程对你来说是没用的。
