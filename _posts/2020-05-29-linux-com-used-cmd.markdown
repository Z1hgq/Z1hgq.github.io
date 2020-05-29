---
layout:     post
title:      "linux常用命令"
subtitle:   ""
date:       2020-05-29 16:32:33
author:     "Z1hgq"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - linux
    - shell
    - centos
---

> “”

---

#### 磁盘逻辑卷拓展

```shell
# 查看磁盘使用情况
df -h

# 减少逻辑卷/dev/mapper/centos-home的容量10G
lvreduce -L -10G /dev/mapper/centos-home

# 拓展逻辑卷/dev/mapper/centos-root的容量10G
lvextend -L +10G /dev/mapper/centos-root

# 刷新，重新加载逻辑卷
resize2fs /dev/mapper/centos-root

# 上述刷新无效或者报错的话，尝试使用一下命令，xfs_growfs不支持减小
xfs_growfs /dev/mapper/centos-root
```

#### 查看端口使用情况
```shell
netstat -nlp| grep <端口号>
```

#### 查看服务运行情况

```
ps -ef|grep <服务名称>
```

