---
title: "Read Go Ethereum Source Code 4"
date: 2018-04-11T09:40:48+08:00
draft: false
tags:
  - 源码阅读
  - Golang
  - Ethereum
---
 
## Trie.go (Merkle-patricia-tree )

Merkle Patricia Tree(简称MPT树，实际上是一种trie前缀树)是以太坊中的一种加密认证的数据结构，可以用来存储所有的(key，value)对。

以太坊区块链中的区块由标题，交易列表和叔区块列表组成。区块头中包含交易的根哈希，它用于验证交易列表。在区块头部包括了交易的hash树根，用来校验交易的列表。在p2p网络上传输的交易是一个简单的列表，它们被组装成一个叫做trie树的特殊数据结构，来计算根hash。值得注意的是，除了校验区块外，这个数据结构并不是必须的，一旦区块被验证正确，那么它在技术上是可以忽略的。但是，这意味着交易列表在本地以trie树的形式存储，发送给客户端的时候序列化成列表。客户端接收到交易列表后重新构建交易列表trie树来验证根hash。RLP(Recursive length prefix encoding,递归长度前缀编码)，用来对trie树种所有的条目进行编码。对RLP的介绍和源码实现阅读可参考[阅读 go-ethereum 源码 - 2](http://www.cnspirit.com/2018/03/go-ethereum-2.html) 。


Merkle Patricia Tree的Wiki可在[https://github.com/ethereum/wiki/wiki/Patricia-Tree](https://github.com/ethereum/wiki/wiki/Patricia-Tree)查看。

go-ethereum源码中对MPT数的处理时在文件 **trie.go **中。
进入go-ethereum 源码查看trie.go .

```
git checkout 9f133a92d0853102863b77dd7c884d1462cf73a4

或通过如下方式在线浏览：

https://github.com/ethereum/go-ethereum/blob/9f133a92d0853102863b77dd7c884d1462cf73a4/trie.go

```
 <!--more-->
**注意：**

在MPT的wiki中制定，树的Merkle部分是一个节点的确定性加密的hash。 该hash 为 rlp编码的value值的sha3. 

```
The "Merkle" part of the radix trie arises in the fact that a deterministic cryptographic hash of a node is used as the pointer to the node (for every lookup in the key/value DB key == sha3(rlp(value)), rather than some 32-bit or 64-bit memory location as might happen in a more traditional trie implemented in C. 

```

但在go-ethereum的早期代码中，该hash实现为sha256。所以源码中间的计算结果会不同于wiki。在后续的实现中，go-ethereum 将会改为sha3 。

```
// Wrapper around the regular db "Put" which generates a key and value
func (t *Trie) Put(node interface{}) []byte {
	enc := Encode(node)     //RLP 编码
	sha := Sha256Bin(enc)   //sha256

	t.db.Put([]byte(sha), enc)

	return sha
}
```


## HP编码

 在以太坊（ethereum）中，使用了一种特殊的十六进制前缀(hex-prefix, HP)编码，所以在字母表中就有16个字符。这其中的一个字符为一个nibble。
  

我们用'\x01\x01\x02'做为key，'hello'作为值存入到trie树中。trie树将将创建一个新的叶节点。并将[hash, rlp(node)]存储在数据库中。 


" \x01\x01\x02" hp编码为20010102 。 k编码中后6位为**010102**，开头两位**20**为HP编码前缀。第一位**2**是字符串的标志位,用二进制的表示为0b10. 

* 二进制最后一位表示key的长度是否为偶数（0，1，0，1，0，2），设为0
* 二进制第一位表示是否终止（终止为1）

0b00 + 0b10 = 0b10 = byte(2)

HP编码后需要为偶数，所以2后再补充一字节 0 ，最终变为[2,0,0,1,0,1,0,2]

相关编码解码的实现在源码 encoding.go 中。 

相关例子说明：

```
> [ 1, 2, 3, 4, 5, ...]
'11 23 45'  //非终止,奇数。 补充1字节前缀  0b01=1
> [ 0, 1, 2, 3, 4, 5, ...]
'00 01 23 45' //非终止,偶数。 补充2字节前缀  0b00,00 
> [ 0, f, 1, c, b, 8, 10]
'20 0f 1c b8' //终止,奇数。 补充1字节前缀  0b10=2
> [ f, 1, c, b, 8, 10]
'20 f1 cb 8a' //终止,偶数。 补充2字节前缀  0b10=2, 0 

```
## MPT中存储内容结构展示

编写一个简单的展示树结构的代码：

```

func printNode(root string, n []byte, trie *Trie, intent int) {

	tip := ""
	for i := 0; i < intent; i++ {
		tip += " "
	}
	fmt.Printf(tip+"node hash: %s \n", hex.EncodeToString([]byte(root)))
	// Decode it
	currentNode := DecodeNode(n)

	if len(currentNode) == 0 {
		fmt.Printf(tip+"node valu: %v \n", currentNode)

	} else if len(currentNode) == 2 {
		// Decode the key
		k := CompactDecode(currentNode[0])
		v := currentNode[1]

		length := len(k)
		if length == 0 || k[length-1] == 16 {
			//终止
			fmt.Printf(tip+"node valu: %v %v\n", k, v)
		} else {
			//非终止，v = next node hash
			fmt.Printf(tip+"node valu: %v %s\n", k, hex.EncodeToString([]byte(v)))
			n, err := trie.db.Get([]byte(v))

			if nil == err {
				printNode(v, n, trie, intent+1)
			}

		}

	} else if len(currentNode) == 17 {

		fmt.Printf(tip + "node swch:")
		printSwitchNode(currentNode, trie, intent)
	}

}

func printSwitchNode(slice []string, trie *Trie, intent int) {
	fmt.Printf("[")
	for i, val := range slice {
		//fmt.Printf("%q", val)

		if i != len(slice)-1 {
			fmt.Printf("%s", hex.EncodeToString([]byte(val)))
			fmt.Printf(",")
		} else {
			fmt.Printf("%v", val)
		}
	}
	fmt.Printf("]\n")

	for i, val := range slice {
		//fmt.Printf("%q", val)
		if i != len(slice)-1 {
			if val != "" {
				//下个节点hash
				n, err := trie.db.Get([]byte(val))

				if nil == err {
					printNode(val, n, trie, intent+1)
				}
			}
		}
	}
}

```
trie.go的单元测试代码写在trie_test.go 中，我们加入一个新的测试用例。

编写一个测试用例：

```
func TestTrieUpdate2(t *testing.T) {

	db, err := NewMemDatabase()
	trie := NewTrie(db, "")

	if err != nil {
		t.Error("Error starting db")
	}

	//树中加入<64 6f> ‘do' : 'verb'
	trie.Update("do", "verb")
	n, _ := trie.db.Get([]byte(trie.root))
	printNode(trie.root, n, trie, 0)

}

```
执行测试：
```
go test -run TestTrieUpdate2
```
输出为：

```
node hash: 40974ec0af4f0e1d466b3e0c6eb8e8c6cb99aa2a349c23b268a5cf65b6221782 
node valu: [6 4 6 15 16] verb
```

树中再次加入：trie.Update("dog", "puppy")

输出:

```
node hash: f7ea383e3a2acd742393382c3d5836cf7379b45dbfbc35cd7076001d3bdf51b7 
node valu: [6 4 6 15] 80b3283481f857ad774e1d932cad83a9b1433de221ec468a63cae62eda2a7217
 node hash: 80b3283481f857ad774e1d932cad83a9b1433de221ec468a63cae62eda2a7217 
 node swch:[,,,,,,9aa7cda514c3d6a2b65947dd8fa85f244d037f4572bd22984057a3e345597955,,,,,,,,,,verb]
  node hash: 9aa7cda514c3d6a2b65947dd8fa85f244d037f4572bd22984057a3e345597955 
  node valu: [7 16] puppy
```

加入: trie.Update("doge", "coin")

输出:

```
node hash: ca34cb14ee0fb6b039e2c26189442b7dd2f2c1cd575cc896ee49e77108e9e6eb 
node valu: [6 4 6 15] 2a937541cb3a17823d0a2f7adf9d8b355a5240ef99046e0d3485076320b41d7a
 node hash: 2a937541cb3a17823d0a2f7adf9d8b355a5240ef99046e0d3485076320b41d7a 
 node swch:[,,,,,,552cf872957cc9cbf82a53e5b89f37f867685d28105acebba60a03ac7f321147,,,,,,,,,,verb]
  node hash: 552cf872957cc9cbf82a53e5b89f37f867685d28105acebba60a03ac7f321147 
  node valu: [7] da821f38c5d331e760a88ae3a14a42c70618463d4509ff0822ad7014e7b388f0
   node hash: da821f38c5d331e760a88ae3a14a42c70618463d4509ff0822ad7014e7b388f0 
   node swch:[,,,,,,31ffd3d3e73ef63a36f6aaf751e53d8171605db7e13b15cf8525df259739d389,,,,,,,,,,puppy]
    node hash: 31ffd3d3e73ef63a36f6aaf751e53d8171605db7e13b15cf8525df259739d389 
    node valu: [5 16] coin
```

加入: trie.Update("horse", "stallion")

输出:

```
node hash: 6529010d2a2f633bfe03e7a3a3503e5a5bccd1ca49989ac0fb195fd022c6cc93 
node valu: [6] 81a164dfe985a4a6f330883c8c713ca48f06976779a0f9fcf43cfe2f2d8872a5
 node hash: 81a164dfe985a4a6f330883c8c713ca48f06976779a0f9fcf43cfe2f2d8872a5 
 node swch:[,,,,7e627874d222eb965ce0bfd13ef518392bc1c39020afb48a5024f1693449e6ad,,,,94c7c69112a354b2beba2f82c4e53f18aface66280be4ea1dd3f778a7917a373,,,,,,,,]
  node hash: 7e627874d222eb965ce0bfd13ef518392bc1c39020afb48a5024f1693449e6ad 
  node valu: [6 15] 2a937541cb3a17823d0a2f7adf9d8b355a5240ef99046e0d3485076320b41d7a
   node hash: 2a937541cb3a17823d0a2f7adf9d8b355a5240ef99046e0d3485076320b41d7a 
   node swch:[,,,,,,552cf872957cc9cbf82a53e5b89f37f867685d28105acebba60a03ac7f321147,,,,,,,,,,verb]
    node hash: 552cf872957cc9cbf82a53e5b89f37f867685d28105acebba60a03ac7f321147 
    node valu: [7] da821f38c5d331e760a88ae3a14a42c70618463d4509ff0822ad7014e7b388f0
     node hash: da821f38c5d331e760a88ae3a14a42c70618463d4509ff0822ad7014e7b388f0 
     node swch:[,,,,,,31ffd3d3e73ef63a36f6aaf751e53d8171605db7e13b15cf8525df259739d389,,,,,,,,,,puppy]
      node hash: 31ffd3d3e73ef63a36f6aaf751e53d8171605db7e13b15cf8525df259739d389 
      node valu: [5 16] coin
  node hash: 94c7c69112a354b2beba2f82c4e53f18aface66280be4ea1dd3f778a7917a373 
  node valu: [6 15 7 2 7 3 6 5 16] stallion
```

简化显示为：

```
node hash: hash1 
node valu: [6] hash2
 node hash: hash2 
 node swch:[,,,,hash3,,,,hash4,,,,,,,,]
  node hash: hash3 
  node valu: [6 15] hash5
   node hash: hash5 
   node swch:[,,,,,,hash6,,,,,,,,,,verb]
    node hash: hash6 
    node valu: [7] hash7
     node hash: hash7 
     node swch:[,,,,,,hash8,,,,,,,,,,puppy]
      node hash: hash8 
      node valu: [5 16] coin
  node hash: hash4 
  node valu: [6 15 7 2 7 3 6 5 16] stallion
  
```

```
var hashNum int64 = 0
var hashMap = make(map[string]string)

func clearSimpleHash() {
	hashNum = 0
	hashMap = make(map[string]string)
}
func simpleHash(hash string) string {
	if h, ok := hashMap[hash]; ok {
		return h
	}
	hashNum += 1
	nhash := "hash" + strconv.FormatInt(hashNum, 10)
	hashMap[hash] = nhash
	return nhash
}
```

