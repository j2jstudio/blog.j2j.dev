---
title: "Ssdb"
date: 2021-04-25T14:41:08+08:00
draft: true
---


## 安装

```
wget --no-check-certificate https://github.com/ideawu/ssdb/archive/master.zip
unzip master
cd ssdb-master
make
# optional, install ssdb in /usr/local/ssdb
sudo make install

```

## 启动



```

cd  /usr/local/ssdb

# start master
./ssdb-server ssdb.conf

# or start as daemon
./ssdb-server -d ssdb.conf

```


## 添加ssdb开机自动启动

1. 添加ssdb服务


cd ssdb-master/tools

vi ssdb.sh


文件内容如下:

```
# 修改安装位置
ssdb_root=/usr/local/ssdb
ssdb_bin=$ssdb_root/ssdb-server
# 修改配置文件位置
# each config file for one instance
# configs="/data/ssdb_data/test/ssdb.conf /data/ssdb_data/test2/ssdb.conf"
# configs="/data/ssdb_data/test/ssdb.conf"
configs="/usr/local/ssdb/ssdb.conf"

... 

```

把 tools/ssdb.sh 脚本放到 /etc/init.d 目录下.

注意: 对于 CentOS 用户, 请将  ssdb.sh 重命名为  ssdb.

```
sudo cp ssdb.sh /etc/init.d/ssdb

```

**CentOS**

```
sudo chkconfig --add ssdb
sudo chkconfig ssdb on

sudo systemctl start ssdb

```

**Ubuntu**

```
sudo chmod a+x /etc/init.d/ssdb
sudo update-rc.d ssdb defaults

```


## 可视化工具  


vscode-database-client


https://marketplace.visualstudio.com/items?itemName=cweijan.vscode-mysql-client2



