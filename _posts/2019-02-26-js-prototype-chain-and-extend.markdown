---
layout:     post
title:      "js原型链及对象的继承"
subtitle:   "原型链图解及对象的几种继承方式"
date:       2019-02-26 13:54:57
author:     "Z1hgq"
header-img: "img/post-bg-js-version.jpg"
catalog: true
tags:
    - js
---

> “”

---

## 原型链

JavaScript对象的原型（prototype）：

- 每一个构造函数都拥有一个prototype属性，这个属性指向一个对象，也就是原型对象。当使用这个构造函数创建实例的时候，prototype属性指向的原型对象就成为实例的原型对象。
- 原型对象默认拥有一个constructor属性，指向指向它的那个构造函数（也就是说构造函数和原型对象是互相指向的关系）。
- 每个对象都拥有一个隐藏的属性`[[prototype]]`，指向它的原型对象，这个属性可以通过
`Object.getPrototypeOf(obj)` 或 `obj.__proto__` 来访问。
- 实际上，构造函数的prototype属性与它创建的实例对象的`[[prototype]]`属性指向的是同一个对象，即 对象`.__proto__ === 函数.prototype`。
- 原型对象就是用来存放实例中共有的那部分属性或方法的对象。
- 在JavaScript中，所有的对象都是由它的原型对象继承而来，反之，所有的对象都可以作为原型对象存在。
- 访问对象的属性时，JavaScript会首先在对象自身的属性内查找，若没有找到，则会跳转到该对象的原型对象中查找，直到找到Object对象上去。
- Object对象的原型对象指向的是NULL，即`Object.__proto__ == NULL`。

如图：
![prototype chain](/img/20191021/1.png "prototype chain")

## 继承

#### 0.构造函数实现继承

```js
function Parent1(name) {
    this.name = name;
    this.sayName = function() {
        window.alert(this.name);
    }
}
Parent1.prototype.say = function(){};
function Child1(name, age) {
    Parent1.call(this, name);//apply,将父构造函数this指向子构造函数的实例上，导致父构造函数的所有属性在子构造函数中也有
    this.age = age;
}
```

这种继承方式不能继承父类原型链上的属性方法，因此这种继承方式是不完整的，即`Child1`中没有`say`方法。

#### 1.原型链实现继承

```js
function Parent2(){
  this.name = 'Parent2';
}
function Child2(){
  this.type = 'Child2';
}
Child2.prototype = new Parent2();//重点

new Child2().__proto__ === new Parent2();//true
```

缺点:当一个实例对象去修改来自原型对象上的属性时，另一个实例对象这个属性也会跟着改变，因为它们都是引用的同一个原型对象的属性。

#### 2.组合方式(常用)

```js
function Parent3(){
  this.name = 'Parent3';
}
function Child3(){
  Parent3.call(this);
  this.type = 'Child3';
}
Child3.prototype = new Parent3();
```

缺点：父级构造函数执行了两次。

#### 3.组合继承的优化1

```js
function Parent4(){
  this.name = 'Parent4';
}
function Child4(){
  Parent4.call(this);
  this.type = 'Child4';
}
Child4.prototype = Parent4.prototype;
```

问题所在：`Child4实例对象.__proto__.constructor === Parent4` 而不是Child4，所以不能正确找到构造函数。

#### 4.组合继承的优化2(完美写法)

```js
function Parent5(){
  this.name = 'Parent5';
}
function Child5(){
  Parent5.call(this);
  this.type = 'Child5';
}
Child5.prototype = Object.create(Parent5.prototype);//隔离
Child5.prototype.constructor = Child5
```

## 参考文献

[*0. 三分钟看完JavaScript原型与原型链*](https://juejin.im/post/5a94c0de5188257a8929d837)