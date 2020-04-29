---
title: "Read Go Ethereum Source Code 2"
date: 2018-03-26T09:40:48+08:00
draft: false
tags:
  - 源码阅读
  - Golang
  - Ethereum
---

 

RLP（递归长度前缀）的目的是编码任意嵌套的二进制数据数组，RLP是以太坊中用于序列化对象的主要编码方法。 RLP的唯一目的是编码结构; 对于编码的数据的具体类型（例如字符串，浮点数）则留给高阶协议自己负责处理。（简单来说编码的二进制数据，代表的是字符串、浮点数还是其他类型。由使用该编码的高级协议定义）;RLP编码的整数必须以大端二进制形式（big endian）表示，且不包含前导零（整数值零等于空字节数组）。 反序列化正整数时如发现前缀有零的必须视为无效。 字符串长度的整数表示也必须以这种方式进行编码，其他部分的整数同样如此。 更多信息可以在[Ethereum黄皮书](https://ethereum.github.io/yellowpaper/paper.pdf) 附录B中找到。

如果有人希望使用RLP对字典进行编码，有两个建议

1. 按照字典顺序使用[[k1,v1],[k2,v2]...]进行编码
2. 或者使用更高级别的[Patricia Tree](https://github.com/ethereum/wiki/wiki/Patricia-Tree)编码

<!--more-->
## 定义（编码)

RLP编码功能需要一个项（item)。 一个项（item)定义如下：

* 一个字符串（即字节数组）是一个item
* 项目列表(a list of item )也是一个项item

例如，空字符串是一个item，包含单词“cat”的字符串也是item，包含任意数量字符串的列表以及更复杂的数据结构也是item，如["cat",["puppy","cow"],"horse",[[]],"pig",[""],"sheep"] 。 请注意，在本文其余部分的上下文中，“string”将用作“一定数量的二进制数据字节数”的同义词; 没有使用特殊的编码，也没有关于字符串内容的知识。

	在Python2 和 Golang 中，string都被定义为二进制数组，数据的保存全为二进制。如需要展示为特定格式的编码字符时，需要指定编解码格式。
	


RLP编码定义如下：

* 对于其值在[0x00, 0x7f]范围内的单个字节，该字节是其自己的RLP编码。
* 如果一个字符串的长度为0-55字节，则RLP编码由一个值为**0x80**的单个字节加上
该字符串后面的字符串长度组成。 第一个字节的范围是[0x80, 0xb7]（[0x80,0x80+55=0xb7]) 。
* 如果一个字符串长度超过55个字节，则RLP编码由一个值为**0xb7**的单个字节加上以二进制形式表示的字符串长度的字节长度，接着是字符串长度，后跟字符串。 例如，长度为1024的字符串将被编码为\xb9\x04\x00后跟该字符串。 第一个字节的范围是[0xb8, 0xbf] 。(1024如何被编码查看下方代码）

* 如果编码的为列表(list/array)，其长度为0-55字节（列表保存的是，所有被RLP编码的二进制数据），则RLP编码由单个字节组成，其值为0xc0加上列表的长度，然后连接项目的RLP编码。 第一个字节的范围是[0xc0, 0xf7] (0xf7-0xc0 = 55) 。
* 如果列表的总长度超过55个字节，则RLP编码由一个值为0xf7的单个字节加上以二进制形式表示的有效长度的字节长度，接着是RLP编码的列表。 第一个字节的范围是[0xf8, 0xff](表示后面最多跟随8字节表示长度的内容），列表的长度最多为2**64次方 。

python代码如下：

```
def rlp_encode(input):
    if isinstance(input,str):
        if len(input) == 1 and ord(input) < 0x80: return input
        else: return encode_length(len(input), 0x80) + input
    elif isinstance(input,list):
        output = ''
        for item in input: output += rlp_encode(item)
        return encode_length(len(output), 0xc0) + output

def encode_length(L,offset):
    if L < 56:
         return chr(L + offset)
    elif L < 256**8:
         BL = to_binary(L)
         return chr(len(BL) + offset + 55) + BL
    else:
         raise Exception("input too long")

def to_binary(x):
    if x == 0:
        return ''
    else: 
        return to_binary(int(x / 256)) + chr(x % 256)

```

Golang 代码如下：

在commit 5db3335dce766bd679c54ea44f6df08a7ff74762 初始提交中，rlp的编码代码在serialization.go文件中。其代码暂未支持整数编码。观察代码RLP的实现与如今的RLP规范定义的不大一致。所以我们选取后续的提交。 

在commit d7eca7bcc12e940f0aa80d45e6e802ba68143b5c 中，rlp编解码基本实现完整。以下代码来自 d7eca7bcc12e940f0aa80d45e6e802ba68143b5c rlp.go中。 

```
可通过：
https://github.com/ethereum/go-ethereum/blob/d7eca7bcc12e940f0aa80d45e6e802ba68143b5c/ethutil/rlp.go 在线查看

或者在go-ethereum源码目录中，迁出到指定提交
git checkout d7eca7bcc12e940f0aa80d45e6e802ba68143b5c

```

```


func Encode(object interface{}) []byte {
	var buff bytes.Buffer

	if object != nil {
		switch t := object.(type) {
		case *Value:
			buff.Write(Encode(t.Raw()))
		// Code dup :-/
		case int:
			buff.Write(Encode(big.NewInt(int64(t))))
		case uint:
			buff.Write(Encode(big.NewInt(int64(t))))
		case int8:
			buff.Write(Encode(big.NewInt(int64(t))))
		case int16:
			buff.Write(Encode(big.NewInt(int64(t))))
		case int32:
			buff.Write(Encode(big.NewInt(int64(t))))
		case int64:
			buff.Write(Encode(big.NewInt(t)))
		case uint16:
			buff.Write(Encode(big.NewInt(int64(t))))
		case uint32:
			buff.Write(Encode(big.NewInt(int64(t))))
		case uint64:
			buff.Write(Encode(big.NewInt(int64(t))))
		case byte:
			buff.Write(Encode(big.NewInt(int64(t))))
		case *big.Int:
			buff.Write(Encode(t.Bytes()))
		case []byte:
			if len(t) == 1 && t[0] <= 0x7f {
				buff.Write(t)
			} else if len(t) < 56 {
				buff.WriteByte(byte(len(t) + 0x80))
				buff.Write(t)
			} else {
				b := big.NewInt(int64(len(t)))
				buff.WriteByte(byte(len(b.Bytes()) + 0xb7))
				buff.Write(b.Bytes())
				buff.Write(t)
			}
		case string:
			buff.Write(Encode([]byte(t)))
		case []interface{}:
			// Inline function for writing the slice header
			WriteSliceHeader := func(length int) {
				if length < 56 {
					buff.WriteByte(byte(length + 0xc0))
				} else {
					b := big.NewInt(int64(length))
					buff.WriteByte(byte(len(b.Bytes()) + 0xf7))
					buff.Write(b.Bytes())
				}
			}

			var b bytes.Buffer
			for _, val := range t {
				b.Write(Encode(val))
			}
			WriteSliceHeader(len(b.Bytes()))
			buff.Write(b.Bytes())
		}
	} else {
		// Empty list for nil
		buff.WriteByte(0xc0)
	}

	return buff.Bytes()
}

```

## 例子

* 字符串 "dog" 编码：

```
case string:
		buff.Write(Encode([]byte(t)))

case []byte:
		if len(t) == 1 && t[0] <= 0x7f {
			buff.Write(t)
		} else if len(t) < 56 {
			buff.WriteByte(byte(len(t) + 0x80))
			buff.Write(t)
		} else {
			b := big.NewInt(int64(len(t)))
			buff.WriteByte(byte(len(b.Bytes()) + 0xb7))
			buff.Write(b.Bytes())

len("dog")==3 < 56,
前缀 = byte(3+0x80) = 0x83
跟随字符串"dog" ,最终为[0x83,'d','o','g'] = [0x83,0x64,0x6f,0x67]				
```

* 列表  [ "cat", "dog" ] 
		
		[ 0xc8, 0x83, 'c', 'a', 't', 0x83, 'd', 'o', 'g' ]
		
* 关注int整数的保存 。

		case *big.Int:
			buff.Write(Encode(t.Bytes()))	
		Bytes方法返回的正式是大端序的byte切片 （returns the absolute value of x as a big-endian byte slice.）



## RLP 解码

根据RLP编码的规则和过程，RLP解码的输入应视为二进制数据的数组，过程如下：

1. 根据输入数据的第一个字节（即前缀），并解码数据类型、实际数据的长度和偏移量;

2. 根据数据的类型和偏移，相应解码数据;

继续解码其余的输入;

其中，解码数据类型和偏移的规则如下：

1. 如果第一个字节（即前缀）的范围是[0x00，0x7f]，则该数据是一个字符串,且内容为该字节本身;

2. 如果第一个字节的范围是[0x80，0xb7]，那么数据是一个字符串，长度等于第一个字节减去0x80,字符串跟在第一个字节后面;

3. 如果第一个字节的范围是[0xb8，0xbf]，那么数据是一个字符串，第一个字节减去0xb7等于表示长度的字节数（跟在第一字节后），取得长度字节后得到实际字符串长度，字符串紧跟其后;

4. 如果第一字节的范围是[0xc0，0xf7]，则数据是列表，第一字节减去0xc0后得到列表的长度;

5. 如果第一个字节的范围是[0xf8,0xff]，那么该数据就是一个列表，第一个字节减去0xf7等于表示长度的字节数，根据长度字节数取得列表长度。列表紧跟其后;

Golang 代码如下：

```
// TODO Use a bytes.Buffer instead of a raw byte slice.
// Cleaner code, and use draining instead of seeking the next bytes to read
func Decode(data []byte, pos uint64) (interface{}, uint64) {
	var slice []interface{}
	char := int(data[pos])
	switch {
	case char <= 0x7f:
		return data[pos], pos + 1

	case char <= 0xb7:
		b := uint64(data[pos]) - 0x80

		return data[pos+1 : pos+1+b], pos + 1 + b

	case char <= 0xbf:
		b := uint64(data[pos]) - 0xb7

		b2 := ReadVarint(bytes.NewReader(data[pos+1 : pos+1+b]))

		return data[pos+1+b : pos+1+b+b2], pos + 1 + b + b2

	case char <= 0xf7:
		b := uint64(data[pos]) - 0xc0
		prevPos := pos
		pos++
		for i := uint64(0); i < b; {
			var obj interface{}

			// Get the next item in the data list and append it
			obj, prevPos = Decode(data, pos)
			slice = append(slice, obj)

			// Increment i by the amount bytes read in the previous
			// read
			i += (prevPos - pos)
			pos = prevPos
		}
		return slice, pos

	case char <= 0xff:
		l := uint64(data[pos]) - 0xf7
		b := ReadVarint(bytes.NewReader(data[pos+1 : pos+1+l]))

		pos = pos + l + 1

		prevPos := b
		for i := uint64(0); i < uint64(b); {
			var obj interface{}

			obj, prevPos = Decode(data, pos)
			slice = append(slice, obj)

			i += (prevPos - pos)
			pos = prevPos
		}
		return slice, pos

	default:
		panic(fmt.Sprintf("byte not supported: %q", char))
	}

	return slice, 0
}

```