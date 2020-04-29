---
title: "Read Go Ethereum Source Code 5"
date: 2018-04-17T09:40:48+08:00
draft: false
tags:
  - 源码阅读
  - Golang
  - Ethereum
---
 
## dagger.go (Dagger工作量证明 )

Ethash 是 Ethereum 的PoW(工作量证明)算法。 该算法需要大量的数据集合，该集合被称为DAG. Ethash 算法由[Dagger-Hashimoto](https://github.com/ethereum/wiki/wiki/Dagger-Hashimoto) 算法改进而得，Dagger Hashimoto是Ethereum 1.0 挖矿算法的推荐规范 。 

Dagger Hashimoto 基于已经存在的关键算法：Hashimoto 和 Dagger。 

在go-ethereum 提交记录 9f133a92d0853102863b77dd7c884d1462cf73a4 中，第一次提交了dagger的实现。 

```
commit 9f133a92d0853102863b77dd7c884d1462cf73a4 (HEAD)
Author: obscuren <geffobscura@gmail.com>
Date:   Wed Jan 8 23:41:03 2014 +0100

    First dagger impl

```
 <!--more-->
## Dagger 工作量证明

Dagger的文档可以在 [https://github.com/ethereum/wiki/blob/master/Dagger.md](https://github.com/ethereum/wiki/blob/master/Dagger.md)查看。

Dagger，由Vitalik Buterin提供的一种算法（January 2014），Dagger命名来自于“directed acyclic graph”（DAG），DAG（有向无环图）是dagger算法的数学结构。 该算法的主要实现内存困难的计算（memory-hard computation )，但内存容易的验证（memory-easy validation）。 简单来说计算需要大量的内存，而验证则只需要少量的内存。 

核心原则是每个随机数只需要大数据树中的一小部分，对于挖矿程序来说，重新计算每个随机数的子树是禁止的 ，所以矿工节点需要因此需要存储树 。但对于单一随机数的验证来说则没有问题。 
Dagger本意是替代现有的内存困难算法，如Scrypt，该算法计是内存困难的，但当增加到真正的安全级别时，也很难验证。 

然而，Dagger 算法被[Sergio Lerner](https://bitslog.wordpress.com/2014/01/17/ethereum-dagger-pow-is-flawed/)证明存在缺陷（可以共享内存，并 导致可硬件加速，无法抵抗ASIC）。随后Ethereum POW逐步演化为 Dagger -> Dagger-Hashimoto -> Ethash 。

### 算法定义

```

D(data,xn,0) = sha3(data)
D(data,xn,n) =
    with v = sha3(data + xn + n)
         L = 2 if n < 2^21 else 11 if n < 2^22 else 3
         a[k] = floor(v/n^k) mod n for 0 <= k < 2
         a[k] = floor(v/n^k) mod 2^22 for 2 <= k < L
    sha3(v ++ D(data,xn,a[0]) ++ D(data,xn,a[1]) ++ ... ++ D(data,xn,a[L-1]))

```

* Dagger 创建了一个有向非循环图（DAG),总计有2**32 -1 = 8388607 个节点。
* 每个节点依赖于前面的3-15个随机的节点
* 如果找到 2\*\*22 至 2\*\*23 见的一个节点的hash值 小于 2**256/diff ，则得到了工作量证明。 （diff 为难度值)


## 代码

```

type Dagger struct {
	hash *big.Int
	xn   *big.Int
}

func (dag *Dagger) Search(diff *big.Int) *big.Int 

func (dag *Dagger) Node(L uint64, i uint64) *big.Int

func (dag *Dagger) Eval(N *big.Int) *big.Int

```

* Search 方法为寻找到制定难度 (diff)的 随机值
* Eval 方法即为算法定义中的函数D 
* Node 方法为计算DAG中的节点数据 


本地提交代码无法运行，所以我们使用测试通过的提交**ba385ccdf15f8379c54d65061c3d62353baffc2b** 

```

git checkout ba385ccdf15f8379c54d65061c3d62353baffc2b

```

运行测试：

```
go test -bench=BenchmarkDaggerSearch
```
输出：
```
goos: darwin
goarch: amd64
pkg: go-ethereum
BenchmarkDaggerSearch-4   	       1	1714457872 ns/op
PASS
ok  	go-ethereum	1.896s
```

发现使用该算法，难度2\*\*36 时，每次查找时间约为1.8秒。 


