---
title: "Read Go Ethereum Source Code 3"
date: 2018-03-30T09:40:48+08:00
draft: false
tags:
  - 源码阅读
  - Golang
  - Ethereum
---
 

从初始代码提交到commit ad048e9f445ff96b7bfd75c104ab923e1e06754b，go-ethereum的结构&功能变化不大。主要变化有：

* 将rlp编解码移动到 rlp.go文件中
* 完善 transaction、block的rlp编解码

到了commit a926686445929d091c2d9e019b017600168e9e47，源码中出现了较大的功能加入。

```
Added sample server, genesis block, and database interface
```

但是本次提交出现，无法编译通过。错误为：

```
./database.go:65: cannot use k (type []int) as type string in function argument

```

所以迁出下一次提交 并逐一分析源码、功能。

```
git checkout f17930eb4661721cd0e1b92765448c589441907b

或在线浏览：

https://github.com/ethereum/go-ethereum/tree/f17930eb4661721cd0e1b92765448c589441907b
```

 <!--more-->

## 新增功能代码

**server.go**

实现客户端运行的主要框架。

1. for循环实现始终运行
2. db 实现数据的本地操作
3. peers ，能够获取到其他客户端的连接。存储客户端列表。

 
```
type Server struct {
  // Channel for shutting down the server
  shutdownChan chan bool
  // DB interface
  db          *LDBDatabase
  // Peers (NYI)
  peers       *list.List
}

``` 
 

**database.go**

使用google的leveldb本地存储数据，这次提交中方法未完全实现。主要的功能就是CRUD，数据的增删查改。

**trie.go**

根据名称应该是数据结构前缀树或字典树的实现，主要功能也未实现。 

**encoding.go**

该文件并不是新提交的，主要功能是实现十六进制数的紧凑编码。并添加可选的是否终止的符号。 

```
Compact encoding of hex sequence with optional terminator
```

了解一下其实现。

## CompactEncode 紧凑编码

传统的十六进制字符串的编码，一般是将十六进制字符串转为二进制数组。比如'0f1248'将会转换为[15,18,73]3字节的二进制数组。 
但这通常会导致一个问题，如将[15,18,73]解码为十六进制字符串时，应该解码为’0f1248‘ 还是 'f1248'。该方法无法确定是否需要前面的0.

第二个问题，在Ethereum的数据结构Merkle Patricia tree中，需要十六进制字符串可以带一个可选的是否终止符号（常量 16），该终止标志仅出现一次，且在字符串最后。

为了同时解决如上两个问题，ethereum设计了CompactEncode编码。其实现的核心算法为：

将十六进制字符串当做字节数组。 根据数组的长度，在数组前增加1字节或2字节的前缀。 然后将连续两个字节的数值合并为一个字节（即：第一个字节作为前4bit，第二个字节当做后4bit。因数组中最大值为0xf，所以不会产生越界)。

### 前缀的个数

如果数组为基数，则添加1字节前缀。如果为偶数，则添加2字节(第2字节为0）。

### 前缀的内容

前缀主要用来表示两个内容：

* 该字符串是否终止
* 该字符串长度是否为奇数

该实现基于位操作：1byte =8bit（0000 0000）。最后一个bit表示是否奇数（0为偶数、1为奇数）。倒数第二个自己表示是否终止。

```
oddlen := len(hexSlice) % 2
flags := 2 * terminator + oddlen
```

### 编码实现

将连续两个字节的数值合并为一个字节（即：第一个字节作为前4bit，第二个字节当做后4bit)
```
var buff bytes.Buffer
  for i := 0; i < len(hexSlice); i+=2 {
    buff.WriteByte(byte(16 * hexSlice[i] + hexSlice[i+1]))
  }

  return buff.String()
```

### 实例

**如 [ 1, 2, 3, 4, 5 ]**

长度为基数，无终止标志（16）。所以增加前缀一个字节。

该字节内容为： flags = 2 * 0 + 1 = 1 

数组变为：[1,1,2,3,4,5] 。
 
编码[1\*16+1,2\*16+3,4\*16+5] = [17,35,69]=[0x11,0x23,0x45]



**[ 0, 1, 2, 3, 4, 5 ]**

长度偶数，无终止标志。 需要增加两个字节，

则： flags= 2 * 0 + 0 = 0

数组变为：[0,0 , 0, 1, 2, 3, 4, 5 ]

编码：[0\*16+0, 0\*16+1,2\*16+3,4\*16+5] = [0,1,35,69] = [0x00,0x01,0x23,0x45]

**[ 0, 15, 1, 12, 11, 8, 16 ]**

编码为： '\x20\x0f\x1c\xb8'

**[ 15, 1, 12, 11, 8, 16 ]**

编码为： '\x3f\x1c\xb8'


## 解码则为编码的反过程

略过


## 总结

本次主要分析了 CompactEncode 的实现。 database、trie、server因无具体实现。则等待稍后分析。


[原文连接 http://www.cnspirit.com/2018/03/go-ethereum-3.html](http://www.cnspirit.com/2018/03/go-ethereum-3.html)
