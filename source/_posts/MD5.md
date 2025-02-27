---
title: MD5加密算法的Python实现流程
mathjax: true
tags:
- Python
- 数字签名
- MD5
categories:
- 通信技术
- 加密技术
---

**MD5信息摘要算法**（英语：MD5 Message-Digest Algorithm），一种被广泛使用的[密码散列函数](https://baike.baidu.com/item/密码散列函数/14937715)，可以产生出一个128位（16[字节](https://baike.baidu.com/item/字节/1096318)）的散列值（hash value），用于确保信息传输完整一致。MD5由美国密码学家[罗纳德·李维斯特](https://baike.baidu.com/item/罗纳德·李维斯特/700199)（Ronald Linn Rivest）设计，于1992年公开，用以取代[MD4](https://baike.baidu.com/item/MD4/8090275)算法。这套算法的程序在 RFC 1321 标准中被加以规范。1996年后该算法被证实存在弱点，可以被加以破解，对于需要高度安全性的数据，专家一般建议改用其他算法，如[SHA-2](https://baike.baidu.com/item/SHA-2/22718180)。2004年，证实MD5算法无法防止碰撞（collision），因此不适用于安全性认证，如[SSL](https://baike.baidu.com/item/SSL/320778)公开密钥认证或是[数字签名](https://baike.baidu.com/item/数字签名/212550)等用途。

<!--more-->

首先先要对MD5使用的初始数据进行初始化：


```python
A=0x67452301

B=0xEFCDAB89

C=0x98BADCFE

D=0x10325476
T = [0xD76AA478,0xE8C7B756,0x242070DB,0xC1BDCEEE,0xF57C0FAF,0x4787C62A,0xA8304613,0xFD469501,
    0x698098D8,0x8B44F7AF,0xFFFF5BB1,0x895CD7BE,0x6B901122,0xFD987193,0xA679438E,0x49B40821,
    0xF61E2562,0xC040B340,0x265E5A51,0xE9B6C7AA,0xD62F105D,0x02441453,0xD8A1E681,0xE7D3FBC8,
    0x21E1CDE6,0xC33707D6,0xF4D50D87,0x455A14ED,0xA9E3E905,0xFCEFA3F8,0x676F02D9,0x8D2A4C8A,
    0xFFFA3942,0x8771F681,0x6D9D6122,0xFDE5380C,0xA4BEEA44,0x4BDECFA9,0xF6BB4B60,0xBEBFBC70,
    0x289B7EC6,0xEAA127FA,0xD4EF3085,0x04881D05,0xD9D4D039,0xE6DB99E5,0x1FA27CF8,0xC4AC5665,
    0xF4292244,0x432AFF97,0xAB9423A7,0xFC93A039,0x655B59C3,0x8F0CCC92,0xFFEFF47D,0x85845DD1,
    0x6FA87E4F,0xFE2CE6E0,0xA3014314,0x4E0811A1,0xF7537E82,0xBD3AF235,0x2AD7D2BB,0xEB86D391]

s = [7,12,17,22,7,12,17,22,7,12,17,22,7,12,17,22,
    5,9,14,20,5,9,14,20,5,9,14,20,5,9,14,20,
    4,11,16,23,4,11,16,23,4,11,16,23,4,11,16,23,
    6,10,15,21,6,10,15,21,6,10,15,21,6,10,15,21]

m = [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,
    1,6,11,0,5,10,15,4,9,14,3,8,13,2,7,12,
    5,8,11,14,1,4,7,10,13,0,3,6,9,12,15,2,
    0,7,14,5,12,3,10,1,8,15,6,13,4,11,2,9]
```

## 实现思路
MD5加密需要经过输入转换、填充、四轮循环操作、置换以及最后的拼接等步骤。其中为了方便循环操作中的值的读取与置换，可以定义一个`MD5`类，并赋予此类一些初始的属性与函数简化后续的函数操作。现将类定义如下：


```python
class md5(object):
    def __init__(self, message):
        self.message = message
        self.A = A
        self.B = B
        self.C = C
        self.D = D
        self.init_A = A
        self.init_B = B
        self.init_C = C
        self.init_D = D
        self.temptext=''
        self.s=s
        self.m=m
        self.T=T
```

对于一个MD5对象，其拥有如下几个主要属性：
* message：  输入的明文
* A,B,C,D：  四轮循环中的操作对象
* temptext： 暂存的结果，用于缓冲与临时存放
* s,m,T:    均为MD5加密时用到的操作参数，以列表的形式存储

## 填充

* 将输入的明文转换成ASCII码，并按照规则进行填充。填充规则如下：
* 当转换的ASCII码的长度的二进制长度不为$512*n+448$的整数倍时，先在末尾补一个“1”；
* 当此时的长度依然不为$512*n+448$时，继续在末尾补“0”直至长度为$512*n+448$
* 将输入密文的长度转换成64位的二进制数，若输入字符串的长度大于$2^{64}$，则取最后的64位。
* 将得到的64位进行小端排序，即按8bit/组进行倒序排列
* 将小端排序得到的结果拼接在步骤3得到的字符后，完成填充

完成此项工作需要一个函数将输入的十进制字符串转换城标准长度的二进制数，代码如下：


```python
def decToBin(num):  # 十进制转标准4位二或8位进制字符串函数 注意此为专用函数不要用于其他程序中
    arry = []
    while True:
        arry.append(str(num % 2))
        num = num // 2
        if num == 0:
            break
    while len(arry)%4!=0:
        arry.append('0')
    return "".join(arry[::-1])  # 列表切片倒叙排列后再用join拼接
```

下面进行填充的操作。将填充的方法归属于MD5对象专用，以防意外调用：


```python
    def fill(self):
        bininput = ''
        raw_input=self.message
        for i in range(len(raw_input)):
            c = decToBin(ord(raw_input[i]))
            bininput += c
        if (len(bininput) % 512 != 448):
            if ((len(bininput) + 1) % 512 != 448):
                bininput += '1'  # 补1
            while (len(bininput) % 512 != 448):
                bininput += '0'  # 其余位补0直到满足长度为512*n+448
        length = len(raw_input) * 8  # ASCII码位数
        if length <= 255:
            length = decToBin(length)  # 二进制转换，此时length保留位数为8
            while len(length) < 8:
                length = '0' + length
        else:
            temp = decToBin(length)
            while len(length) < 16:
                length = '0' + length
            length = length[8:12] + length[12:16] + length[0:4] + length[4:8]
        bininput += length
        while len(bininput) % 512 != 0:
            bininput += '0'
        self.temptext=bininput
```

## 4轮循环
### 分解消息子块
下面将得到的临时变量进行分组。将其分解成$16*32$的矩阵，并将其存放于`Mlist`中。后续将配合MD5类中的m矩阵进行操作。


```python
    def Cycolyce(self):
        Mlist=[]#分16组
        bininput=self.temptext
        for i in range(0,512,32):#16
            sym=''
            for j in range(0,len(bininput[i:i+32]),4):
                temp=hex(int((bininput[i:i+32][j:j+4]),2))#取十六进制
                sym+=temp[2]#只取字母 忽略前面的0x 为下面的小端排序做准备

            sym_tmp=''
            for j in range(8,0,-2):#小端排序
                temp=sym[j-2:j]
                sym_tmp+=temp

            fn_sym=''
            for j in range(len(sym_tmp)):
                fn_sym+=decToBin(int(sym_tmp[j], 16))#再次转换成二进制
            Mlist.append(fn_sym)
```

### 四轮循环的操作
我们需要定义如下四个函数：

$$
F(x,y,z)=(x\land y)\lor (\overline x \land z)\\
G(x,y,z)=(x\land z)\lor (y \lor \overline z)\\
H(x,y,z)=x\oplus y \oplus z\\
I(x,y,z)=y\oplus(x\lor \overline z)
$$

以及定义四个操作：

$$
FF(a,b,c,d,Mj,s,Ti)=(F(b,c,d)+a+mj+Ti)<<s\\
GG(a,b,c,d,Mj,s,Ti)=(G(b,c,d)+a+mj+Ti)<<s\\
HH(a,b,c,d,Mj,s,Ti)=(H(b,c,d)+a+mj+Ti)<<s\\
II(a,b,c,d,Mj,s,Ti)=(I(b,c,d)+a+mj+Ti)<<s\\
$$

其中，$<<$是循环左移，$\oplus$是异或操作。在程序实现中，将四个函数定义在MD5类外，四个操作定义在MD5类以内。如此操作有两个原因：

* 定义在外部的函数可以根据需求进行更改，对于封装后的类调用而言更具操作性
* 定义在类内部的操作是MD5的专用函数，必须专类专用
* 二者分离有助于设置断点、检查中间结果进行调试分析

定义完函数以及相关操作后，我们将进行如下操作：



```python
        for j in range(0, 16, 4):
            self.FF(Mlist[m[j]], s[j], T[j])
            self.FF(Mlist[m[j+1]], s[j+1], T[j+1])
            self.FF(Mlist[m[j+2]], s[j+2], T[j+2])
            self.FF(Mlist[m[j+3]], s[j+3], T[j+3])

        for j in range(0, 16, 4):
            self.GG(Mlist[m[16+j]], s[16+j], T[16+j])
            self.GG(Mlist[m[16+j+1]], s[16+j+1], T[16+j+1])
            self.GG(Mlist[m[16+j+2]], s[16+j+2], T[16+j+2])
            self.GG(Mlist[m[16+j+3]], s[16+j+3], T[16+j+3])


        for j in range(0, 16, 4):
            self.HH(Mlist[m[32+j]], s[32+j], T[32+j])
            self.HH(Mlist[m[32+j+1]], s[32+j+1], T[32+j+1])
            self.HH(Mlist[m[32+j+2]], s[32+j+2], T[32+j+2])
            self.HH(Mlist[m[32+j+3]], s[32+j+3], T[32+j+3])


        for j in range(0, 16, 4):
            self.II(Mlist[m[48+j]], s[48+j], T[48+j])
            self.II(Mlist[m[48+j+1]], s[48+j+1], T[48+j+1])
            self.II(Mlist[m[48+j+2]], s[48+j+2], T[48+j+2])
            self.II(Mlist[m[48+j+3]], s[48+j+3], T[48+j+3])
```

## 初值相加与拼接输出

完成四轮循环后，需要将此时的ABCD与初始的ABCD相加，并进行小端排序。将小端排序后的结果拼接即可得到MD5加密的结果。


```python
self.A = (self.A+self.init_A)%pow(2, 32)
        print(self.A)
        self.B = (self.B+self.init_B)%pow(2, 32)
        print(self.B)
        self.C = (self.C+self.init_C)%pow(2, 32)
        print(self.C)
        self.D = (self.D+self.init_D)%pow(2, 32)
        print(self.D)

        answer = ""
        for each in [self.A, self.B, self.C, self.D]:
            each = hex(each)[2:]
            for i in range(8, 0, -2):
                answer += str(each[i - 2:i])
        return answer
```

我们可以运行一下检验成果：


```python
MD=md5('abc')
MD.fill()
result=MD.Cycolyce()
print(result)
```
