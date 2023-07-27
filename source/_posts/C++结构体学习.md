---
title: C/C++核心语法学习（二）
mathjax: false
tags:
- 结构体
- C++
- C
- 类
categories:
- 学习记录
- C++
---


![banner](https://s2.loli.net/2023/05/03/aNE6XkqKnSY2oQM.jpg)

本篇主要是结构体的介绍与使用。通过实现一个链表的结构，学习存储期的概念以及全局与局部的更深一层的理解。

<!-- more -->

## 结构体基础

### 结构体的使用场景

结构体的本质是通过组合已有的数据类型来构建新的类型。它的使用语法如下：

```c++
struct Student {
    unsigned int number;
    char name[30];
};
```

这里构建了一个`Student`类型的结构体，之后可以使用：

```c++
Student a, b;
```

进行调用。如果要对其内容进行声明与定义，则可以这样：

```c++
a.number = 12345;
// 令 a.name 为 "Bob"，但是数组不能直接赋值，所以只能逐元素处理
a.name[0] = 'B';
a.name[1] = 'o';
a.name[2] = 'b';
a.name[3] = '\0';
```

### 结构体的定义

结构体是由若干个数据“合起来”组成的一种新类型。这些数据被称为结构体的**成员**（Member）。若想定义一个新的结构体类型，你需要这样写：

```
struct 结构体类型名 {
    成员列表
};
```

其中，`结构体类型名` 是一个引入的类型名，它遵循和变量名相同的命名规则。`成员列表` 是一系列可选的带初始化器的声明，比如：

```c++
struct Coordinate {
    int x, y;
    int z{42};
}; // 别忘了这有一个分号
```

一定是先定义再使用！你在定义结构体类型变量时，也可以带一个初始化器。初始化器中是一系列由逗号分隔的值，它们会依次初始化每一个成员。比如：

```c++
struct Student {
    unsigned int number;
    char name[30];
};
Student bob{12345, "Bob"};
```

建议将结构体的定义写在全局作用域内。这样做的原因是，我们可以在所有的函数中使用这个结构体类型：

```c++
#include <iostream>
using namespace std;
struct Student {
    unsigned int number;
    char name[30];
};
void printName(Student student) {
    cout << student.name << endl;
}
int main() {
    Student bob{12345, "Bob"};
    printName(bob);
}
```

### 结构体成员访问

访问一个结构体的成员需要通过成员运算符 `.` 来实现，甚至可以变为指针类型：

```c++
#include <iostream>
using namespace std;
struct Student {
    unsigned int number;
    char name[30];
};
int main() {
    Student a{10001, "Alice"};
    Student* p{&a};
    cout << (*p).name << endl;
}
```

### 一个结构体的例子

链表，不再赘述这种数据结构了。我们给出实现：

```c++
struct Node {
    int data;
    Node* next; // 指向下一节点的指针
};
int main() {
    Node a{12};
    Node b{99};
    Node c{37};
    a.next = &b;
    b.next = &c;
    c.next = nullptr;
}
```

但是再实现链表功能前我们还需要一些别的知识前置：

## 存储期

在学习如何建立一个链表之前，我们首先要学习一个叫做**存储期**（Storage duration）的概念。每个变量都拥有一个存储期，指出在什么时候为这个变量分配内存，在什么时候将这个变量的内存释放

### 自动存储期

在复合语句和形参中声明的局部变量拥有自动存储期（Automatic storage duration）。自动存储期的变量在**进入复合语句**（或函数体）时分配到内存，在**离开复合语句**（或函数体）时释放内存。

### 静态存储期

全局变量拥有静态存储期（Static storage duration）。不同于自动存储期，它们会在**程序启动**的时候**立即**分配内存。全局变量会在分配内存后进行初始化。它们**直到程序退出**的时候才和程序共同将存储空间释放给操作系统。全局变量若有初始化器，则它的初始化值必须是编译期常量；若没有则零初始化。

### 动态存储期

但是这些都不适合链表。因为链表中每一个节点都是独立的，我们需要**手动控制**何时创建一个节点，何时令一个节点消亡。我们称那些可以在运行期间控制何时分配内存、释放内存的变量拥有动态存储期（Dynamic storage duration）。

我们通过 new 表达式手动**向操作系统申请**一片内存，并进行初始化。

|  运算符  |     名称     |                 作用                  |
| :------: | :----------: | :-----------------------------------: |
|  new T   |  new 运算符  |  获取一个非数组的T类型变量的存储空间  |
| delete p | delete运算符 | 将 `p` 指向的存储空间释放（稍后解释） |

new 表达式 `new T` 的结果是一个指向 `T` 类型的指针，这个指针所存放的地址就是刚刚申请到的那片存储空间。比如：

```c++
int* ptr{nullptr};
ptr = new int;
*ptr = 42;
```

这里，`ptr` 这个指针赋值为 new 表达式申请的地址，可以通过这个指针来对这个刚刚得到的存储空间进行操作。为了体现动态存储期的特点，我们这样写：

```c++
#include <iostream>
using namespace std;
// 获得一个存储空间，并赋值为 value
int* getSpace(int value) {
    int* ptr{new int};
    *ptr = value;
    return ptr;
}
int main() {
    int* ptr{nullptr};
    cout << "Continue?" << endl;
    if (cin.get() == 'y')  {
        ptr = getSpace(42);
        cout << *ptr << endl;
    }
    delete ptr; // 见后文
}
```

这里，`getSpace` 函数用于获取一片空间，并将指向这个空间的指针返回。而 main 函数中，是否获取这个空间取决于运行期间的输入：如果我们的输入是 `y`，则获取空间并存放变量；否则不做任何事情，也不会有任何变量被定义。而且，`ptr` 指向的这块存储空间可以在任何场合下访问：我们既可以在 `getSpace` 函数中给它赋值，也可以在 main 函数中将它的值输出。这是全局变量和局部变量都无法实现的，也是我们之后实现链表的必需手段。

new 表达式在分配内存完成后会自动进行初始化。你可以把初始化器放在 new 表达式中，比如：

```c++
struct Node {
    int data;
    Node* next;
};
void f() {
    Node* head{new Node{42, nullptr}};
}
```

然后再通过delete表达式释放new申请的内存：

```c++
int* ptr{new int};
// [...] 对 *ptr 的操作
delete ptr;
```

当执行 `delete ptr;` 后，`ptr` 指向的内存空间**归还给操作系统**，因此 `*ptr` 成为了非法操作——这片存储空间已经不能再用了！我们在删除链表节点的时候，就需要执行这样的操作。new 到的存储空间用 delete 释放，这是一个必须养成的编程习惯。**如果 new 完不 delete，可能导致严重的内存泄漏问题**。所以请务必细心编写你的代码。

### 小结

![image-20230504104115475](https://s2.loli.net/2023/05/04/qSskVKDWOrRuTIz.png)

## 链表构建

我们尝试来构造一个链表。因为我们需要手动控制每一个节点的增删，所以每一个节点都得是动态存储期存储的。因此每一个节点都需要通过 new 表达式来创建。

```c++
struct Node {
    int data;
    Node* next;
};
```

需要注意的是，在写下 `Node* next;` 这行声明时，`Node` 的定义还没有完成——它只是一个声明。所以，如果在这个时候写下 `Node sth`，作为成员是会报错的。这里能够使用 `Node*` 的原因就是，C/C++ 允许定义不完整类型的指针。先来创建第一个节点。链表的第一个节点称为头结点，用一个名叫 `head` 的指针指向它，并让其中 `data` 初始化为 `0`：

```c++
struct Node {
    int data;
    Node* next;
};
int main() {
    Node* head{new Node{0}};
}
```

然后，定义一个指针 `current`，指向当前正在操作的节点。现在，就让 `current` 和 `head` 都指向头结点：

```c++
struct Node {
    int data;
    Node* next;
};
int main() {
    Node* head{new Node{0}};
    Node* current{head};
}
```

令 `current` 的 `next` 指针指向一个新节点，其 `data` 初始化为 `1`。

```c++
struct Node {
    int data;
    Node* next;
};
int main() {
    Node* head{new Node{0}};
    Node* current{head};
    (*current).next = new Node{1};
}
```

接下来，我们需要把 `current` 指针挪到新节点上。我们现在已经生成了两个节点：第一个是头结点，其 `data` 为 `0`；第二个的 `data` 为 `1`:

```c++
struct Node {
    int data;
    Node* next;
};
int main() {
    Node* head{new Node{0}};
    Node* current{head};
    (*current).next = new Node{1};
    current = (*current).next;
}
```

我们可以用循环的方式完成多个节点的拼接：

```c++
struct Node {
    int data;
    Node* next;
};
int main() {
    Node* head{new Node{0}};
    Node* current{head};
    int n{5}; // 节点个数
    for (int i{1}; i < n; i++) {
        (*current).next = new Node{i};
        current = (*current).next;
    }
    // [...]
}
```

当生成完 n 个节点之后，我们用 `nullptr` 来表示链表的终点，这就完成了链表的构建。上述代码构造了由 n 个节点组成的链表，其中它们存储的数据分别是$ 0\sim n - 1$∼的正整数。但是这个方法不能构建不含任何节点的链表（即 `head` 为 `nullptr`），这种情况需要特殊判断。

```c++
struct Node {
    int data;
    Node* next;
};
int main() {
    Node* head{new Node{0}};
    Node* current{head};
    int n{5}; // 节点个数
    for (int i{1}; i < n; i++) {
        (*current).next = new Node{i};
        current = (*current).next;
    }
    (*current).next = nullptr;
}
```

### 链表的访问

链表可以很快地进行插入和删除操作，但相对地，若想访问其中一个节点就会比数组麻烦不少。如果想访问第 x 个节点，那我们不得不从 `head` 开始，一个一个 `next` 地去找，直到第 x 个为止。

```c++
#include <iostream>
using namespace std;

struct Node {
    int data;
    Node* next;
};

/**
 * 获取第 x 个节点（从 0 开始）
 */
Node* getNode(Node* head, unsigned int x) {
    Node* current{head};
    for (unsigned int i{0}; i < x; i++) {
        current = (*current).next;
    }
    return current;
}

int main() {
    Node* head{new Node{0}};
    Node* current{head};
    int n{5}; // 节点个数
    for (int i{1}; i < n; i++) {
        (*current).next = new Node{i};
        current = (*current).next;
    }
    (*current).next = nullptr;
    Node* thirdNode{getNode(head, 3)};
    cout << (*thirdNode).data << endl;
}
```

### 一种简化的表达

在上面的例子中，我们频繁地使用一种形式的表达式：`(*a).b`，即访问一个指针指向的结构体的成员。C/C++ 为此提供了一个更方便的写法 `a->b`：

| 运算符 | 名称           | 作用            |
| ------ | -------------- | --------------- |
| `a->b` | 指针成员运算符 | 等价于 `(*a).b` |

其实 `->` 这个运算符只是提供了一个“快捷方式”，它实际上仍然是先解地址，然后取得其成员。有了它，我们可以把刚才的遍历代码改成这样：

```c++
#include <iostream>
using namespace std;

struct Node {
    int data;
    Node* next;
};

/**
 * 获取第 x 个节点（从 0 开始）
 */
Node* getNode(Node* head, unsigned int x) {
    Node* current{head};
    for (unsigned int i{0}; i < x; i++) {
        current = current->next;
    }
    return current;
}

int main() {
    Node* head{new Node{0}};
    Node* current{head};
    int n{5}; // 节点个数
    for (int i{1}; i < n; i++) {
        current->next = new Node{i};
        current = current->next;
    }
    current->next = nullptr;
    Node* thirdNode{getNode(head, 3)};
    cout << thirdNode->data << endl;
}
```

### 链表元素的插入

如果我们插入的位置不是头部，那么首先需要拿到插入位置前面的结构体，并令指针 `prev` 指向它。随后，用 `temp` 指向一个新生成的结构体，调整它们 `next` 指向的位置即可。

```c++
Node* temp{new Node{}};
temp->next = prev->next;
prev->next = temp;
```

![图示](https://s2.loli.net/2023/05/04/ojaI42PpiZzVYET.png)

如果插入的位置是头部，则需要修改 `head` 指针指向的位置。

```c++
Node* temp{new Node{}};
temp->next = head->next;
head = temp;
```

### 链表元素的删除

同样地，删除一个节点也需要先拿到前面的节点，令 `prev` 为指向前面那个节点的指针。同样地，令 `temp` 指向要删除的节点，随后调整 `next` 指针。最后别忘记释放 `temp` 这片内存。

```c++
Node* temp{prev->next};
prev->next = temp->next;
delete temp;
```

同理可删除头节点：

```c++
Node* temp{head};
head = temp->next;
delete temp;
```

### 链表的释放

我们使用完一个链表务必要将其所有节点占用的存储空间都释放掉。最典型的方法是，从头到尾一个一个删掉，直到 `nullptr` 尾部为止。

```c++
void freeList(Node* head) {
    Node* current{};
    while (head) {
        current = head;
        head = head->next;
        delete current;
    }
}
```

当链表长度较小的时候，使用递归也可：

```c++
void freeList(Node* head) {
    if (!head) return;
    freeList(head->next);
    delete head;
}
```

## 动态大小数组

在使用 new 表达式时，C++ 并不允许申请数组类型的空间，但是C++ 提供了一个称为 new[] 表达式的东西来实现相关的操作：

| 运算符       | 名称            | 作用                                    |
| ------------ | --------------- | --------------------------------------- |
| `new T[n]`   | new[] 运算符    | 获取 `n` 个连续的 `T` 类型存储空间      |
| `delete[] p` | delete[] 运算符 | 释放 new[] 表达式申请到的内存（见下文） |

new[] 表达式的写法是 `new T[n]`，它可以申请 `n` 个连续的 `T` 类型的空间并初始化，最后将其中首个空间的地址作为表达式的结果值。所以，new[] 的返回值类型为 `T* `类型的地址：

```c++
int* ptr{new int[4]};
```

这里，`ptr` 指向刚刚申请到的 4 个连续的 `int` 存储空间中的首个 `int`。不过仍然注意，这里的 `ptr` 仍是一个指针而非一个数组：

```c++
sizeof(ptr); // 等价于 sizeof(int*)，而非 sizeof(int[4])
```

注意：`new[]` 完不 `delete[]` 会导致严重的内存泄漏问题。`new[]` 完只 `delete` 是不够的，因为还有 `n - 1` 块后续内存未被释放。new[] 和 delete[] 表达式是 C++ 中实现动态大小“数组”的唯一方式。而且，无法通过这种方式创建在“两个维度上动态”的“数组”。

## 一些注意事项

如果用C来声明并定义一个结构体，需要在类型说明符前加上详述类型说明符 `struct`。比如：

```
struct Coord {
    int x, y;
};
Coord a;        /* C 中错误，C++ 中 OK */
struct Coord a; /* C 和 C++ 都合法 */
```

即你在使用结构体名字前必须再额外写一个 `struct`，否则 C 编译器无法理解这个名字。你可以通过 typedef 声明来避免这个啰嗦：

```c++
struct Coord_impl {
    int x, y;
};
typedef struct Coord_impl Coord;
Coord a; /* 合法 C */
```

或者直接：

```c++
typedef struct {
    int x, y;
} Coord;
Coord a; /* 合法 C */
```

除此之外，C和C++还有如下的不同之处：

除了上述内容，C 和 C++ 的不同之处还有：

- 结构体不引入作用域（C++ 中引入）
- 关键字 `inline` 含义为优先内联（C++ 中为[容许重复定义](https://learn-cpp.tk/ch09/inline)）
- 全局作用域只读变量拥有外部连接（C++ 中拥有内部连接）
- 对齐相关关键字 `_Alignas` `_Alignof`（C++ 中 `alignas` `alignof`）
- 静态断言 `_Static_assert`（C++ 中 `static_assert`）
- 线程存储期 `_Thread_local`（C++ 中 `thread_local`）
