---
layout:     post
title:      "Redux基本概念"
subtitle:   "Redux"
date:       2020-10-25 10:47:40
author:     "Z1hgq"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - redux
    - JavaScript
---

> 世上只有想不通的人，没有走不通的路

---

## redux 是什么

  react 是一个将DOM抽象出来的框架，但是它没有解决组件通信的问题。因此，在此基础上先出现了`flux`，在`flux`的基础上，2015年出现了`redux`。

  `redux`是一种前端数据管理机制，它认为Web应用是一个状态机，数据和视图是一一对应的，同时它将所有的数据都保存在一个对象里面。

  `redux`的**三大原则**:
  - 单一数据源
  - state是只读的
  - 使用纯函数对state执行修改
## 为什么要用redux

从业务层面来说，`redux`解决了以下问题：

> - 用户使用方式复杂
> - 不同用户身份有不同的使用方式
> - 多个用户之间可以协作
> - 与服务器之间有大量的交互，或者使用了WebSocket
> - View要从多个数据源获取数据

从组件层面来说，`redux`解决了以下问题

> - 某个组件的状态需要共享
> - 某个状态在任何地方都可以拿到
> - 一个组件需要改变全局状态
> - 一个组件需要改另一个组件的状态

## redux的执行流程

### 基本概念

#### store 

  `Store` 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 `Store`。

#### state 

  `Store`对象包含所有数据。如果想得到某个时点的数据，就要对 `Store` 生成快照。这种时点的数据集合，就叫做 `State`。
  Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。

#### action

  State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。
  Action 是一个对象。其中的type属性是必须的，表示 Action 的名称。其他属性可以自由设置，社区有一个[规范](https://github.com/redux-utilities/flux-standard-action)可以参考。

```js
const action = {
    type: 'ADD_TODO',
    payload: 'Learn Redux'
};
```
  上面代码中，Action 的名称是ADD_TODO，它携带的信息是字符串Learn Redux。

  可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。
#### dispatch
  dispatch是View触发action的唯一方法。
```js
dispatch({
    type: 'ADD_TODO',
    payload: 'Learn Redux'
});
```
  上面代码中 dispatch 接受一个 action 对象作为参数，将它发送出去。

#### reducers
  Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。

  Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。
  
```js
const reducer = function (state, action) {
    // ...
    return new_state;
};
```
  整个应用的初始状态，可以作为 State 的默认值。下面是一个实际的例子。
```js
const defaultState = 0;
const reducer = (state = defaultState, action) => {
    switch (action.type) {
        case 'ADD':
            return state + action.payload;
        default: 
            return state;
    }
};

const state = reducer(1, {
    type: 'ADD',
    payload: 2
});
```
  上面代码中，reducer函数收到名为ADD的 Action 以后，就返回一个新的 State，作为加法的计算结果。其他运算的逻辑（比如减法），也可以根据 Action 的不同来实现。

#### effects
  
  `effects`意为副作用，上面说到reducer必须是纯函数，那么effect对于state的修改就是带副作用的。我们常常将数据请求放在effect中，拿到数据后，再调用reducer修改state。

#### subscribe

  store 允许通过 subscribe 方法设置监听函数，一旦 state 发生变化就立即执行这个函数，例如在 react 中，redux store 的 state 发生更新之后自动执行react的render进行组件的从新渲染

#### 纯函数

Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。

纯函数是函数式编程的概念，必须遵守以下一些约束。

- 不得改写参数
- 不能调用系统 I/O 的API
- 不能调用Date.now()或者Math.random()等不纯的方法，因为每次会得到不一样的结果

由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象，请参考下面的写法。

```js
// State 是一个对象
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];
}
```
最好把 State 对象设成只读。你没法改变它，要得到新的 State，唯一办法就是生成一个新对象。这样的好处是，任何时候，与某个 View 对应的 State 总是一个不变的对象。



### 执行过程

![执行流程](/img/20201026/1.jpg)

## 如何实现一个redux