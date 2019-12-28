---
layout:     post
title:      "Mac OS git操作加速设置"
subtitle:   ""
date:       2019-12-28 17:49:33
author:     "Z1hgq"
header-img: "img/post-bg-rwd.jpg"
catalog: false
tags:
    - git
---

> “修养的花儿在寂静中开过去了，成功的果子便要在光明里结实 ——冰心”

---
### vpn搭建

推荐[ssrcloud](https://ssrclouddingyuelianjie029319ajdnsjdf.com/ "ssrcloud")，目前比较稳定，且性价比高

### 查看代理的端口

点击小飞机,http代理设置,如图
![port](/img/20191228/port.png)

### git代理设置

```shell
git config --global https.proxy http://127.0.0.1:1087
git config --global https.proxy https://127.0.0.1:1087
```

亲测速度clone可达1M/s