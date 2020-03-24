---
title: 实现一个简单的发布订阅模式
date: 2020-03-24 16:24:05
tags: "基础"
categories: "编程"
---

关于未来，我们能做的就是活在当下。

关于发布和订阅，我一直都没有搞懂，有人说 websocket 属于发布订阅模式，还有 vue 的 bus 也属于发布订阅模式。

解释：定义对象间的一种`一对多`的关系，当一个对象的状态发生改变的时候，所有依赖于它的对象都会得到通知。

顺序：先订阅后发布

以售楼部为例子，客户留下手机号码的过程叫做订阅，之后售楼部发出消息叫做发布。我们只有先订阅了，才能够得到发布。

<!-- more -->

接下来就是用代码来实现它吧。

首先，我们需要一个对象，用来存储订阅者、添加订阅方法和发布方法。

```js
let pubsub = {
  subscribers: [],
  subscribe(fn) {
    // 订阅
    this.subscribers.push(fn);
  },
  publish(str) {
    // 发布
    this.subscribers.forEach(fn => fn(str));
  }
};
pubsub.subscribe(function(str) {
  console.log(str);
});
pubsub.subscribe(function(str) {
  console.log(str);
});
pubsub.publish(123);
pubsub.publish(456);

// -> 123
// -> 123
// -> 456
// -> 456
```

运行如上代码我们会发现无法点对点发布消息，都是点对全部发布消息，这不是我们所想看到的对吧。

所以我们需要给每个订阅者一个唯一标识。

```js
let pubsub = {
  subscribers: {}, // 这里我们使用对象来存储
  subscribe(key, fn) {
    // 订阅
    if (!this.subscribers[key]) this.subscribers[key] = []; // 当唯一标识不存在的时候，创建唯一标识
    this.subscribers[key].push(fn);
  },
  publish() {
    // 比较重点的就是这个发布环节了
    // 发布
    const key = Array.prototype.shift.call(arguments);
    const fns = this.subscribers[key];
    if (!fns || fns.length === 0) return;
    fns.forEach(fn => fn.apply(this, arguments));
  }
};
pubsub.subscribe("sub01", function(str) {
  console.log("sub01: " + str);
});
pubsub.subscribe("sub02", function(str) {
  console.log("sub02: " + str);
});
pubsub.publish("sub01", 123);
pubsub.publish("sub01", 456);
pubsub.publish("sub02", 456);

// -> sub01: 123
// -> sub01: 456
// -> sub02: 456
```

这样我们就能做到点对点发布消息，点对多发布消息了。

不知道你有没有发现，我们有订阅，却没有取消订阅，就像我们订购牛奶一样，定了就不能退，这完全没有道理嘛！？

```js
let pubsub = {
  // ...
  remove(key, fn) {
    const fns = this.subscribers[key];
    if (!fns || fns.length === 0) return;
    for (let i = fns.length - 1; i >= 0; i--) {
      if (fns[i] === fn) fns.splice(i, 1);
    }
  }
};

pubsub.subscribe(
  "sub01",
  (fn1 = function(str) {
    console.log("sub01: " + str);
  })
);
pubsub.subscribe("sub02", function(str) {
  console.log("sub02: " + str);
});
pubsub.remove("sub01", fn1);
pubsub.publish("sub01", 123);
pubsub.publish("sub01", 456);
pubsub.publish("sub02", 456);

// -> sub02: 456
```

这里有一个细节，就是关于 `fn === fn` 输出结果为 false 的。这关系到那个叫做数据存储的问题，正常情况下，值类型（Number、String、Boolean、undefined、null）可以直接比较，因为他们的值是放在栈里且一一独立的，可以直接比较的。但是引用类型就比较头疼了，当我们在定义一个引用类型时，我们的变量是缓存在栈里，而值是缓存在堆里，然后从堆中返回一个索引给到栈中。所以我们比较引用类型时是比较栈中的索引，而不是堆中的值。而赋值则是将索引复制，而不是将值复制。

这样大概就写完了。放一个关于 pubsub-js 的[链接](https://github.com/mroderick/PubSubJS)。

然后最后来封装一下，毕竟我们不可能每次都写那么多代码。

```js
let pubsub = {
  subscribers: {}, // 这里我们使用对象来存储
  subscribe(key, fn) {
    // 订阅
    if (!this.subscribers[key]) this.subscribers[key] = []; // 当唯一标识不存在的时候，创建唯一标识
    this.subscribers[key].push(fn);
  },
  publish() {
    const key = Array.prototype.shift.call(arguments);
    const fns = this.subscribers[key];
    if (!fns || fns.length === 0) return;
    // 发布
    fns.forEach(fn => fn.apply(this, arguments));
  },
  remove(key, fn) {
    const fns = this.subscribers[key];
    if (!fns || fns.length === 0) return;
    for (let i = fns.length - 1; i >= 0; i--) {
      if (fns[i] === fn) fns.splice(i, 1);
    }
  }
};
function initPubsub(obj) {
  for(let key in pubsub) {
    obj[key] = pubsub[key]
  }
}

exports.initPubsub = initPubsub
```

这样只需要引入这个文件就可以啦。