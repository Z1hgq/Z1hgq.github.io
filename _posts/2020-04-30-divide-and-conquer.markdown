---
layout:     post
title:      "常用算法--分治(Divide And Conquer)"
subtitle:   ""
date:       2020-04-30 09:57:56
author:     "Z1hgq"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - JavaScript
    - 算法
    - 分治
---

> “”

---
### 算法原理

分治，也就是分而治之，它的一般步骤如下：

1、对于规模为N的问题，当N足够小时可以直接解决，否则执行下面的步骤。<br>
2、将问题分解为M个规模较小的子问题，这些子问题相互独立，并且形式与原问题相同。<br>
3、递归解决子问题。<br>
4、将子问题的解合并，得到最终问题的解。<br>

![01](/img/20200430/1.png)

---
### 常用分治算法

#### 二分搜索

```js
// 非递归
function BinarySearch(arr, key) {
  let low = 0;
  let high = arr.length;
  while (low < high) {
    const mid = Math.floor((low + high) / 2);
    if (key > arr[mid]) {
      low = mid + 1;
    } else if (key < arr[mid]) {
      high = mid - 1;
    } else {
      return mid;
    }
  }
  return -1;
}
// 递归
function RecursiveBinarySearch(arr, low, high, key) {
  const mid = Math.floor((low + high) / 2) + low;
  if (arr[mid] == key) return mid;
  if (low >= high) {
    return -1;
  } else if (key > arr[mid]) {
    return RecursiveBinarySearch(arr, mid + 1, high, key);
  } else if (key < arr[mid]) {
    return RecursiveBinarySearch(arr, low, mid - 1, key);
  }
  return -1;
}
```


