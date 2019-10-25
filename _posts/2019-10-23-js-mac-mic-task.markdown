---
layout:     post
title:      "JS 事件循环--宏任务与微任务"
subtitle:   "Event Loop"
date:       2019-10-23 13:54:57
author:     "Z1hgq"
header-img: "img/post-bg-js-version.jpg"
catalog: true
tags:
    - JavaScript
---

> “”

---
## Event Loop

JavaScript是一门单线程的脚本语言，在一行代码执行的过程中，必然不会存在同时执行的另一行代码，就像使用`alert()`以后进行疯狂`console.log`，如果没有关闭弹框，控制台是不会显示出一条log信息的。

js的主线程只负责维护任务队列，并通过事件循环机制（Event Loop）按照顺序将需要执行的任务放进栈中。

![](/img/20191025/2.png)

- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数
- 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
- 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
- 上述过程会不断重复，也就是常说的Event Loop(事件循环)。

## 宏任务与微任务

在Js中，有两类任务队列：宏任务队列（macro tasks）和微任务队列（micro tasks）。宏任务队列可以有多个，微任务队列只有一个。

>宏任务：script（全局任务）, setTimeout, setInterval, setImmediate, I/O, UI rendering.
>
>微任务：process.nextTick, Promise, Object.observer, MutationObserver.

![](/img/20191025/3.png)

#### 浏览器Event Loop

>0. 取一个宏任务来执行。执行完毕后，下一步。
>1. 取一个微任务来执行，执行完毕后，再取一个微任务来执行。直到微任务队列为空，执行下一步。
>2. 更新UI渲染

每执行下一次宏任务之前重新渲染。

#### NodeJs Event Loop

>1. 初始化 Event Loop
>2. 执行您的主代码。这里同样，遇到异步处理，就会分配给对应的队列。直到主代码执行完毕。
>3. 执行主代码中出现的所有微任务：先执行完所有nextTick()，然后在执行其它所有微任务。
>4. 开始 Event Loop

NodeJs 的 Event Loop 分为六个阶段：timers、pending callbacks、idle\prepare、poll、check、close callbacks。

>    ┌───────────────────────────┐
>
> ┌─│-----------timers----------│
>
> │  └─────────────┬─────────────┘
>
> │  ┌─────────────┴─────────────┐
>
> │  │---- pending callbacks-----│
>
> │  └─────────────┬─────────────┘
>
> │  ┌─────────────┴─────────────┐
>
> │  │-------idle, prepare-------│
>
> │  └─────────────┬─────────────┘      ┌───────────────┐
>
> │  ┌─────────────┴─────────────┐      │   incoming:   │
>
> │  │-----------poll------------│<─────┤  connections, │
>
> │  └─────────────┬─────────────┘      │   data, etc.  │
>
> │  ┌─────────────┴─────────────┐      └───────────────┘
>
> │  │-----------check-----------│
>
> │  └─────────────┬─────────────┘
>
> │  ┌─────────────┴─────────────┐
>
> └──┤------close callbacks------│
>
>    └───────────────────────────┘

- timers: 这个阶段执行setTimeout()和setInterval()设定的回调。
- pending callbacks: 上一轮循环中有少数的 I/O callback 会被延迟到这一轮的这一阶段执行。
- idle, prepare: 仅内部使用。
- poll: 执行 I/O callback，在适当的条件下会阻塞在这个阶段
- check: 执行setImmediate()设定的回调。
- close callbacks: 执行比如socket.on('close', ...)的回调。

## 示例分析

```js
setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})

```

在上面的代码中，node环境下的输出顺序为`7\6\8\2\4\9\11\3\10\5\12`,在浏览器中的输出顺序为`7\8\2\4\5\9\11\12`,在浏览器中去除了`Process.nextTick()`这个方法。

## 总结

![](/img/20191025/1.png)

## 参考文献

[*0. JavaScript 异步、栈、事件循环、任务队列*](https://segmentfault.com/a/1190000011198232)
[*1. JS事件循环机制（event loop）之宏任务/微任务*](https://juejin.im/post/5b498d245188251b193d4059)



