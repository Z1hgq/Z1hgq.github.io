---
layout:     post
title:      "BoyerMoore-JavaScript"
subtitle:   "字符串匹配算法"
date:       2019-09-29 23:04:00
author:     "Z1hgq"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 算法
---

> “ 在人类历史的长河中，真理因为像黄金一样重，总是沉于河底而很难被人发现，相反地，那些牛粪一样轻的谬误倒 漂浮在上面到处泛滥。 —— 培根 ” 

## 算法原理
阮一峰博客 [字符串匹配的Boyer-Moore算法](http://www.ruanyifeng.com/blog/2013/05/boyer-moore_string_search_algorithm.html)

## 算法实现
```javascript
function Bm(parent, child) {
    let baseP = 0;

    while (true) {
        let badC = '';
        let goodS = '';
        let flag = 0;
        for (let i = child.length - 1; i >= 0; i--) {
            if (child[i] == parent[baseP + i]) {
                flag++;
                if (flag == child.length) {
                    return baseP;
                }
            } else {
                badC = parent[baseP + i];
                goodS = child.substring(i + 1);
                //选择坏字符规则和好后缀规则计算出的较长的后移位数
                let b = bad(badC, child, i);
                let g = good(goodS, child)
                if (b > g) {
                    baseP += b;
                } else {
                    baseP += g;
                }
                break;
            }
        }
        if (baseP >= parent.length - child.length) return -1;
    }
}

function bad(badC, child, i) {
    return i - child.indexOf(badC);
}

function good(goodS, child) {
    if (goodS == "") return 0;
    //最长的好后缀至在字串中出现了一次
    if (child.lastIndexOf(goodS) == child.indexOf(goodS)) {
        let tmp = "";
        for (let i = 1; i < goodS.length; i--) {
            tmp = goodS.slice(i);
            if (child.indexOf(tmp) == 0) {
                let str = child.slice(0, child.length - goodS.length);
                let newChild = str + child;
                return newChild.length - goodS.length;
            }
        }
        return child.length;
    } else
    //最长的好后缀至在字串中出现了多次
    {
        let str = child.slice(0, child.length - goodS.length); //去除字串末尾的最长好后缀
        let lastIndex = str.lastIndexOf(goodS) + goodS.length - 1; //最长好后缀的最后一个字符上一次出现的位置
        return goodS.length - lastIndex;
    }
}
```