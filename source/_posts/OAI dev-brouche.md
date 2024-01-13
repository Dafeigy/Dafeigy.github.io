## Clone,Build and Run

### Clone

首先需要安装git，然后再`bash`里输入如下指令：

```bash
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
```

### Build

可以参考`./doc/BUILD.md`中的内容。一般来说进行开发需要安装UHD驱动以及各种依赖库。前者的安装可参考[此链接](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_COTS_UE.md?ref_type=heads#build-uhd-from-source)，后者可使用如下命令安装必需依赖：

```bash
cd ~/openairinterface5g/cmake_targets
./build_oai -I
```

> 这一步可能需要挂上代理。使用`export(Linux)或set(Windows) https_proxy`命令配置代理。

#### 编译gNB和nrUE

高效编译gNB和nrUE使用如下指令：

```bash
./build_oai -w USRP --ninja --gNB --nrUE -c
```

下面式一些编译的选项说明，你也可以通过`./build_oai -h`指令查看：

- `-I`: 安装所有的必需库，一般来说只需要安装一次。
- `-w`: 用于选择射频设备前端。可选`USRP`或者`SIMU`，后者是单独构建RFSimulator，前者是除了USRP外也同时构建RFSimulator，所以一般我们选择`USRP`。
- `--gNB`: 用于构建`nr-softmodem`和`nr-cuup`的可执行文件以及其所需的库。
- `--nrUE`: 用于构建`nr-uesoftmodem`的可执行文件以及其所需的库。
- `--ninja`: 使用`ninja`构建工具，可以加速编译过程。
- `-c`: 清空工作区，并强制进行一个完全的构建。

#### 额外的构建

额外的工具有`telnet` ,` enbscope` ,` uescope` ,` nrscope` 以及` nrqtscope`。这里常用的有`nrscope`和`nrqtscope`。他们可以按照如下的格式构建：

```shell
./build_oai --build-lib all # build all
./build_oai --build-lib telnet  # build only telnet
./build_oai --build-lib "telnet enbscope uescope nrscope nrqtscope"
./build_oai --build-lib telnet --build-lib nrqtscope
```

如果使用`nrscope`，则在运行`nr-softmodem`或`nr-uesoftmodem`的时候配上`-d`参数。如果使用`nrqtscope`，则需要添加`--dqt`参数。

> 注意，如果要使用`nrqtscope`，需要预装Qt5的库以及相关依赖：
>
> ```
> sudo apt-get install build-essential
> sudo apt-get install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
> sudo apt-get install libqt5charts5-dev
> ```
>
> 否则会报错。
>

## Efficient Commands/Tools

### AI Tools

目前的AI生产力工具非常多，耳熟能详的如GPT3.5及其后的各种版本，谷歌的Bard，还有有更长上下文内容的Claude。你可以选择自己部署这些模型，或者用其他方法使用以提高你的开发效率。我使用Vercel托管了anse-app，并使用api2d作为api代理以获得国内的流畅体验。

### grep

grep命令是在Unix和类Unix操作系统中常用的文本搜索工具。它用于在文件或标准输入中查找匹配某个模式的行，并将匹配的行打印出来。grep命令的基本语法如下：

```
grep [选项] 模式 [文件...]
```

常用的选项包括：
- `-i`：忽略大小写的匹配。
- `-v`：打印不匹配模式的行。
- `-r`：递归地在目录层级中搜索匹配的行。
- `-l`：只打印包含匹配的文件名，而不是匹配的行。
- `-n`：打印匹配的行及其行号。

模式可以是简单的字符串，也可以是正则表达式。grep将搜索包含模式的行，并将其打印出来。

使用示例：
1. 在文件中搜索匹配的行：
```
grep "pattern" file.txt
```
2. 忽略大小写搜索：
```
grep -i "pattern" file.txt
```
3. 递归地搜索目录中的文件：
```
grep -r "pattern" directory/
```
4. 打印不匹配模式的行：
```
grep -v "pattern" file.txt
```
5. 打印匹配的行及其行号：
```
grep -n "pattern" file.txt
```
6. 只打印包含匹配的文件名：
```
grep -l "pattern" directory/
```

以上是grep命令的基本介绍和使用示例，它是一种强大的文本搜索工具，可以快速定位和提取所需的信息。



OAI的代码构成非常庞大，但是幸运的是基本所有的功能函数都是使用C进行编写的。在掌握了OAI的代码风格以及命名习惯后，是可以通过`grep`命令来搜索一些感兴趣的函数、变量、参数等。举个例子，我们可以在`openairinterface5g`的根目录下运行如下命令：

```bash
grep -ir "SRS" --include=*.c .
```

这样就可以找到`.c`源码文件中包含”SRS“文本的内容了。

### GNU Project Debugger

OAI的代码推荐使用VSCode进行代码的编写。作为宇宙第一优秀的IDE，VSCode提供了非常多方便的插件以及各种的语言支持加速我们的代码编写。在这个基础上，其对GDB这一工具的支持也非常到位。仅需再OAI的根目录下的`.vscode`文件夹中修改`launch.json`文件即可调试构建好的程序文件，并自定义若干个命令方便进行测试。举个例子：

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "公网终端-测试1",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/cmake_targets/ran_build/build/nr-uesoftmodem",
            "args": [
                "-r","273","--numerology","1","--band","78","-C",
                "3408960000","--ue-fo-compensation","--ue-rxgain","75","--nokrnmod","1",
                "--sa",
                "--ssb","150",
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}/cmake_targets",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        },
        {
            "name": "公网终端-测试2",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/cmake_targets/ran_build/build/nr-uesoftmodem",
            "args": [
                "-r","273","--numerology","1","--band","78","-C",
                "3408960000","--ue-fo-compensation","--ue-rxgain","75","--nokrnmod","1",
                "--sa",
                "--ssb","1518",
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}/cmake_targets",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

上述这两条在configurations字段的配置信息，等价于在`./cmake`目录下执行了如下命令：

```bash
## 公网终端-测试1

sudo ./ran_build/build/nr-uesoftmodem -r 273 --numerology 1 --band 78 -C 3450000000 --ue-fo-compensation --sa --ue-rxgain 75 --ue-txgain 10 --usrp-args "addr=192.168.20.2" --ue-nb-ant-rx 1 --ue-nb-ant-tx 1 --ssb 150

## 公网终端-测试2

sudo ./ran_build/build/nr-uesoftmodem -r 273 --numerology 1 --band 78 -C 3408960000 --ue-fo-compensation --sa --ue-rxgain 75 --ue-txgain 10 --usrp-args "addr=192.168.20.2" --ue-nb-ant-rx 1 --ue-nb-ant-tx 1 --ssb 1518
```

在配置完`launch.json`文件后，按下`Ctrl+Shift+D`，随后找到并选择需要调试的对象：

![image-20231108180946108](https://s2.loli.net/2023/11/08/buaxynQorRcmt3W.png)

随后点击绿色的播放箭头即可开始gdb调试，可以在预设的位置打好断点，观察变量以及堆栈、线程调用的情况。
