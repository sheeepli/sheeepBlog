---
title: 比较两个数据是否相同
date: 2019-12-25 16:08:02
tags: [js, 基础]
categories: 编程
---

在我们的实际开发过程中，常常需要去比较两个数据是否相同，对于字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol这些基础类型来说，我们只需要用 `===` 比较一下就可以了，也就没有必要写这个东西，但是对于对象和数组呢？

<!-- more -->

### 初衷

项目开发过程中遇到需要比较两个对象，数组的内容，也有尝试过如转换成 base64，转换成字符串比较，结果都不理想，毕竟**对象是没有顺序的**。

我也有在网上看过一些别人写的，有比我写的好的，但也有用 toString() 之后比较的。

然后也想到了我前几天复习的（尾）递归，就动手写了一下。

主要用到的是 ES6 的 arr.every() 函数。

```js
function compare(before, after) {
  // object 和 array 不能直接比较
  // symbol string number boolean undefined 类型可以直接比较
  if (typeof before === "symbol" && typeof after === "symbol" ||
    typeof before === "string" && typeof after === "string" ||
    typeof before === "number" && typeof after === "number" ||
    typeof before === "boolean" && typeof after === "boolean" ||
    typeof before === "undefined" && typeof after === "undefined") {

    // NaN === NaN = false，且除了 number 类型之外的数据都是 NaN，所以要另外判断
    if (isNaN(before) === true &&
      isNaN(after) === true &&
      typeof before === "number" &&
      typeof after === "number") return true
    /*
    * 直接比较两个值，Symbol 类型也可以，但就算 symbol 描述相同，他们的比较结果依旧为 false
    * Symbol(123） === Symbol(123) // -> false
    * 这是由于 Symbol 类型的（唯一性）属性决定的。
    */
    if (before === after) return true

    return false
  }

  // typeof 不能判断出 array 类型，输出结果为 object。但 Array.isArrya() 可以。
  // 数组判断，先判断长短，再判断是否对应
  if (Array.isArray(before) === true && Array.isArray(before) === true) {
    // 两个数组长度不同
    if (before.length !== after.length) return false

    return before.every((item, index, arr) => compare(item, after[index]))
  }
  
  // object 比较
  if (typeof before === 'object' && typeof after === 'object') {

    // 众所周知，{} === {} 的结果是 false，而 typeof null 的结果是 object，所以这里的比较是针对两个 null 的比较，null === null 的结果为 true
    if (before === after) return true

    // !{} 结果为 false，!null 结果为 true，所以当两个对象其中一个为 null 时，可以返回 false。
    // 由于上面两个 null 的情况已经返回了，所以这里并不会比较两个 null 的情况
    if (!before || !after) return false

    // 当两个对象的 keys 长度不同时，没有比较的必要
    if (Object.keys(before).length !== Object.keys(after).length) return false

    return Object.keys(before).every((item, index, arr) => compare(before[item], after[item]))
  }
  
  // 当两者类型不同时，返回false
  return false
}
```

代码如上，开箱即用，我也写了一些简单数据来测试，结果还算满意。但是遇上大量数据时，会不会因为递归导致崩溃这就难说了`(●'◡'●)`。

### 差点忘了说

还有好多好多没有去比较，继承类、函数等等

如果有遇到不能用这个方法去比较的，我会回来检查更新的。

其实不知道函数到底要怎么比较，也不知道类要怎么比较，很多很多都还不了解，能力有限，到此为止叭。

### 再说一遍

ES6 真好用`(●'◡'●)`。

### 仙人指路

[Array.prototype.every](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
