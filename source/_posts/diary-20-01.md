---
title: diary-20-01
date: 2020-01-02 19:32:57
categories: "周记"
---

描述一下这周的发现以及找个有点意义的句子写在这里

<!-- more -->

## 关于解构赋值新发现

```js
const obj = {
  a: 1,
  b: 2,
  c: 3,
  d: 4,
}
const {a, ...e} = obj;
a; // -> 1
e; // -> {b: 2, c: 3, d: 4}
```

与数组的解构赋值相同耶

```js
const arr = [1, 2, 3, 4, 4];
const [a, ...b] = arr;
a; // -> 1
b; // -> [2, 3, 4, 4]
```

并不会去重，去重是 Set() 的功能，不要弄混了。

## 返回整组元素

React 组件也能够返回存储在数组中的一组元素。这也就意味着许多 ES6 写法都可以直接组成由 JSX 组成的数组并返回直接渲染页面。

以 array.map 为例：

```js
const arr = [1, 2, 3, 4, 5];
const arrList = arr.map(item => (<li key={item}>{item}</li>))
render(){
  return (
    <ul>arrList</ul>
  )
}
```

能够渲染出 1 到 5 的有序列表。

## <></>

在项目中曾经看到 `<>{/* */}</>` 这样一个最外层标签，其实它是 `<React.Fragment></React.Fragment>`的简写。这个标签的出现是因为组件最外层只能由一个元素，那么就会有这种代码：

```js
function Fun(props) {
  return (
    <div>
      <td>1</td>
      <td>2</td>
    </div>
  )
}
render() {
  return (
    <tr>
      <Fun />
    </tr>
  )
}

/*
* 输出结果
* <tr>
*   <div>
*     <td>1</td>
*     <td>2</td>
*   </div>
* </tr>
*/
```

这样的代码在浏览器渲染中是没有效果的。故而 React.Fragment 出现了。

类似于 vue 的 template 标签，包裹在组件的最外层，且并不会添加任何元素。

`<></>` 简写与 React.Fragment 功能相同，但是不支持 key 或[属性](https://zh-hans.reactjs.org/docs/fragments.html#keyed-fragments)（暂时还没有这个东西）。
