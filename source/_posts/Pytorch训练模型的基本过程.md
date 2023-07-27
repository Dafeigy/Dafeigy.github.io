---
title: Pytorch训练神经网络的一般过程
mathjax: true
tags:
- Python
- Pytorch

---



这个是我在使用`Pytorch`进行图像相关任务的神经网络构建和训练过程中的一个记录，没有什么参考价值的，可以忽略~

<img src="https://s2.loli.net/2022/08/20/oxvPZeT8G7ydzct.png" alt="avata1r.png" style="zoom:25%;" />

<!-- more -->

# 整体流程

`Pytorch`中构建神经网络训练自己的数据集是个挺繁琐的事情，主要流程分为数据集的构建、网络构建、训练调参以及结果测试四个部分。数据集的构建最好是使用`DataLoader`类进行构造，这是`Pytorch`写好的一个类别，我们只需要简单了解下接口即可顺利地在有监督学习中使用。网络构建则是一个比较中规中矩的过程，写法比较固定，比较好上手；训练调参包含的东西比较繁琐，包括模型参数的调整、加载，以及包含`Tensorboard`对训练结果的可视化(可选)；最后则是测试部分，这个相对也比较简单。

# 数据加载

```python
import torch.utils.data.DataLoader as DataLoader

"""
torch.utils.data.DataLoader(dataset, batch_size=1, shuffle=False, sampler=None,
batch_sampler=None, num_workers=0, collate_fn=None,
pin_memory=False, drop_last=False, timeout=0,
worker_init_fn=None)
"""
```

下面对几个重要的参数进行解析。

## dataset

dataset只接受两种类型的输入，分别是`map-style datasets`和`iterable-style datasets`。

### map-style datasets

是一个类，需自行构建两个魔术方法`__getitem__()`和`__len__()`，用于表示数据的索引到数据的内容的映射。

其中：

* `__getitem__()`用于根据索引遍历全部数据
* `__len__()`用于返回数据集长度
* 创建dataset类时可以对数据进行预处理，预处理的函数可以夹杂在`__getitem__()`方法中进行调用，或者直接在`__init__()`或`__getitem__()`中进行编写，但`__getitem__()`必须根据`index`返回响应值，因为该值会通过index传到DataLoader类中后续处理。

这里给出一个基本的模板：

```python
class Dataset(object):
    """An abstract class representing a :class:`Dataset`.

    All datasets that represent a map from keys to data samples should subclass
    it. All subclasses should overwrite :meth:`__getitem__`, supporting fetching a
    data sample for a given key. Subclasses could also optionally overwrite
    :meth:`__len__`, which is expected to return the size of the dataset by many
    :class:`~torch.utils.data.Sampler` implementations and the default options
    of :class:`~torch.utils.data.DataLoader`.

    .. note::
      :class:`~torch.utils.data.DataLoader` by default constructs a index
      sampler that yields integral indices.  To make it work with a map-style
      dataset with non-integral indices/keys, a custom sampler must be provided.
    """

    def __getitem__(self, index):
        raise NotImplementedError
        
    def __len__(self):
        raise NotImplementedError

```

给出几种常用的Dataset构造：

```python
import torch
from torch import nn
from torch.utils.data import Dataset, DataLoader

# 数值类型的数据构造
class Num_dataset(Dataset):
    def __init__(self):
        super().__init__()
    	# 注意这里输入进来的每个数据是Tensor类型的，承载的部分随便
        self.x = torch.randn(1000,3)
        self.y = self.x.sum(axis=1)
        self.src,  self.trg = [], []
        for i in range(1000):
            self.src.append(self.x[i])
            self.trg.append(self.y[i])
           
    def __getitem__(self, index):
        return self.src[index], self.trg[index]

    def __len__(self):
        return len(self.src) 
    
# 图片类型的数据构造
# 还不完善 等我修补下
train_dir = "../data/hotdog/train"
test_dir = "../data/hotdog/test"

mean = [0.485, 0.456, 0.406]
std = [0.229, 0.224, 0.225]
class Img_dataset(Dataset):
    def __init__(self, path):
        data_root = pathlib.Path(path)
        all_image_paths = list(data_root.glob('*/*'))
        self.all_image_paths = [str(path) for path in all_image_paths]
        label_names = sorted(item.name for item in data_root.glob('*/') if item.is_dir())
        label_to_index = dict((label, index) for index, label in enumerate(label_names))
        self.all_image_labels = [label_to_index[path.parent.name] for path in all_image_paths]
        self.mean = np.array(mean).reshape((1, 1, 3))
        self.std = np.array(std).reshape((1, 1, 3))

    def __getitem__(self, index):
        img = cv.imread(self.all_image_paths[index])
        img = cv.resize(img, (224, 224))
        img = img / 255.
        img = (img - self.mean) / self.std
        img = np.transpose(img, [2, 0, 1])
        label = self.all_image_labels[index]
        img = torch.tensor(img, dtype=torch.float32)
        label = torch.tensor(label)
        return img, label

    def __len__(self):
        return len(self.all_image_paths)

```

### Iterable-style datasets

可迭代样式的数据集是IterableDataset的一个实例，该实例必须重写__iter__方法,该方法用于对数据集进行迭代。这种类型的数据集特别适合随机读取数据不太可能实现的情况，并且批处理大小batchsize取决于获取的数据。比如读取数据库，远程服务器或者实时日志等数据的时候，可使用该样式，一般时序数据不使用这种样式。我就摸了~

## batchsize

一次抽取的数据多少，最好根据数据量的多少以及自己的设备的内存、GPU、显存进行选择，尽量选择2的正整数次幂作为batchsize。

## numworkers

参与程序使用的CPU核心数，注意使用该参数后需要自写主函数入口：

```python
if __name__ == "__main__":
    main()
```

## shuffle

是否对数据进行打乱，一般都选True，因为数据的相关性会影响网络的训练泛化性能。

# 网络构造

所有的网络构造都需要继承`nn.Module`类，然后在构造函数`__init__()`中构造一些网络层的成员，然后重写`forward(self,x)`成员函数。下面给出LeNet的一个构造示例：

```python
import torch.nn as nn
import torch

class LeNet(nn.Module):
    def __init__(self) -> None:
        super().__init__()

        self.conv1 = nn.Conv2d(1,6,3)
        self.pool1 = nn.MaxPool2d(2,2)
        self.conv2 = nn.Conv2d(6,16,3)
        self.pool2 = nn.MaxPool2d(2,2)
        self.fc1 = nn.Linear(16*6*6, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84,10)

    def forward(self,x):
        x = self.pool1(torch.relu(self.conv1(x)))
        x = self.pool2(torch.relu(self.conv2(x)))
        x = x.view(x.size(0),-1)
        x = torch.relu(self.fc1(x))
        x = torch.relu(self.fc2(x))
        x = self.fc3(x)
        return x
```

`Pytorch`构造的网络使用方式比较直观，在实例化对象后，可以直接使调用输出结果：

```python
if __name__ == "__main__":
    model = LeNet()
    ret = model(torch.randn(1,1,32,32))
    print(ret.shape)
    #torch.Size([1, 10])
```

这是因为它继承了`nn.Module`中的`__call__()`方法，而默认的`__call__()`方法则是定义为调用`forward(self,x)`这个成员函数。

# 模型训练

```python
import torch
import torch.nn as nn
from model import LeNet

#Dataset
from torchvision.datasets import MNIST
import torchvision.transforms as transforms
from torch.utils.data import DataLoader


# Load Data

data_train = MNIST('./data', 
                    download = True, 
                    transform = transforms.Compose([transforms.Resize((32,32)),transforms.ToTensor()]))

data_train_loader = DataLoader(data_train, batch_size = 32, shuffle= True, num_workers=1)

# Model
model = LeNet()
# 切换状态
model.train()
lr = 1e-2
loss_func = nn.CrossEntropyLoss()
optimizer = torch.optim.SGD(model.parameters(), lr = lr, momentum = 0.9, weight_decay = 5e-4)

train_loss = 0
correct = 0
total = 0

if __name__ == '__main__':


    for batch_idx, (inputs, targets) in enumerate(data_train_loader):

        optimizer.zero_grad()
        outputs = model(inputs)
        loss = loss_func(outputs, targets)
        loss.backward()
        optimizer.step()

        train_loss += loss.item()
        _, predicted = outputs.max(1)
        total += targets.size(0)
        correct += predicted.eq(targets).sum().item()

        print(batch_idx, len(data_train_loader), "Loss: %.3f | ACC: %.3f%% (%d/%d)" % (train_loss/(batch_idx+1), 100.*correct/total, correct, total))
        
```

值得注意的是，这里的训练只训练了一个epoch，如果训练多个epoch则需要再套一层for循环。

如果要使用一些已经预训练好的模型的权重，则可以选择加载部分权重。分为几种情况：

## 结构相同，某些层不加载

```python
model = LeNet()
model_dict = model.state_dict()#新的模型结构
pre_train = torch.load('model.pth')
pretrain_dict = pre_train.state_dict()#已有的模型权重
for each in pretrain_dict:
    print(each)
print('--------------------')

load_pretrained_dict = {key: value for key, value in pretrain_dict.items() if (key in model_dict and 'fc1' not in key)}
for each in load_pretrained_dict:
    print(each)

'''
conv1.weight
conv1.bias
conv2.weight
conv2.bias
fc1.weight
fc1.bias
fc2.weight
fc2.bias
fc3.weight
fc3.bias
--------------------
conv1.weight
conv1.bias
conv2.weight
conv2.bias
fc2.weight
fc2.bias
fc3.weight
fc3.bias
'''
```

## 结构不同，不同层不加载

与上面类似，只不过把筛选语句修改以下就可以了：

```python
load_pretrained_dict = {key: value for key, value in pretrain_dict.items() if (key in model_dict)}
```

