---
title: "编译安装蓝鲸智云配置平台(BlueKing CMDB) - CentOS8"
date: 2019-12-11T10:31:51+08:00
draft: false
---

## 步骤说明

1. Golang环境安装配置
2. Nodejs环境安装配置
3. 代码编译bk-cmdb
4. 部署ZooKeeper
5. 部署Redis
6. 部署MongoDB
7. 安装bk-cmdb
8. 运行

<!--more-->

## 环境准备

sudo dnf update
sudo dnf groupinstall 'Development Tools'


sudo dnf install python3 python2
sudo alternatives --set python /usr/bin/python3

## Golang环境安装配置

```
wget https://dl.google.com/go/go1.13.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xf go1.13.4.linux-amd64.tar.gz

mkdir ~/go


vim ~/.bash_profile

export PATH=$PATH:/usr/local/go/bin
GOPATH=$HOME/go
export GOPATH

source ~/.bash_profile

```

测试：
```
# go version
go version go1.13.4 linux/amd64
```

## 2. Nodejs环境安装配置

```
yum install nodejs
```


```
# node --version
v10.16.3
```

安装cnpm

```
npm install cnpm -g --registry=https://registry.npm.taobao.org
```


```
npm install cnpm -g --registry=https://registry.npm.taobao.org
/usr/bin/cnpm -> /usr/lib/node_modules/cnpm/bin/cnpm
+ cnpm@6.1.0
```



## 4 Zookeeper

### a. 安装Java

```
yum install java-1.8.0-openjdk
java -version

```

输出如下：
```
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)


```

### b. zookeeper

```
安装：

cd /data
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz

tar -zxvf zookeeper-3.4.14.tar.gz 
cd zookeeper-3.4.14/
mkdir data

配置：
vi conf/zoo.cfg

tickTime = 2000
dataDir = /data/zookeeper-3.4.14/data
clientPort = 2181
initLimit = 5
syncLimit = 2

启动：
./bin/zkServer.sh start

ZooKeeper JMX enabled by default
Using config: /data/zookeeper-3.4.14/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

关闭：
./bin/zkServer.sh stop

Using config: /data/zookeeper-3.4.14/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```

## 5. 部署Redis
  
## 6. 部署MongoDB


vim /etc/yum.repos.d/mongodb.repo

```
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/8/mongodb-org/testing/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

dnf install mongodb-org

检测安装：
mongod --version

```
db version v4.2.2-rc1
git version: a0bbbff6ada159e19298d37946ac8dc4b497eadf
OpenSSL version: OpenSSL 1.1.1 FIPS  11 Sep 2018
allocator: tcmalloc
modules: none
build environment:
    distmod: rhel80
    distarch: x86_64
    target_arch: x86_64
```

启动：

```
systemctl start mongod
systemctl enable mongod
```

测试连接：

mongo

```
> db
test
>
```

顺便配置cmdb数据库：
```
> use cmdb
switched to db cmdb
> db.createUser({user: "cc",pwd: "cc",roles: [ { role: "readWrite", db: "cmdb" } ]})
Successfully added user: {
	"user" : "cc",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "cmdb"
		}
	]
}
```

## 7. 安装bk-cmdb

不启用全文检索。进入cmdb的安装目录，编译好后将tar文件移动至/data

```
# cd /data/cmdb/
# ls
cmdb_adminserver  cmdb_datacollection  cmdb_operationserver    cmdb_taskserver  cmdb_webserver  init.py     start.sh  upgrade.sh
cmdb_apiserver    cmdb_eventserver     cmdb_procserver         cmdb_tmserver    docker          ip.py       stop.sh   web
cmdb_coreservice  cmdb_hostserver      cmdb_synchronizeserver  cmdb_toposerver  image.sh        restart.sh  tool_ctl
```

初始化配置

```
python2 init.py  \
  --discovery          127.0.0.1:2181 \
  --database           cmdb \
  --redis_ip           127.0.0.1 \
  --redis_port         6379 \
  --redis_pass         1111 \
  --mongo_ip           127.0.0.1 \
  --mongo_port         27017 \
  --mongo_user         cc \
  --mongo_pass         cc \
  --blueking_cmdb_url  http://192.168.2.105:8888/ \
  --blueking_paas_url  http://192.168.2.105:7004 \
  --listen_port        8888 \
  --auth_scheme        internal \
  --auth_enabled       false \
  --full_text_search   off \
  --log_level          3

```

./start.sh 

./init_db.sh 
```
{
  "result": true,
  "bk_error_code": 0,
  "bk_error_msg": "",
  "permission": null,
  "data": "migrate success",
  "pre_version": "",
  "current_version": "y3.6.201911261109",
  "finished_migrations": [
   "v3.0.8",
   "v3.0.9-beta.1",
   "v3.0.9-beta.3",
   "v3.1.0-alpha.2",
   "x08.09.04.01",
   "x08.09.11.01",
   "x08.09.13.01",
   "x08.09.17.01",
   "x08.09.18.01",
   "x08.09.26.01",
   "x18.09.30.01",
   "x18.10.10.01",
   "x18.10.30.01",
   "x18.10.30.02",
   "x18.11.19.01",
   "x18.12.12.01",
   "x18.12.12.02",
   "x18.12.12.03",
   "x18.12.12.04",
   "x18.12.12.05",
   "x18.12.12.06",
   "x18.12.13.01",
   "x18_12_13_02",
   "x19.01.18.01",
   "x19.02.15.10",
   "x19.04.16.01",
   "x19.04.16.02",
   "x19.04.16.03",
   "x19.05.16.01",
   "x19_08_19_01",
   "x19_08_20_01",
   "x19_08_26_01",
   "x19_08_26_02",
   "x19_09_03_01",
   "x19_09_03_02",
   "x19_09_03_03",
   "x19_09_03_04",
   "x19_09_03_05",
   "x19_09_03_06",
   "x19_09_03_07",
   "x19_09_03_08",
   "x19_10_22_01",
   "x19_10_22_02",
   "x19_10_22_03",
   "y3.6.201909062359",
   "y3.6.201909272359",
   "y3.6.201910091234",
   "y3.6.201911121930",
   "y3.6.201911122106",
   "y3.6.201911141015",
   "y3.6.201911141516",
   "y3.6.201911261109"
  ]
 }
```