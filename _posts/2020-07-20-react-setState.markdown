---
layout:     post
title:      "React源码解析（一）setState"
subtitle:   "React v16.13.0"
date:       2020-07-20 23:55:43
author:     "Z1hgq"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - JavaScript
    - React
    - 源码
---

> “只有创造，才是真正的享受，只有拼搏，才是充实的生活”

---

#### 入口

react中ReactBaseClasses.js中声明的`Component`类的setState方法
```js
Component.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};
```
类`Component`的声明
```js
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}
```
`ReactNoopUpdateQueue`用于校验组件是否初始化完成，Component未传入updater就调用了setState表明此时组件未初始化完成，在开发环境下给出error。详情查看react源码`ReactNoopUpdateQueue.js`

#### 组件即将更新
![run-setState](/img/20200720/1.png)
react-dom在每一次处理ReactElement都会调用processChild方法。在processChild中用了一个队列`quene`来存储setState传进来的第一个参数，重写了一个updater来处理部分更新，其中的enqueueSetState方法每次在组件内调用setState时执行，在quene中添加一个state数据，enqueueReplaceState方法在调用replaceState时调用，将quene置成只有一个最新state元素的队列。<br>
每一次处理组件时传入原来组件的信息和updater返回一个新的组件。

#### state更新
以下为state更新的关键代码
![run-setState](/img/20200720/2.png)

1. 清空quene，用oldQueue来存储此次操作的quene数据
2. 仅仅执行了replaceState的话则直接将quene中的第一个元素赋值给新的state
3. 如果在执行了replaceState之后又执行了setState，则将quene第一个值直接赋值给nextState，否则nextState为组件更新之前的state
4. 遍历quene中的state数据，开始依次使用`Object.assign`合并state,如果是步骤2中的情况则从第二个元素开始
5. state合并完成之后，调用组件的render方法，重新渲染组件，最后processChild方法返回child

PS：
- setState的第一个参数可以是函数，返回值为想要更新的数据，传入三个参数<nextState, element.props, publicContext>
- setState每次开始合并对象时都会用一段新的内存来存放更新之后的state数据

#### callback执行
**callback是在setState批量更新之后批量执行的**
![run-setState](/img/20200720/3.png)

1. 每次调用setState往`_callbacks`中添加一个setState的回调函数
2. 如下图所示在render完成之后使用`_invokeCallbacks`函数批量执行callback

![run-setState](/img/20200720/4.gif)

举个栗子，以下代码点击button执行两个setState，但是两个setState的callback中拿到的都是最新的state

![run-setState](/img/20200720/6.png)
![run-setState](/img/20200720/5.gif)

#### setState异步

先说**结论**：

1. `setState` 只在**合成事件**和**钩子函数**中是“异步”的，在**原生事件**和 **setTimeout** 中都是同步的。
2. `setState`的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果。
3. `setState` 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和`setTimeout` 中不会批量更新，在“异步”中如果对同一个值进行多次 `setState` ， `setState` 的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时 `setState` 多个不同的值，在更新时会对其进行合并批量更新。

有下面一段代码，componentDidMount中有一个钩子函数直接调用和setTimeout调用，还有一个原生事件和一个合成事件
```js
class App extends React.Component {
  state = {
    count: 0
  };

  componentDidMount() {
    // 生命周期中调用
    this.setState({ count: this.state.count + 1 });
    console.log("lifecycle: " + this.state.count);
    setTimeout(() => {
      // setTimeout中调用
      this.setState({ count: this.state.count + 1 });
      console.log("setTimeout: " + this.state.count);
    }, 0);
    document.getElementById("div2").addEventListener("click", this.increment2);
  }

  increment = () => {
    // 合成事件中调用
    this.setState({ count: this.state.count + 1 });
    console.log("react event: " + this.state.count);
  };

  increment2 = () => {
    // 原生事件中调用
    this.setState({ count: this.state.count + 1 });
    console.log("dom event: " + this.state.count);
  };

  render() {
    return (
      <div className="App">
        <h2>couont: {this.state.count}</h2>
        <div id="div1" onClick={this.increment}>
          合成点击事件
        </div>
        <div id="div2">原生点击事件</div>
      </div>
    );
  }
}
```
直接运行结果如下，钩子函数中直接使用setState不会拿到最新的值，但是在setTimeout中可以
![run-setState](/img/20200720/7.png)

在原生事件和合成事件中，出发合成事件执行setState拿不到最新的值，但是在原生事件中可以
![run-setState](/img/20200720/8.gif)

react为了解决跨平台，兼容性问题，自己封装了一套事件机制，代理了原生的事件，像在jsx中常见的`onClick`、`onChange`这些都是合成事件。<br>
setState并不是真正意义上的异步操作，它只是模拟了异步的行为。React中会去维护一个标识，判断是直接更新还是先暂存state进队列。setTimeout以及原生事件都会直接去更新state，因此可以立即得到最新state。而合成事件和React生命周期函数中，是受React控制的，从而走的是类似异步的那一套。

#### 为什么setState要设计成异步
1. 保证内部的一致性：即使state是同步更新，props也不是。（只有在父组件重新渲染时才会更新props）
2. 将state的更新延缓到最后批量合并再去渲染对于应用的性能优化是有极大好处的，如果每次的状态改变都去重新渲染真实dom，那么它将带来巨大的性能消耗。

可以参考[RFClarification: why is `setState` asynchronous?](https://github.com/facebook/react/issues/11527#issuecomment-360199710)

#### fiber与setState

待续...


