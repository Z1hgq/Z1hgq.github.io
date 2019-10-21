---
layout:     post
title:      "mongo远程连接设置"
subtitle:   "mongo远程连接设置"
date:       2019-10-14 16:04:00
author:     "Z1hgq"
header-img: "img/contact-bg.jpg"
catalog: true
tags:
    - mongodb
    - linux
---

> “自然界没有凤凤雨雨，大地就不会有春华秋实”

---
## 服务器设置
#### 0. 创建配置文件`/etc/mongod.conf`
```
bind_ip = 0.0.0.0        #允许访问本机mongo服务的ip
port = 27017             #mongo服务监听的端口
auth=false               #是否开启用户验证，false：不必使用用户名和密码就可以连接
dbpath=/data/db          #mongo的本地数据文件地址
logpath=/log/mongodb.log #mongo日志记录文件地址
logappend=true
fork=true                #后台启动配置
```
#### 1. 重启mongo服务
`ps -ef|grep mongo`查看mongod服务的**pid**，如：

```
root     19978     1  0 15:31 ?        00:00:09 mongod -f /etc/mongod.conf
root     20076 18938  0 16:05 pts/2    00:00:00 vi /etc/mongod.conf
root     20120 18938  0 16:21 pts/2    00:00:00 grep --color=auto mongo
```

这里使用`kill`命令把进程结束掉，pid是第二列。

使用配置文件启动mongo服务`mongod -f /etc/mongod.conf `，这样mongo就在后台运行了。

#### 2.添加用户

使用`mongo`命令进入mongo的控制命令行

```
//添加管理员用户（role:root表示最高权限）
use admin

db.createUser({
    user:"admin",
    pwd:"admin",
    roles:[{
        role:"root",
        db:"admin"
    }]
})

db.auth("admin", "admin")
```

#### 3.设置防火墙允许访问端口

添加端口访问权限`iptables -A INPUT -p tcp --dport 27017 -j ACCEPT`

CentOS重启防火墙`firewall-cmd --reload`

如果是云服务器，在服务器的安全组配置中，将端口添加进规则即可，如下图：

![1](/img/20191014/1.png '1')
![2](/img/20191014/2.png '2')
![3](/img/20191014/3.png '3')
![4](/img/20191014/4.png '4')
![5](/img/20191014/5.png '5')

---

## 外部连接设置

连接地址为`mongodb://<username>:<password>@host:port`，如果授权验证`auth`设置的false，则地址为`mongodb://host:port`。配置mongodb地址时，如果不设置连接方式默认是`MONGODB-CR`，如果安装的时候设置的是`SCRAM-SHA-1`，连接地址为：`mongodb://<username>:<password>@host:port/<dbname>?authSource=admin&authMechanism=SCRAM-SHA-1`

推荐一个mongo可视化工具：[adminMongo](https://github.com/mrvautin/adminMongo)

---
## 参考文献


[*0. Mongodb后台daemon方式启动*](https://blog.csdn.net/roler_/article/details/38820247)

[*1. MongoDB 3.4.2 添加用户、设置权限*](https://blog.csdn.net/zZ_life/article/details/78664794)

[*2. mongoDB的远程连接配置*](https://blog.csdn.net/AggressionStorm/article/details/79956231)

[*3. Centos防火墙设置与端口开放的方法*](https://blog.csdn.net/u011846257/article/details/54707864)

[*4. adminMongo*](https://github.com/mrvautin/adminMongo)

[*5. mongodb连接认证失败*](https://blog.csdn.net/qq_29143909/article/details/81909991)

---


