<!--
 * @Author: your name
 * @Date: 2020-01-06 20:55:27
 * @LastEditTime : 2020-01-07 08:49:19
 * @LastEditors  : Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \sheeep-37\source\_posts\reflow-and-repaints.md
 -->
---
title: 回流与重绘
date: 2020-01-06 19:05:08
tags: [js, 优化]
categories: "前端"
---

今天在写打印弹出框的时候的卡顿引起的想法。

但那是 antd 与插件之间的显示问题（虽然与我也有很大关系，毕竟插件是我用的）。

<!-- more -->

## 什么是回流（重排）

回流（reflow），或叫做重排，当 render tree 中的一部分或全部元素的尺寸，布局，隐藏等改变而重新构建的过程就叫做回流。

每个页面至少需要一次回流，即页面第一次加载的时候。

## 什么是重绘

当 render tree 中的一些元素需要更新属性，而这些属性只是影响元素的外观、风格，而**不影响布局**时，则称为重绘（repaints）。

**<p style="color: red;">重绘不一定会回流，回流必然导致重绘。</p>**

## 何时发生

当页面布局和几何属性改变时，就会发生回流。

1. 添加或删除可见的 DOM 元素；
2. 元素位置改变；
3. 元素尺寸改变，包括边距、填充、边框、宽度、高度；
4. 内容改变；
5. 页面渲染初始化；
6. 浏览器矿口尺寸改变——resize 事件。

当然上面的每种情况都会进行重绘。但是在不影响布局的情况下就只会发生重绘而不会发生回流。

```js
const box = document.getElementById('box').style;
box.padding = '2px'; // 回流 + 重绘
box.border = '1px solid #000'; // 回流 + 重绘
box.height = '20px'; // 回流 + 重绘
box.background = '#000'; // 重绘
box.color = 'blue'; // 重绘
```

**不可见的元素不影响重排和重绘。**

## 对于性能的影响

回流和重绘会不断发生，这是无可避免的。他们非常消耗资源，是导致网页性能低下的根本原因。

**提高网页性能，就是要降低回流和重绘的成本，尽量少触发重新渲染。**

现代浏览器做出了足够多的优化，会尽量把所有的变动集中在一起，拍成一个队列，然后一次性执行，尽量避免多次重新渲染。

```js
box.color = 'red';
div.marginTop = '10px';
```

尽管上面有两个样式变动，但是浏览器只会触发一次回流和重绘。

但是，在样式的写操作之后，如果有下面这些属性的读操作，都会引发浏览器的立即重新渲染。

* offsetTop / offsetLeft / offsetWidth / offsetHeight
* scrollTop / scrollLeft / scrollWidth / scrollHeight
* clientTop / clientLeft / clientWidth / clientHegiht
* getComputedStyle()
* getBoundingClientRect

一般规则：

* 样式表越简单，回流和重绘越快；
* 回流和重绘的 DOM 元素层级越高，成本就越高；
* table 元素的回流和重绘成本要高于 div 元素。

## 怎么避免回流重绘，优化（提高）性能

1. DOM 的多个读操作应该放在一起，不要在两个读操作之间插入一个写操作。反之亦然；
2. 如果某个样式是通过回流得到的，那么最好缓存结果。避免下次用到的时候又要回流；
3. 不要一条一条改变样式，而是通过 class，或者 csstext 属性，一次性改变样式；
4. 尽量使用离线 DOM，而不是真实的网面 DOM 来改变元素样式；
    * 操作 Document Fragment 对象，完成后再把这个对象加入 DOM。
    * 使用 CloneNode() 方法，在克隆的节点上进行操作，然后再用克隆的节点替换原始节点。
5. 先将元素设为 display: none（一次回流和重绘），然后对这个节点进行多次操作，最后在恢复显示（一次回流和重绘）。这样就可以使用两次重新渲染代替多次重新渲染；
6. position 属性为 absolute 或 fixed 的元素，重拍的开销会比较小，因为不用考虑对其他元素的影响；
7. 只在必要的时候才将元素的 display 属性设置为可见，因为不可见的元素不影响回流的重绘。visibility: hidden 的元素支队重绘有影响，不影响回流；
8. 使用虚拟 DOM 的脚本库，比如 React 等；
9. 使用 window.requestAnimationFrame()、window.requestldleCallback() 这两个方法调节重新渲染。

## 参考

* [网页性能管理详解](https://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)
* [网页性能篇——回流与重绘](https://www.codenong.com/js35073767a887/)
* [细谈页面回流与重绘](https://juejin.im/post/5c87bd375188257e3e47fdc5)
