---
title: 高阶函数（HOF） -> 高阶组件（HOC）
date: 2020-03-29 19:04:07
tags: [js, 基础, react]
categories: 编程
---

有时候人们很喜欢造一些名字很吓人的名词，让人一听这个名词就觉得自己不可能学会，从而让人望而却步。但是其实这些名词背后所代表的东西其实很简单。—— 来自 React.js 小书

第一次遇到高阶函数是曾溪叫我们写 PHP 的时候教的，当初好像搞懂了，又好像没懂。

之后遇到 react 高阶组件就蒙了，才发现我根本没懂。

以此告诫自己，要多记笔记，多学习。

<!-- more -->

## 高阶函数

高阶函数只要满足参数或返回值为函数就可以成为高阶函数。

来自 wiki 的解释

```text
在数学和计算机科学中，高阶函数是至少满足下列一个条件的函数：

* 接受一个或多个函数作为输入
* 输出一个函数
```

在很多函数式编程语言中能找到的 map 函数是高阶函数的一个例子。它接受一个函数 f 作为参数，并返回接受一个列表并应用 f 到它的每个元素的一个函数。

当然，许多 ES6 新添加的数组方法都是高阶函数。

```js
const arr = [1, 2, 3, 4, 5];
function map (item, index, array) {
  return item * 2
}
const newArr = arr.map(map)
```

上面这个就是一个简单的高阶函数了。可这是 js 实现的，那我们自己怎么实现一个高阶函数呢？

```js
function hof(options) {
  return function(fn) {
    return fn(options)
  }
}

let options = {
  a: 1,
  b: 2,
  c: 3
}

function fn(options) {
  let newObj = {}
  for (let key in options) {
    newObj[key] = options[key]
  }
  return newObj
}

let obj = hof(options)(fn) // -> {a: 1, b: 2, c: 3}
```

这样我们就实现了一个非常简单的高阶函数 `hof`。虽然这个功能完全用不到高阶函数。

顺带一提，我们平常说的函数柯里化实际上就是一种高阶函数的实现。

## 高阶组件

> 具体而言，高阶组件是参数为组件，返回值为新组件的函数。

react 官网如是说。

我们都喜欢模块化，可是模块化之后的代码还是有很多相似的地方，它们仅仅是因为配置不同，但是我们又不得不将他单独作为一个模块。

高阶组件很好的解决了这个问题，我们只需要将配置出来，然后剩下的都是同样的内容了。

像 `myHoc(options)(WrappedComponent)` 我们可以仅仅传入不同的 options 以生成不同的组件。

**对不起，不想写了，就到这里吧，我要把我基础的补回来先，很快的**

## 参考

[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)
[助你完全理解React高阶组件（Higher-Order Components）](https://github.com/brickspert/blog/issues/2)
[React高阶组件（译）](https://imweb.io/topic/5907038a2739bbed32f60dad)