---
title: NES 平台
categories: 杂项
tags:
- NES
- 模拟器
- Rust
---

## 

### 架构

软硬件交互的一个简要交互架构如图所示：

<img src="https://bugzmanov.github.io/nes_ebook/images/ch2/image_1_computer_arch.png" alt="img" style="zoom: 50%;" />

从顶部到底部分别是：

- 应用运行着逻辑代码，并通过操作系统（Operating System）与硬件进行交互
- 操作系统使用机器语言与硬件进行通信
- 在硬件层，每一个设备都被看做存储元件、处理单元或者二者的结合体。基于这个观点，NES joypad不过就是一个包含8个1-bit元素的数组，每一个元素对应的就是joypad上的按钮被按下/释放的状态而已。
- 在ALU层以下的层以及存储层并非我们关注的重点。从硬件的角度来看，他们无非就是逻辑门以及对应的变种组合。

> 如果你想获得更多的计算机组成原理的知识，我强烈建议你阅读这本由Noam Nisan和Shimon Schocken 编写的["The Elements of Computing Systems. Building a Modern Computer from First Principles"](https://www.goodreads.com/book/show/910789.The_Elements_of_Computing_Systems).

幸运的是，NES并没有操作系统。这意味着我们的应用层(Gamezzz)直接通过机器语言与硬件进行通信。因此简化后的架构类似下图：

<img src="https://bugzmanov.github.io/nes_ebook/images/ch2/image_2_nes_emul_arch.png" alt="img" style="zoom: 50%;" />

如你所见，机器语言是我们的模拟器和NES游戏的接口。在我们接下来要构建的模拟器中，我们需要实现NES的计算机架构，算术逻辑单元以及内存。使用高级的编程语言，我们不需要关心布尔运算和时序逻辑的问题，我们仅需要使用现有的Rust特性和语言结构即可。

### NES 平台主要组件

![img](https://bugzmanov.github.io/nes_ebook/images/ch2/image_3_nes_components.png)

精简版的NES的主要硬件组成如下：

- 中央处理单元(CPU) - NES的2A03是[6502 芯片](https://en.wikipedia.org/wiki/MOS_Technology_6502)的一个“魔改版”。不管是什么型号的CPU，该模块的作用就是去执行主程序的指令。
- 图像处理单元(PPU) - 也是上面的CPU制造商Ricoh的产品，基于2C02芯片。这个模块的主要目标就是绘制当前游戏状态并将其在一块TV'屏幕上进行展示。
- CPU和PPU都需要从随机接入内存(Random Access Memory,RAM)中获得2 KiB(2046 字节)的空间。
- 音频处理单元(APU) - 这是2A03芯片的一部分，它负责生成特定的5通道的音频（chiptune，也就是目前广为人知的8-bit音乐）。
- 卡带(Cartridges) - 是平台的一个重要组件。由于NES的控制台并没有操作系统，每个卡带至少携带两个大的ROM芯片：角色ROM(CHR ROM)和程序ROM(PRG ROM)。前者用于存储游戏的视觉图像信息，后者则存储CPU的指令集——也就是游戏的代码（事实上，当卡带插入卡槽时，CHR ROM直接就和PPU进行连接，而PRG ROM则直接和CPU连接）。之后版本的卡带携带了额外的硬件（RAM和ROM），这些硬件可以通过一种称为“映射器 (mappers)”的东西进行访问。这也就解释了为何不升级游戏平台，但后续的游戏也可以获得更好的游玩和视觉体验，

<img src="https://bugzmanov.github.io/nes_ebook/images/ch2/image_4_cartridge.png" alt="img" style="zoom:50%;" />

- 游戏手柄 - 用于获取玩家的输入并将其转换为可行的游戏逻辑。我们随后可以认识到，游戏手柄在8-bit的平台中仅仅有8个按钮并不是一个巧合。

有意思的是，CPU、PPU、APU是相互独立的。这一事实使得NES成为一个分布式系统，其中独立的组件必须协调才能生成一个无缝的游戏体验。我们可以用一张图来描述一下NES 的主要组件，并将其作为我们后面的模拟器编写的提纲：

<img src="https://bugzmanov.github.io/nes_ebook/images/ch2/image_6_impl_plan.png" alt="img" style="zoom:50%;" />

我们必须对所有这些模块进行模拟，我们的目标是尽快地可以在模拟器上能“玩”到些什么。使用迭代方法，我们将逐渐增加特性来实现这个目标。粗略地估计了一下每个组件的实现难度，PPU将会是最困难的，而BUS则是最简单。

编写一个完美模拟器是一个永无止境的挑战。但千里之行始于足下，我们马上就会通过编写一个CPU的模拟程序。

![img](https://bugzmanov.github.io/nes_ebook/images/ch2/image_5_motherboard.png)