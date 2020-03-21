---
title: Promise 基础
date: 2020-03-20 15:20:59
tags: 基础
categories: 编程
---

要手写 Promise 我们必须先要了解 Promise 的作用以及设计思想。

Promise 对象是 Javascript 的异步操作解决方式，为异步操作提供统一接口。它的设计思想是，所有异步任务都返回要给 Promise 实例。

<!-- more -->

在 Promise 出来之前，我们通常是使用回调函数来处理异步操作。

```js
function cb(err, data) {
  if (err) throw err
  // some code
}
http.get('/', cb(err, data))
```

很熟悉对吧，接下来我们看一下 Promise 怎么写（其实我也不是很懂这里）

```js
function emmm(resolve) {
  console.log(1)
  resolve()
}
const promise = new Promise(emmm)
promise.then((res) => {
  console.log(res)
}).then(() => {
  console.log(2)
}).catch((err) => {
  console.log(err)
})
```

## 状态

Promise 对象通过自身的状态来控制异步操作。Promise 实例具有三种状态：

* 异步操作未完成（pending）
* 异步操作成功（fulfilled）
* 异步操作失败（rejected）

但是这三种状态只有两种变化途径

* 从“未完成”到“成功”
* 从“未完成”到“失败”

一旦状态变化，就会有新的状态变化。因此，Promise 的最终结果只有两种

* 异步操作成功，Promise 实例返回一个值（value），状态变为 fulfilled
* 异步操作失败，Promise 实例抛出一个错误（error），状态变为 rejected

## 构造函数

Javascript 提供原生的 Promise 构造函数，用来生成 Promise 实例。

```js
const promise = new Promise(function (resolve,reject) {
  if (true) {
    // 异步操作成功
    resolve()
  } else {
    // 异步操作失败
    reject()
  }
})
```

Promise 构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve 和 reject，**它们是两个函数，由 javascript 引擎提供，不用自己实现**。

resolve 函数的作用是，将 Promise 实例的状态从“未完成”变为“成功”（即从 pending 变为 fulfilled），在异步操作成功时调用，并将异步操作的结果作为参数传递出去。并交由传入 then 的回调函数执行。

reject 函数的作用是，将 Promise 实例的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时，并将异步操作报出的错误作为参数传递出去。并交由传入 then 的回调函数执行。

```js
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  })
}
timeout(1000).then(() => {
  console.log(1) // 一秒之后控制台输出 1
})
```

## Promise.prototype.then()

Promise 实例的 then 方法是用来添加回调函数的。

then 方法可以接受两个回调函数，第一个是异步操作成功时的回调函数，第二个时异步操作失败时的回调函数（可以省略）。

```js
const p1 = new Promise((resolve, reject) => {
  resolve('success')
})
p1.then(console.log, console.error) // 输出 success

const p2 = new Promise((resolve, reject) => {
  reject(new Error('error'))
})
p2.then(console.log, console.error) // 输出 error

const p3 = new Promise((resolve, reject) => {
  reject('error')
})
p3.then(console.log, console.error) // 输出 error

function step1(data) {
  console.log('step1')
  return data + ',step1'
}

function step2(data) {
  console.log('step2')
  throw new Error('err')
  // return data + ',step2'
}

function step3(data) {
  console.log('step3')
  return data + ',step3'
}
p1.then(step1).then(step2).then(step3).then(console.log, console.error)
// -> step1
// -> step2
// -> Error: err
```

then 的链式调用中，如果要让下一个 then 获取到上一个 then 的值，那么上一个 then 就必须要返回值。

最后一个 then 方法，回调函数是 console.log 和 console.error，用法上有一个重要的区别。console.lo 只显示 step3 的返回值，而 console.error 可以显示 p1、step1、step2、step3 之中任意一个发生的错误。Promise 对象的报错具有传递性。

Promise 的用法简单说就是一句话：使用 then 方法添加回调函数。但是不同的写法有细微的差别。

```js
f1().then(function() {
  return f2()
}).then(f3) // f3 回调函数的参数是 f2 函数运行的结果

f1().then(function() {
  f2()
}).then(f3) // f3 回调函数的参数是 undefined。then 没有进行返回

f1().then(f2).then(f3) // f3 回调函数的参数是 f2 函数运行的结果

f1().then(f2).then(f3) // f3 回调函数的参数是 f1() 返回的结果
```

## 例子

### 图片加载

```js
const preloadImage = function(path) {
  return new Promise((resolve, reject) => {
    const image = new Image()
    image.onload = resolve // 妙啊！当图片加载好之后会执行 resolve ，然后进入 then
    image.onerror = reject // 当如片加载失败时执行 reject，然后进入 then
    image.src = path
  })
}
preloadImage(`https://tse4-mm.cn.bing.net/th/id/OIP.1e3YVW946dgy5uJH764JXwHaFj?w=272&h=204&c=7&o=5&pid=1.7`)
  .then(function(e) {
      console.log(e)
      document.body.append(e.target)
  })
  .then(function() {
      console.log('加载成功')
  })
```

### 异步请求

不想写

## 微任务

微任务这一方面的内容关系到 js 的运行机制，这是另一个篇幅的内容了。但是在这里简单的了解一下。

Promise 的回调函数属于异步任务，会在同步任务之后运行。

```js
new Promise(function(resolve, reject) {
  console.log(1)
  resolve(2)
}).then(console.log)
console.log(3)
// -> 1
// -> 3
// -> 2
```

上面代码先输出 1，然后是 3， 最后是 2，我们在构造 Promise 实例时传入的函数是会立即执行的，相当于同步任务，之后的 console.log(3) 也属于同步任务，而 then 的回调函数属于异步任务，一定晚于同步任务执行。

但是，Promise 的回调函数不是正常的异步任务，而是微任务（microtask）。它们的区别在于：正常任务追加到下一轮时间循环，微任务追加到本轮时间循环。这意味着，微任务的执行时间一定早于正常任务。

```js
setTimeout(() => {
  console.log(1)
}, 0)
new Promise((resolve, reject) => {
  console.log(2)
  resolve(3)
}).then(console.log)
console.log(4)
// -> 2
// -> 4
// -> 3
// -> 1
```

## 参考

[Promise - 阮一峰](https://javascript.ruanyifeng.com/advanced/promise.html)
