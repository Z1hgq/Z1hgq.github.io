---
layout:     post
title:      "一段表情包爬虫demo"
subtitle:   "py-spider"
date:       2019-10-14 16:04:00
author:     "Z1hgq"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - 爬虫
    - Python
---

> “活在世界上，有时候人家不害你，就是对你最大的帮忙。”

---

## 源码
```python
#!/usr/bin/env python
# encoding=utf-8
import requests
import re
import random
from bs4 import BeautifulSoup
import os
import string

DOWNLOAD_URL  = 'https://pic.sogou.com/pic/emo/searchList.jsp?statref=home_form&spver=0&rcer=&keyword='


def download_page(url):
    """获取url地址页面内容"""
    headers = [
        'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36',
        "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36",
        "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_2) AppleWebKit/537.75.14 (KHTML, like Gecko) Version/7.0.3 Safari/537.75.14",
        "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; Win64; x64; Trident/6.0)",
    ]
    random_header = random.choice(headers)
    data = requests.get(url, headers={"User-Agent": random_header}, timeout=30).content
    return str(data,encoding="utf-8")

def getImgList(html):
    soup = BeautifulSoup(html, 'html.parser')
    img = re.compile(r'rsrc="(.+?)"')
    imgList = re.findall(img, html)
    return imgList

def mkdir(key):
	if not os.path.exists('img/'+key):
		os.mkdir('img/'+key)

def download_img(imgList, key):
    mkdir(key)
    for line in imgList:
        img_content = requests.get(line).content
        img_path = 'img/'+ key + '/' + ''.join(random.sample(string.ascii_letters + string.digits, 8)) + '.jfif'
        if not os.path.exists(img_path):
            with open(img_path, 'wb') as f:
                f.write(img_content)
                f.close()

def main():
    key = "giao"
    url = DOWNLOAD_URL + key
    if url:
        html = download_page(url)
        imgList = getImgList(html)
        download_img(imgList, key)

if __name__ == '__main__':
    main()
```

## 爬取结果
![res](/img/20191018/1.png "res")

---


