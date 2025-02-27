---
title: Protobuf学习
mathjax: false
tags:
- Protobuf
categories:
- 学习记录
---


Google Protocol Buffer( 简称 Protobuf) 是 Google 公司内部的混合语言数据标准，目前已经正在使用的有超过 48,162 种报文格式定义和超过 12,183 个 .proto 文件。他们用于 RPC 系统和持续数据存储系统。Protocol Buffers 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

<!-- more -->

## Protobuf安装

用apt安装

```bash
sudo apt install protobuf-compiler
```

## Protobuf使用

### .proto的书写

最好将`.proto`文件命名成如下格式：`packageName.MessageName.proto`。

```protobuf
package lm;
message helloworld
{
   required int32     id = 1;  // ID
   required string    str = 2;  // str
   optional int32     opt = 3;  //optional field
}
```

`package`会变成命名空间，然后我们定义的消息叫做`helloworld`，这个消息有三个成员 ` id ` ,  ` str `

 以及 ` opt `，其中最后一项的`optional`是可选项，消息可以不包含该成员。

### .proto文件的编译

写好protobuf文件后，假设文件存放在`$SRC_DIR`下，则有：

```bash
$ protoc -I=$SRC_DIR --xxx_out=$DST_DIR   $SRC_DIR/yyy.proto

# 参数说明
# 1. $SRC_DIR：指定需要编译的.proto文件目录 (如没有提供则使用当前目录)
# 2. --xxx_out：xxx根据需要生成代码的类型进行设置
	"""
	对于 Java ，xxx =  java ，即 -- java_out
	对于 C++ ，xxx =  cpp ，即 --cpp_out
	对于 Python，xxx =  python，即 --python_out
	"""
# 3. $DST_DIR ：编译后代码生成的目录 (通常设置与$SRC_DIR相同)
# 4. 最后的路径参数：需要编译的.proto 文件的具体路径
# 5. yyy.proto: 代表要编译的。proto文件
```

一个使用示例：

```bash
protoc --python_out=. ./test.proto
protoc --cpp_out=. ./test.proto
```

## protobuf的使用

在CMake中使用的时候需要注意下目标文件和`./`之间要有一个空格，即`./ *.proto`：

```cmake
find_package(Protobuf REQUIRED)
if (PROTOBUF_FOUND)
    message("protobuf found")
else ()
    message(FATAL_ERROR "Cannot find Protobuf")
endif ()

set(protobuf_generated_dir ${OPENAIR_BIN_DIR})
set(SRS_TRAN_MSG_DIR ${OPENAIR1_DIR}/SCHED_NR/MESSAGES )
set(SRS_TRAN_MSG_FILES
  ${SRS_TRAN_MSG_DIR}/ srs-info.proto
  )

execute_process(COMMAND protoc -I ${SRS_TRAN_MSG_FILES} --cpp_out ${SRS_TRAN_MSG_DIR} --python_out ${SRS_TRAN_MSG_DIR})
# nFAPI
```

