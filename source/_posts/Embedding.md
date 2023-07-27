---
title: Embedding的概念与实现
mathjax: true
tags:
- 深度学习
- 概念理解
categories:
- 学习记录
---


Embedding是流形假说中的一个常用概念，本质上来讲是高维数据到低维数据的映射。

<!-- more -->

## Embedding概念

天才少年Colah的博客中的一篇文章[Neural Networks, Manifolds, and Topology ](https://colah.github.io/posts/2014-03-NN-Manifolds-Topology/)中记录了流形与神经网络的对应关系。Embedding（嵌入）是拓扑学里面的词，在深度学习中常常与Manifolds（流形）搭配使用。一个事物，它可以有很多种表示方法，举个例子：我们所处的空间是三维空间，但我们所处的地方可以用一个二维的经纬度坐标进行表示，我们称这个关系叫做三维数据到二维空间的一个嵌入。

Old school派对Embedding的理解是“自然的原始数据是低维的流形嵌入于(embedded in)原始数据所在的高维空间”，深度学习的任务就是将分布在高维空间的原始数据映射到低维空间，从而在低维空间上对事物进行划分。然而随着深度学习的发展以及概念的误用，Embedding逐渐变成了特征提取的过程，而输入与输出的关系不再仅限于是从高到低维的映射。

另一个问题就是，多模态里面为什么需要做Embedding。基于上面的假说，多模态本质上是针对任务进行信息的融合，而分布在不同的高维数据需要将他们映射到同一个向量空间中，再进行融合，所以这个映射需要用embedding实现。

## Embedding实现

### 简单实现

`Pytorch`中有`torch.nn.embedding(num_embeddings, embedding_dim, padding_idx)`这个类实现embedding，常见的一个应用场景是NLP的词向量嵌入。该函数有三个参数：

* **num_embeddings**:词表大小
* **embedding_dim**:需要多少个维度表示一个符号
* **padding_idx**:即需要用0填充的符号在词表中的位置

```python
import torch
import torch.nn as nn

#词表
word_to_id = {'hello':0, '<PAD>':1,'world':2}
embeds = nn.Embedding(len(word_to_id), 4,padding_idx=word_to_id['<PAD>'])

text = 'hello world <PAD> <PAD>'
hello_idx = torch.LongTensor([word_to_id[i] for i in text.split()])
#词嵌入得到词向量
hello_embed = embeds(hello_idx)
print(hello_embed)

tensor([[-1.1436, 1.4588, -1.2755, 0.0077],
[-0.9600, -1.9986, -1.1087, -0.1520],
[ 0.0000, 0.0000, 0.0000, 0.0000],
[ 0.0000, 0.0000, 0.0000, 0.0000]], grad_fn=)
```

### 稍微复杂的实现

稍微复杂的实现是我在用指针网络解决TSP问题的过程中参考别人的代码时发现的：

```python
class GraphEmbedding(nn.Module):
    def __init__(self, input_size, embedding_size, use_cuda=USE_CUDA):
        super(GraphEmbedding, self).__init__()
        self.embedding_size = em bedding_size
        self.use_cuda = use_cuda
        
        self.embedding = nn.Parameter(torch.FloatTensor(input_size, embedding_size)) 
        self.embedding.data.uniform_(-(1. / math.sqrt(embedding_size)), 1. / math.sqrt(embedding_size))
        
    def forward(self, inputs):
        batch_size = inputs.size(0)
        seq_len    = inputs.size(2)
        embedding = self.embedding.repeat(batch_size, 1, 1)  
        embedded = []
        inputs = inputs.unsqueeze(1)
        for i in range(seq_len):
            embedded.append(torch.bmm(inputs[:, :, :, i].float(), embedding))
        embedded = torch.cat(embedded, 1)
        return embedded
```

本质是一个矩阵乘法。
