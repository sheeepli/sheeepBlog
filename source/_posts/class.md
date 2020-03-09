---
title: 类
date: 2020-01-14 18:44:56
tags: 基础
categories: 编程
---

原本是想写 react static 这方面的内容的，但是又想到 static 实际上是 ES6 或者说是类的一个属性，所以改成写类比较合适一点。

<!-- more -->

## 类与函数

众所周知，js 是没有类这个东西的。这是一个无法改变的事实，即使 ES6 推出了 class 语法糖让我们看起来好像是有这么一回事的，可实际上那只是对类的一种模仿。

> 类实际上是个“特殊的函数”，就像你能够定义的函数表达式和函数声明一样，类语法有两个组成部分：类表达式和类声明。

这句官方的解释就是告诉我们类实际上是属于函数整个大类型的一部分。

## 实现类

在 ES5 之前，我们可以使用构造函数去模拟类，当然现在也可以。

在函数内部使用 this 关键字指代实例对象。

可以使用 new 关键字来生成一个实例。

类的属性和方法可以定义在构造函数的 prototype 对象上

```js
function Cat() {
  this.name = 'little cat'
}
var cat = new Cat()
cat.name; // -> little cat
Cat.prototype.sound = function() {
  console.log('miao')
}
cat.sound(); // -> miao
```

这大概就是基本的实现了，但是 Cat 类中的 name、sound 都是公有属性，会被外部传进来的参数修改，那怎么声明私有属性呢？

还是用 Cat 的例子

```js
function Cat(name) {
  this.name = name;
  color = 'yellow';
}
var cat = new Cat('hanabi')
cat.name; // -> hanabi
cat.color; // -> undefined
```

私有属性有什么用呢？在上面的例子中，似乎没有用处，但是在实际其他的情况下，用处就显现出来了。

```js
function Calculate(num) {
  this.num = num
  baseNum = 64
  this.add = function () {
    return num + baseNum
  }
}
const cal = new Calculate(64)
const result = cal.add()
result; // 128
```

没错，就像函数那样，但是又跟函数不一样。我们这里只有一个返回加法而已，还有减乘除...。写成函数的话就需要把 baseNum 定义多几次。

其实模拟类还有好多的方法，但是我的目的不是说这么多基础的东西（其实就是自己不懂），我们要尽快地来看一下 ES6 给我们的语法糖的使用。

## ES6 class

这是现在着重要学的，也是实际要用的（react项目）。

### 声明

开头已经说过定义类有两种方法，类声明和类表达式。

```js
// 类声明
class Cat {
  /* some code */
}

// 类表达式
// 1. 匿名类
const Cat = class {
  /* come code*/
}

// 2. 具名类
const Cat = class Cat {
  /* come code*/
}
```

类与函数还有一个不同点：函数会提升，而类不会。在这里就必须注意类定义的位置了。

```js
const cat = new Cat(); // -> ReferenceError: Cannot access 'Cat' before initialization

class Cat {
  /* some code */
}
```

### construct

在写 react 类组件时，经常性的就会用到这个方法，只知道能够在这个函数里面初始化数据，也没有去深究这个到底是什么。

官方给出的解释：

> constructor方法是一个特殊的方法，这种方法用于创建和初始化一个由class创建的对象。

官方也没有详细介绍这个方法，但是有提到一个类只能由一个 construct 方法。如果包含多个则会报错（SyntaxError: A class may only have one constructor）。嗯，这个报错也挺详细。

### 原型方法

### 子类

### 超类

### 静态方法

### 实例属性

### 字段声明

## 注意

* 类声明和类表达式的主体都执行在严格模式下。

## 参考

[Javascript定义类（class）的三种方法](http://www.ruanyifeng.com/blog/2012/07/three_ways_to_define_a_javascript_class.html)
[class](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
