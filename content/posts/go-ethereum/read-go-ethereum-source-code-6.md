---
title: "Read Go Ethereum Source Code 6"
date: 2018-04-20T09:40:48+08:00
draft: false
tags:
  - 源码阅读
  - Golang
  - Ethereum
---
 
go-ethereum 网络的雏形。 代码仍然停留在

```
commit ba385ccdf15f8379c54d65061c3d62353baffc2b (HEAD)
Author: obscuren <geffobscura@gmail.com>
Date:   Fri Jan 10 10:59:57 2014 +0100

    sudo not udo
```
 <!--more-->
## server.go   

```
type Server struct {
  // Channel for shutting down the server
  shutdownChan chan bool
  // DB interface
  db          *LDBDatabase
  // Block manager for processing new blocks and managing the block chain
  blockManager *BlockManager
  // Peers (NYI)
  peers       *list.List
}
```

Server 结构定义了go-ethereum启动后网络的基本功能。 

### 启动本地网络绑定

```
func (s *Server) Start() {
  // For now this function just blocks the main thread
  ln, err := net.Listen("tcp", ":12345")
  if err != nil {
    log.Fatal(err)
  }

  go func() {
    for {
      conn, err := ln.Accept()
      if err != nil {
        log.Println(err)
        continue
      }

      go s.AddPeer(conn)
    }
  }()
}
```

该方法调用 net 模块，在tcp 12345 端口进行绑定，并等待其他网络连接。 当网络连接到达后，将该连接封装到 Peer 结构内，并保存到列表内。 

```
添加连接节点
func (s *Server) AddPeer(conn net.Conn)
```

### 关闭网络连接

```
func (s *Server) Stop() {
  // Close the database
  defer s.db.Close()

  // Loop thru the peers and close them (if we had them)
  for e := s.peers.Front(); e != nil; e = e.Next() {
    if peer, ok := e.Value.(*Peer); ok {
      peer.Stop()
    }
  }

  s.shutdownChan <- true
}
```

逐一停止列表内的节点，并关闭本地数据库。 


### 广播 

```
func (s *Server) Broadcast(msgType string, data []byte)
```
向所有连接发送消息。

### 主动连接到其他节点

```
func (s *Server) ConnectToPeer(addr string) error

```

## peer.go 

peer.go 封装了 Peer 结构。 主要实现具体网络节点连接之间的消息发送/收取操作。

```
type Peer struct {
  // Server interface
  server      *Server
  // Net connection
  conn        net.Conn
  // Output queue which is used to communicate and handle messages
  outputQueue chan OutMsg
  // Quit channel
  quit        chan bool
}
```

### 启动 

Peer 创建后就会启动。 

```
func (p *Peer) Start() {
  // Run the outbound handler in a new goroutine
  go p.HandleOutbound()
  // Run the inbound handler in a new goroutine
  go p.HandleInbound()
}
```
同时启动协程 分别处理发送（p.HandleOutbound）和 接收（p.HandleInbound）。 两协程循环处理，直至收到退出信号。 

### 发送消息

```
func (p *Peer) WriteMessage(msg OutMsg) {
  // Encode the type and the (RLP encoded) data for sending over the wire
  encoded := Encode([]interface{}{ msg.msgType, msg.data })
  // Write to the connection
  _, err := p.conn.Write(encoded)
  if err != nil {
    log.Println(err)
    p.Stop()
  }
}
```

首先将需要发送的消息，进行RLP编码，然后调用net 库向连接写入数据。

### 接收消息

```
func ReadMessage(conn net.Conn) (*InMsg, error) {
  buff := make([]byte, 4069)

  // Wait for a message from this peer
  n, err := conn.Read(buff)
  if err != nil {
    return nil, err
  } else if n == 0 {
    return nil, errors.New("Empty message received")
  }

  // Read the header (MAX n)
  // XXX The data specification is made up. This will change once more details have been released on the specification of the format
  decoder := NewRlpDecoder(buff[:n])
  t := decoder.Get(0).AsString()
  // If the msgdata contains no data we throw an error and disconnect the peer
  if t == "" {
    return nil, errors.New("Data contained no data type")
  }

  return &InMsg{msgType: t, data: decoder.Get(1).AsBytes()}, nil
}

```

消息到达后，读取，并RLP解码。并交由其他功能模块处理。 



## 测试运行

cd go-ethereum 

go build 

./go-ethereum 

输出：

``` 

2018/04/28 16:09:27 Starting Ethereum
2018/04/28 16:09:27 Connected to peer :: [::1]:12345
2018/04/28 16:09:27 Peer connected :: [::1]:57114

```

在 ethereum.go main()入口函数中，如不传入任何参数。 Server则主要做如下工作：

```
//创建Server
server, err := NewServer()
//注册终止信号
RegisterInterupts(server)
//启动Server
server.Start()

//同时启动一个连接，连向刚才的服务端口12345
err = server.ConnectToPeer("localhost:12345")

```
