---
title: "Read Go Ethereum Source Code 7.1"
date: 2018-04-28T09:40:48+08:00
draft: false
tags:
  - 源码阅读
  - Golang
  - Ethereum
---
 
从 commit f6fa4f88797030b8e83066cb262a68958953974e 后，go-ethereum 将功能比较独立的代码分装为模块了。 

后面的几次提交 分别独立出来了一下几个模块：

* github.com/ethereum/ethwire-go wire协议处理
* github.com/ethereum/ethutil-go  编解码等功能处理
* github.com/ethereum/ethdb-go 数据库处理

所以我们选择模块拆分后，并通过编译的版本分析代码变化。 

```
git checkout 4ea93eba501cf996395e4c0829e21cab77267cf9 

```

```
commit 4ea93eba501cf996395e4c0829e21cab77267cf9 (HEAD)
Author: obscuren <geffobscura@gmail.com>
Date:   Wed Jan 15 23:32:30 2014 +0100

    Updated tests

```


以上三个独立模块，在github上原仓库已经没有了。 有部分用户在github上克隆了一份。我们将代码clone下来并按照原包名放置。 

 <!--more-->

## 取回子模块代码

ethutil-go 和 ethdb-go github上有备份，处理起来比较容易。 

```
cd $GOPATH/src
mkdir -p github.com/ethereum

cd $GOPATH/src/github.com/ethereum


```
### ethutil-go


```
git clone https://github.com/Etherbeard/ethutil-go.git

cd ethutil-go

//获取最早的提交记录 
git log --reverse
commit cee03d5890875534be81c3c6748d8b79047cc1d5 
Author: obscuren <geffobscura@gmail.com>
Date:   Fri Jan 10 21:06:24 2014 +0100


git checkout cee03d5890875534be81c3c6748d8b79047cc1d5 

```

### ethdb-go

同上操作

```
git clone https://github.com/josephyzhou/ethdb-go.git

cd ethdb-go

//获取最早的提交记录 
git log --reverse
commit e2c3c1c9e4a82f37b576d5752297e0e0645bec95 
Author: obscuren <geffobscura@gmail.com>
Date:   Fri Jan 10 22:44:27 2014 +0100

    Moving the ethgo to individual packages
    
git checkout e2c3c1c9e4a82f37b576d5752297e0e0645bec95 

```
