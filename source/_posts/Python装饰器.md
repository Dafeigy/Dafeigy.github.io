---
title: Python装饰器解析与使用场景
mathjax: false
tags:
- python
- 装饰器
categories:
- python技巧
---

![img](https://s2.loli.net/2022/10/15/EwZH8YS1Nkt67mr.jpg)

如果你是开发人员，那装饰器对你开发调试有很大帮助。装饰器作为python这门语言特有的一个东西，和其他语言的东西很不一样，而且网上的例子又很让人退却三步，所以我看了一位B站up主的[视频](https://www.bilibili.com/video/BV1Gu411Q7JV/?spm_id_from=333.999.0.0&vd_source=ab34db443b112b108b42c31ac575fd1f)后就大概理清楚了，也推荐大家关注一下。

<!-- more -->

## 函数是什么

函数就是一段可以复用的代码段，这是每门语言都有的东西。但是python中的函数与其他语言中的函数的区别，或者说python这门语言和其他语言的本质区别就在于，python中的任何东西都是一个`object`：

![image-20221015211022717](https://s2.loli.net/2022/10/15/Fv1QB8wEZpCPKus.png)

<center><small>编译后的字节码，摘自Up主@码农高天 视频</small></center>

我们可以看到整个过程就是1.先把`code object`加载进来2.加载函数名称`double`3.制作了一个`function obeject`4.将function object保存到double里面。因此定义函数的整个过程就是：新建一个变量，然后在变量里面保存一个`function object`。

### 函数作为参数

python中的函数对象有一个特点，那就是`callable`的，这意味着你可以调用它。到这里你也能理解了所谓的函数对象也只不过是一个普通的对象而已，它也可以作为参数传入其他函数：

```python
def double(x):
    return x * 2

def triple(x):
    return x * 3

def cal_number(func,x):
    print(func(x))
    
cal_number(double,3)
cal_number(triple,3)

#6 9
```

### 函数作为返回值

作为参数和作为返回值都体现着函数只不过是一个普通的对象这一事实，因此我们可以这样定一个函数：

```python
def triplet_loss(alpha = 0.2):
    def _triplet_loss(y_pred,Batch_size):
        anchor, positive, negative = y_pred[:int(Batch_size)], y_pred[int(Batch_size):int(2*Batch_size)], y_pred[int(2*Batch_size):]

        pos_dist = torch.sqrt(torch.sum(torch.pow(anchor - positive,2), axis=-1))
        neg_dist = torch.sqrt(torch.sum(torch.pow(anchor - negative,2), axis=-1))
        
        keep_all = (neg_dist - pos_dist < alpha).cpu().numpy().flatten()
        hard_triplets = np.where(keep_all == 1)

        pos_dist = pos_dist[hard_triplets]
        neg_dist = neg_dist[hard_triplets]

        basic_loss = pos_dist - neg_dist + alpha
        loss = torch.sum(basic_loss) / torch.max(torch.tensor(1), torch.tensor(len(hard_triplets[0])))
        return loss
    return _triplet_loss
```

这是我在做人脸识别的时候写的一个损失函数。它的使用过程是这样的：传入一个alpha的参数然后返回一个`_triplet_loss`函数，这个`_triplet_loss`函数的具体返回值就是`loss`。使用时是这样的：

```python
loss_fn = triplet_loss()
loss = loss_fn(y_hat,batch_size)

#0.012
```

注意一点：这和`Pytroch`里面常用的那几个损失函数不一样，尽管他们的使用语法很相近：

```python
import torch.nn as nn

loss_fn = nn.CrossEntropyLoss()
loss = loss_fn(output, target)
```

但是他们的本质是一样的。`Pytorch`中的`loss_fn`是一个对象的实例化，然后通过重写对象的`__call__`方法调用`nn`模块的标准`foward()`函数进行运算，所以是在函数中调用函数，调用`__call__`方法的就是类装饰器中执行的函数。

## 假如我要测个时间

其实也不用假如，因为按现在这个就业形式迟早都会变成开发...言归正传，在刚刚看完罗里吧嗦一大堆之后我们放松下设想一个场景：

你需要测试一个函数`Deny996()`的运行时间，你怎么办？你二话不说：

```python
def Deny996()():
	return None

import time
start = time.time()
Deny996()
print(time.time()-start)
# 0.0
```

非常的简单！但是问题来了，下面你老板让你继续测其他的函数运行时间：

```python
def Deny997():
    return None
def Deny007():
    return None
def Deny667():
    return None
def Deny887():
    return None
def Deny995():
    return None
...
```

你说这简单，就是废我的`ctrl`键和`c`键`v`键。为了避免这种低效的方法，我们使用了装饰器：

```python
def Deny996():
    return None

def count_time(func):
    def wrapper():
        start = time.time()
        func()
        print(time.time()-start)
    return wrapper

if __name__ == "__main__":
    func = count_time(Deny996)
    func()	#相当于执行wrapper()
```

这里面的`count_time`就是一个装饰器，这里面又定义了一个函数`wrapper`，并把`func`作为参数放进来运行。当然啦`wrapper`的名字可以随便起，只要符合变量命名规则就行。

## 装饰器的语法糖

其实大家对装饰器的语法糖更熟悉，它的符号是`@`。上面的那段代码可以用这样来等价替代：

```python
def count_time(func):
    def wrapper():
        start = time.time()
        func()
        print(time.time()-start)
    return wrapper

@count_time
def Deny996():
    return None

if __name__ == "__main__":
    Deny996()
```

## 但是我的函数需要传入参数...?

很明显我写的装饰器只考虑了固定形式的参数，但是万一我有一个装饰器是对不同参数设置的函数进行测试呢？或者说我希望我的装饰器也能通过参数更改功能怎么办？

### 装饰器传参

我们先看装饰器传入参数。Python中对不定参数传入有个办法：`*args`和`**kwargs`。前者是传入非键值对参数，后者是传入键值对函数比如说字典类型的变量。给个例子：

```python
import time
 
def count_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        func(*args, **kwargs)
        print("执行时间为：", time.time() - t1)
 
    return wrapper

@count_time
def Deny996(name1,name2):
    print(name1,name2)
    
@count_time
def Deny997(name):
    print(name)
    

if __name__ == "__main__":
    Deny996("hello","world")
    Deny997("Capitalism")
```



### 装饰器带参

很简单，就是用装饰器去装饰装饰器：

```python
import time
 
def count_time_args(msg = None)：
    def count_time(func):
        def wrapper(*args, **kwargs):
            t1 = time.time()
            func(*args, **kwargs)
            print("{}执行时间为：".format(msg), time.time() - t1)

        return wrapper
    return count_time

@count_time_args(msg="996")
def Deny996(name1,name2):
    print(name1,name2)
    
@count_time_args(msg="997")
def Deny997(name):
    print(name)
    

if __name__ == "__main__":
    Deny996("hello","world")
    Deny997("Capitalism")

# hello world
# 996执行时间为： 0.0009996891021728516
# Capitalism
# 997执行时间为： 0.0
```

## 类装饰器到底是不是装饰器？

虽然我在前面说了一嘴`Pytorch`中的损失函数和一般的装饰器运行过程不太一样，但也不要否认人家是装饰器这一事实！我们可以看看`Pytorch.nn.CrossEntropyLoss`类：

```python
class CrossEntropyLoss(_WeightedLoss):
    __constants__ = ['ignore_index', 'reduction', 'label_smoothing']
    ignore_index: int
    label_smoothing: float

    def __init__(self, weight: Optional[Tensor] = None, size_average=None, ignore_index: int = -100,reduce=None, reduction: str = 'mean', label_smoothing: float = 0.0) -> None:
        super(CrossEntropyLoss, self).__init__(weight, size_average, reduce, reduction)
        self.ignore_index = ignore_index
        self.label_smoothing = label_smoothing

    def forward(self, input: Tensor, target: Tensor) -> Tensor:
        return F.cross_entropy(input, target, weight=self.weight,
                               ignore_index=self.ignore_index, reduction=self.reduction,
                               label_smoothing=self.label_smoothing)
```

你会说这也没写那个`__call__`方法啊？倒是`forward()`函数写了。别急，我们看看这个类的继承关系：

`CrossEntropyLoss -> _WeightedLoss -> _Loss -> Module`。点开`Module`类，在1176行有这么一条：

```
__call__ : Callable[..., Any] = _call_impl
```

我们再点开`_call_impl`：

```python
def _call_impl(self, *input, **kwargs):
        forward_call = (self._slow_forward if torch._C._get_tracing_state() else self.forward)
        # If we don't have any hooks, we want to skip the rest of the logic in
        # this function, and just call forward.
        if not (self._backward_hooks or self._forward_hooks or self._forward_pre_hooks or _global_backward_hooks
                or _global_forward_hooks or _global_forward_pre_hooks):
            return forward_call(*input, **kwargs)
        # Do not call functions when jit is used
        full_backward_hooks, non_full_backward_hooks = [], []
        if self._backward_hooks or _global_backward_hooks:
            full_backward_hooks, non_full_backward_hooks = self._get_backward_hooks()
        if _global_forward_pre_hooks or self._forward_pre_hooks:
            for hook in (*_global_forward_pre_hooks.values(), *self._forward_pre_hooks.values()):
                result = hook(self, input)
                if result is not None:
                    if not isinstance(result, tuple):
                        result = (result,)
                    input = result

        bw_hook = None
        if full_backward_hooks:
            bw_hook = hooks.BackwardHook(self, full_backward_hooks)
            input = bw_hook.setup_input_hook(input)

        result = forward_call(*input, **kwargs)
        if _global_forward_hooks or self._forward_hooks:
            for hook in (*_global_forward_hooks.values(), *self._forward_hooks.values()):
                hook_result = hook(self, input, result)
                if hook_result is not None:
                    result = hook_result

        if bw_hook:
            result = bw_hook.setup_output_hook(result)

        # Handle the non-full backward hooks
        if non_full_backward_hooks:
            var = result
            while not isinstance(var, torch.Tensor):
                if isinstance(var, dict):
                    var = next((v for v in var.values() if isinstance(v, torch.Tensor)))
                else:
                    var = var[0]
            grad_fn = var.grad_fn
            if grad_fn is not None:
                for hook in non_full_backward_hooks:
                    wrapper = functools.partial(hook, self)
                    functools.update_wrapper(wrapper, hook)
                    grad_fn.register_hook(wrapper)
                self._maybe_warn_non_full_backward_hook(input, result, grad_fn)

        return result
```

好长一大段是吧？别急，我们只需关注第一行最后面那个`self.foward`就行了：

```python
forward_call = (self._slow_forward if torch._C._get_tracing_state() else self.forward)
```

现在应该懂了`__call__()`是干啥的了吧？总之通过复杂的继承关系，让最后的`CrossEntropyLoss`的`__call__()`调用了当前类里定义的`foward()`函数。所以这也是一种装饰器！

## 装饰器顺序

结论就是离函数最近的装饰器最先执行。可以这么理解：装饰器包裹着一个函数，然后又被另一个装饰器包裹，然后在执行函数的时候最先完成执行的是函数，随后才是函数外第一层的装饰器，然后到第二层，第三层...



## 装饰器用在哪？

以上就是装饰器的所有用法啦！除了我们用来测时间的应用场景，我们还可以通过装饰器来设定参数传入函数，比如说你在测试用户权限啥的，你可以写一个用户等级的函数，然后在测试的时候调用这个等级函数作为装饰器去测试你的业务；总之这个装饰器的用法很多，恰当使用的话可以提高开发效率。

