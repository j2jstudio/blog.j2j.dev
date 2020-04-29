---
title: "Read Go Ethereum Source Code 1"
date: 2018-03-26T09:33:48+08:00
draft: false
tags:
  - 源码阅读
  - Golang
  - Ethereum
---

 


如何理解系统的设计？最好的方法当然是：

>Read The Fucking Source Code

## Go Ethereum 是什么

以太坊从项目早起，就有不同操作系统下的多客户端实现。这些客户端可以互相验证以太坊的协议正确性。go-ethereum 是以太坊协议的go语言实现的客户端。

截止2016年9月，[go-ethereum](http://www.ethdocs.org/en/latest/ethereum-clients/go-ethereum/index.html#go-ethereum)(go语言实现) 和 [Parity](http://www.ethdocs.org/en/latest/ethereum-clients/parity/index.html#parity)(Rust语言实现) 是领先的实现方案。 

以太坊的协议标准在其[黄皮书](https://github.com/ethereum/yellowpaper)中定义。 

go-ethereum 由 [Jeffrey Wilcke](https://github.com/obscuren) 领导开发，项目启动与2013年，开发语言为Go。go-ethereum在2015年7月30日成功发布，标志着创世区块和以太坊平台的发布。 

Jeff Wilcke 为以太坊的创始人之一，并是以太坊的三位董事之一。

<!--more-->

## 源码下载 

从github网站下载源码：

```
git clone https://github.com/ethereum/go-ethereum.git

```

查看提交记录：

```
cd go-ethereum/

git log --reverse

commit 5db3335dce766bd679c54ea44f6df08a7ff74762
Author: obscuren <obscuren@obscura.com>
Date:   Thu Dec 26 12:45:52 2013 +0100

    Initial commit

commit f77424d3fe89d6ece9bc5d24f784c648306d611a
Author: obscuren <obscuren@obscura.com>
Date:   Thu Dec 26 12:46:02 2013 +0100

    Initial commit

commit f201547731aca98ae24a62b816a9957927481a2f
Author: obscuren <obscuren@obscura.com>
Date:   Thu Dec 26 12:47:06 2013 +0100

    added git ignore

commit fe5577f59e0b5b254013263675e7a0989e7cd82a
Author: obscuren <obscuren@obscura.com>
Date:   Thu Dec 26 13:29:45 2013 +0100

    Added readme

```
看到初始提交记录为： 5db3335dce766bd679c54ea44f6df08a7ff74762

切换源码到初始提交：

```
git checkout 5db3335dce766bd679c54ea44f6df08a7ff74762

Note: checking out '5db3335dce766bd679c54ea44f6df08a7ff74762'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 5db3335dc... Initial commit
```

## 阅读第一次提交的源码
第一次提交时间2013/12/26 （东一区), 记录如下：


```
git log 5db3335dce766bd679c54ea44f6df08a7ff74762
commit 5db3335dce766bd679c54ea44f6df08a7ff74762

Author: obscuren <obscuren@obscura.com>
Date:   Thu Dec 26 12:45:52 2013 +0100

    Initial commit
```

文件如下 

| 文件名        | 说明    |  |
| --------   | -----:   | -----:   |
| big.go     | 大数字操作     |  |
| transaction.go | 交易(tx)的struct       |    |
| block.go        | 区块链中的块（block）结构      |   |
| block_manager.go   | 块管理     |   |
| parsing.go        | 将代码语句编码      |   |
| serialization.go  | 序列化，实现rlp编码      |   |
| ethereum.go        | 程序main函数入口     |   |

### 编译&运行

```
cd go-ethereum     #进入目录
go build       		#编译
./go-ethereum 		#执行

#输出如下：

init Ethereum VM
stack size = 256

# processing Tx (8fee28c5311d91212d92cbf14548e9e96ab39a)
# fee = 0.000000, ops = 12, sender = 1234567890, value = 20

# processing Tx (3ab78afb9e495acc6eabd8982730dbb679db2f)
# fee = 0.000000, ops = 2, sender = 1234567890, value = 20
0   67 [10 6 0 0 0 0]
0   67 [10 6 0 0 0 0]
# finished processing Tx

2   66 [10 10 0 0 0 0]
3   67 [255 7 0 0 0 0]
4   81 [20 255 0 0 0 0]
... JMP 7 ...
7   66 [30 31 0 0 0 0]
8   67 [255 22 0 0 0 0]
9   81 [31 255 0 0 0 0]
... JMP 22 ...
# finished processing Tx

rlp encoded Tx "\x01\x00\x010\x00\n1234567890\x00\x010\x00\x0220\x00\x010\x01\x00\x06395843\x00\x06657986\x00\t335612448\x00\x06524099\x00\b16716881\x00\x010\x00\b13114947\x00\a2039362\x00\a1507139\x00\b16719697\x00\a1048387\x00\x0565360"

```

执行过程中偶尔会报错误：

```
fatal error: concurrent map writes
```

是因为go语言版本 处理并发读写的变动有关。 暂时忽略这个问题。 或者修改 ethereum.go 的28行，设置blck []*Transaction数组中仅传一个对象。 


### big.go
 
 以太坊中操作的数字，取值范围较大。基本类型的int、int64、float等无法存储。所以需要golang的  ["math/big"](https://golang.org/pkg/math/big/) 包进行大精度数字处理。big.go中封装了两个快捷函数，实现
 
 * 字符串转big.Int  Big(num string) *big.Int 
 * int类型数字的指数操作 BigPow(a,b int) *big.Int。

### transaction.go

 定义了区块中交易的数据结构。 结构如下：
 
 ```
 type Transaction struct {
  sender      string
  recipient   uint32
  value       uint32
  fee         uint32
  data        []string
  memory      []int
  signature   string

  addr        string
}
 
 ```
 
 源码中亦注释了结构体成员的意义、字节大小等
 
```
/* 
Transaction   Contract       Size
-------------------------------------------
sender        sender       20 bytes
recipient     0x0          20 bytes
value         endowment     4 bytes (uint32)
fee           fee           4 bytes (uint32)
d_size        o_size        4 bytes (uint32)
data          ops           *
signature     signature    64 bytes
*/

```

同时定义了处理交易过程中的收费内容：


```
var StepFee   *big.Int = new(big.Int)
var TxFee     *big.Int = new(big.Int)
var MemFee    *big.Int = new(big.Int)
var DataFee   *big.Int = new(big.Int)
var CryptoFee *big.Int = new(big.Int)
var ExtroFee  *big.Int = new(big.Int)

var Period1Reward *big.Int = new(big.Int)
var Period2Reward *big.Int = new(big.Int)
var Period3Reward *big.Int = new(big.Int)
var Period4Reward *big.Int = new(big.Int)
```


以太坊中的矿工处理交易，是需要收取一定费用的。根据如上的一些定义可以粗略看出几种收费类型： 计算步骤、交易费、内存使用费、数据费等等。 可以对比比特币区块链的收费，其主要根据交易数据的大小来判断费用。


### block.go

定义的区块的数据结构 ，区块由交易所组成

```
type Block struct {
  transactions []*Transaction
}

```

### block_mananger.go

区块处理的逻辑。 

```
type BlockManager struct {
  vm *Vm
}

func (bm *BlockManager) ProcessBlock(block *Block) error
func (bm *BlockManager) ProcessTransaction(tx *Transaction, lockChan chan bool)

```

block的处理逻辑就是循环处理block下transaction的过程。 对transaction的处理，调用虚拟机 vm *Vm 对象来处理。 



### serialization.go

用于处理以太坊中的RLP编码。 下一篇专门分析RLP编码。

### parsing.go

将类似于汇编语言的操作：
```
"SET 10 6"
"LD 10 10"
```

指令转换为6字节的Big Int

 Big int equation = op + x * 256 + y * 256\**2 + z * 256\**3 + a * 256\**4 + b * 256\**5 + c * 256\**6 。 

上面第一行代码解析后内容为：

```
十六进制表示：
0x 00 00 00 06 0A 43
转为string后：
395843

反向的十进制表示:

67 10 6 0 0 0 0

```




### vm.go

处理区块链交易的最核心代码。Transaction对象的data，保存的是代码经过parsing.go 转换后的string数组。

"SET 10 6" => 395843(big.Int)=>"395843" (string)

vm处理RunTransaction的简单步骤为：

1. 用map模拟stack栈 vm.stack = make(map[string]string)
2. 为交易分配一段内存，map模拟。vm.memory[tx.addr] = make(map[string]string)
3. 将编码后的内容还原为语句。
4. 获得语句中的操作码、参数。并执行相关操作

 
如

```
67 10 6 0 0 0 0
```

代码的内容如下：

```

定义数组索引
x := 0; y := 1; z := 2; //a := 3; b := 4; c := 5

67 = 0x43 代表代码操作码
10 6 0 0 0 0 代表参数 args = [10,6,0,0,0,0]

```

操作码含义

``` 

	oSET        int = 0x43

```
代码的操作方式为：

```
case oSET:
// Set the (numeric) value at Iy to Rx
vm.stack[args[ x ]] = args[ y ]

```

所有语句处理完毕后，完成交易操作
