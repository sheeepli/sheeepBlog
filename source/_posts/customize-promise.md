---
title: 自定义（手写） promise
date: 2020-03-26 22:54:54
tags: [js, 基础]
categories: 编程
---

我必须羞耻地承认，我可以谈话的对象数量越来越少了。 —— 维特根斯坦

前面我们写过一篇 [Promise 基础](/2020/03/20/promise/)，这让我们知道了怎么用 Promise。
但是仅仅知道怎么用就够了嘛？我们还需要知道怎么自行封装一个 Promise。以及如何应付面试(●'◡'●)。

<!-- more -->

## 面试

由于我的代码实现过程真的又臭又长，先把面试的写了。

### 回调函数分为 同步回调函数 和 异步回调函数

同步回调函数会立即执行，不会放入回调队列中，例如 [].forEach(回调函数)、promise 的 excutor 函数

异步回调函数不会立即执行，会放入到回调队列中，例如 setTimeout(回调函数, 0)、ajax 回调、promise 的成功或失败的回调

### 具体说说 Promise 是什么

- Promise 是 js 中进行异步编程的新的解决方案（旧的是纯回调）
- 语法上来说：Promise 是一个构造函数
- 功能上来说：promise 对象用来封装一个异步操作并可以获取结果

### promise 比 纯回调好在哪里（优点）

- promise 指定回调函数的方式更灵活

  纯回调需要再启动前指定回调函数
  promise：启动异步任务 -> 返回 promise 对象 -> 给 promise 对象绑定回调函数（甚至可以在异步操作完成后指定）

- 支持链式调用，可以解决回调地狱问题

  什么是回调地狱 回调函数嵌套调用，外部回调函数异步执行结果是嵌套函数的回调函数执行的条件
  回调地狱的缺点 不便于阅读，不便于异常处理
  解决方案 promise 链式调用
  终极解决方案 [async/await](/2020/03/20/promise/#async-amp-await)

### promise.then() 返回的新 promise 的结果状态由什么决定

    1. 简单表达：由 then() 指定的回调函数执行的结果决定
    2. 详细表达：
        1. 如果抛出异常，新 promise 变为 rejected，reason 为抛出的异常
        2. 如果返回的是非 promise 的任意值，新 promise 变为 resolved，value 为返回的值
        3. 如果返回的是另一个新的 promise，此 promise 的结果就会成为新 promise 的结果

### promise 异常传透

- 当使用 promise 的 then 链式调用时，可以在最后指定失败的调用
- 前面任何操作出了异常，都会传到最后失败的回调中处理

### 中断 promise 链

- 问题：当使用 promise 的 then 链式调用时，在中间中断，不再调用后面的回调函数
- 解决方法：在回调函数中返回一个 pending 状态的 promise 对象 `return new Promise(()=>{})` 只要没有执行 resolve 或者 reject 就会保持 pending

## 代码实现

首先，我们需要先定义一个大的框架。

```js
// 自定义Promise函数模块：IIFE
(function(window) {
  /**
   * Promise 构造函数
   * @param {function} excutor 传入一个接收两个函数的执行器函数（同步执行）
   */
  function Promise(excutor) {}

  /**
   * Promise 原型对象的 then()
   * 指定成功和失败的回调函数
   * 返回一个新的 Promise
   */
  Promise.prototype.then = function(onResolved, onRejected) {};

  /**
   * Promise 原型对象的 catch()
   * 执行失败的回调函数
   * 返回一个新的 Promise
   */
  Promise.prototype.catch = function(onRejected) {};

  /**
   * Promise 函数对象的 resolve()
   * 返回一个指定结果的成功的 promise（不一定）
   */
  Promise.resolve = function(value) {};

  /**
   * Promise 函数对象的 reject()
   * 返回一个指定结果为失败的 promise
   */
  Promise.reject = function(reason) {};

  /**
   * Promise 函数对象的 all()
   * 返回一个 promise，只有当所有的 promise 都成功时成功，否则失败
   */
  Promise.all = function(promises) {};

  /**
   * Promise 函数对象的 race()
   * 返回一个 promise，其结果由第一个完成的 promise 决定
   */
  Promise.race = function(promises) {};

  window.Promise = Promise;
})(window);
```

### 实现执行器

我们的执行器要做的事情有三件。

1. 修改状态
2. 保存结果
3. 保存回调

所以我们首先需要定义三个属性。

```js
/**
 * Promise 构造函数
 * @param {function} excutor 传入一个接收两个函数的执行器函数（同步执行）
 */
function Promise(excutor) {
  this.status = "pending"; // 给 promise 对象指定 status 属性，默认值为 pending
  this.data = undefined; // 给 promise 对象指定一个用于存储结果数据的属性
  this.callbacks = []; // 每个元素的结构：{ onResolved(){}, onRejected(){} }

  // 执行成功操作，将 status 变更为 pending
  function resolve(value) {
    // 由于我们状态只能从 pending 变更成 fulfilled 或者 rejected 所以我们需要一个判断
    if (this.status === "pending") return;
    this.status = "fulfilled";
    this.data = value;
    // 当回调列表中存在数据时，循环调用回调列表中的回调函数
    if (this.callbacks.length > 0) {
      // 如果我们直接调用回调函数，则会直接触发，相当于同步调用，这不符合逻辑，所以我们需要把它写成异步调用
      setTimeout(() => {
        this.callbacks.forEach(callbacksObj => {
          callbacksObj.onResolved();
        });
      }, 0);
    }
  }
  // 失败与上面成功的一样
  function reject(reason) {
    // 由于我们状态只能从 pending 变更成 fulfilled 或者 rejected 所以我们需要一个判断
    if (this.status === "pending") return;
    this.status = "rejected";
    this.data = reason;
    // 当回调列表中存在数据时，循环调用回调列表中的回调函数
    if (this.callbacks.length > 0) {
      // 如果我们直接调用回调函数，则会直接触发，相当于同步调用，这不符合逻辑，所以我们需要把它写成异步调用
      setTimeout(() => {
        this.callbacks.forEach(callbacksObj => {
          callbacksObj.onRejected(reason);
        });
      }, 0);
    }
  }

  // 这里使用 try catch 的原因是我们除了要自行抛出异常之外，我们还需要捕获程序语法异常等其他异常。
  try {
    excutor(resolve, reject);
  } catch (error) {
    reject(error);
  }
}
```

这时候我们就可以创建一个 promise 对象了

```js
const p = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    resolve("操作成功");
  }, 500);
});
```

发现什么没有？创建 promise 对象时是不是报错了？

这里涉及到一个 this 指向的问题，如果函数是直接执行，则 this 指向 window，所以这里我们要么使用箭头函数，要么绑定 this，要么存储 Promise 的 this。实现方法很多，随意发挥叭。在 Promise 中，之后都会存储 this。后面就不在这样声明了。

```js
function Promise(excutor) {
  const _this = this;
  _this.status = "pending";
  _this.data = undefined;
  _this.callbacks = [];

  function resolve(value) {
    if (_this.status !== "pending") return;
    _this.status = "fulfilled";
    _this.data = value;
    if (_this.callbacks.length > 0) {
      setTimeout(() => {
        _this.callbacks.forEach(callbacksObj => {
          callbacksObj.onResolved(value);
        });
      }, 0);
    }
  }
  function reject(reason) {
    if (_this.status !== "pending") return;
    _this.status = "rejected";
    _this.data = reason;
    if (_this.callbacks.length > 0) {
      setTimeout(() => {
        _this.callbacks.forEach(callbacksObj => {
          callbacksObj.onRejected(reason);
        });
      }, 0);
    }
  }

  try {
    excutor(resolve, reject);
  } catch (error) {
    reject(error);
  }
}
```

这样就可以给自己 new 一个对象啦，开不开心呀(●'◡'●)

### 实现 then()

可是光能创建一个 promise 对象是没有任何意义的，我们还需要能够得到执行结果才行呀。

then 方法就是用来获取 promise 对象执行结果的方式，也是整个 Promise 构造函数的核心。

先来个简单的，就只能够放入回调函数的就可以了。

```js
Promise.prototype.then = function(onResolved, onRejected) {
  const _this = this;
  _this.callbacks.push({ onResolved, onRejected });
};
```

我们只要在构造器中添加这段代码就可以实现最简单的回调了，试试看吧。

可是这样还是不够啊，我们这里实现的是获取结果比我们添加回调函数晚的例子，那如果我们需要立即执行并获取结果呢？

```js
const p = new Promise((resolve, reject) => {
  // 立即执行
  resolve("操作成功");
}).then(
  value => {
    console.log(value);
  },
  reason => {
    console.log(reason);
  }
);
```

执行不了吧，因为我们在执行 resolve 的时候，回调函数还没有被放入 callbacks 中去，所以无法顺利执行回调。

那我们要怎么改呢？

俗话说 `if else 大法好`。不过这里我们确实需要用到判断。

```js
Promise.prototype.then = function(onResolved, onRejected) {
  const _this = this;

  // 如果我们还没有执行 resolve 或者 reject 那么 promise 对象的状态就不会改变，依旧还是 pending
  if (_this.status === "pending") {
    _this.callbacks.push({ onResolved, onRejected });
  } else if (_this.status === "fulfilled") {
    // 当执行 resolve 之后，promise 对象的状态就会变成 fulfilled
    onResolved(_this.data);
  } else {
    // 当执行 reject 之后，promise 对象的状态就会变成 rejected
    onRejected(_this.data);
  }
};
```

按照 Promise/A+ 规范，我们需要返回一个新的 promise 对象。

```js
Promise.prototype.then = function(onResolved, onRejected) {
  const _this = this;
  return new Promise((resolve, reject) => {
    if (_this.status === "pending") {
      _this.callbacks.push({
        onResolved,
        onRejected
      });
    } else if (_this.status === "fulfilled") {
      const result = onResolved(_this.data);
      // 判断 onResolved 返回值是否是 Promise 类型
      if (result instanceof Promise) {
        // 我们要知道新 promise 的运行结果就必须要使用 then 来获取
        result.then(resolve, reject);
      } else {
        resolve(result);
      }
    } else {
      const result = onRejected(_this.data);
      if (result instanceof Promise) {
        result.then(resolve, reject);
      } else {
        resolve(result);
      }
    }
  });
};
```

写到这里，是不是觉得没有问题了呢，我带着疑惑测试了下面这两个 promise。

```js
const p1 = new Promise((resolve, reject) => {
  resolve("操作成功");
});
p1.then(
  value => {
    return 1;
  },
  reason => {
    console.log(reason);
  }
)
  .then(
    value => {
      console.log(value);
      return value;
    },
    reason => {
      console.log(reason);
      return reason + 1;
    }
  )
  .then(
    value => {
      console.log("success:", value);
    },
    reason => {
      console.log("error: ", reason);
    }
  );

const p2 = new Promise((resolve, reject) => {
  resolve("操作成功");
});
p2.then(
  value => {
    return new Promise((resolve, reject) => {
      reject(1234);
    });
  },
  reason => {
    console.log(reason);
  }
)
  .then(
    value => {
      console.log(value);
      return value;
    },
    reason => {
      console.log(reason);
      return reason + 1;
    }
  )
  .then(
    value => {
      console.log("success:", value);
    },
    reason => {
      console.log("error: ", reason);
    }
  );
```

然后发现没有问题，哇我的天，是不是成功了。

然后我就去测试了一个延时成功的。

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("操作成功");
  }, 1000);
});
p.then(
  value => {
    console.log(value);
    return new Promise((resolve, reject) => {
      reject(1234);
    });
  },
  reason => {
    console.log(reason);
  }
).then(
  value => {
    console.log("success:", value);
  },
  reason => {
    console.log("error: ", reason);
  }
);
```

延时一秒之后输入 `操作成功`，然后再无其他显示，通过不断的调试，发现问题出在了

```js
_this.callbacks.push({ onResolved, onRejected });
```

这一行，为什么呢，因为我们仅仅只是传入一个新的 promise 到 callbacks 中去，而我们还需要有结果啊，结果。

于是我们需要更改这一部分代码。

```js
Promise.prototype.then = function(onResolved, onRejected) {
  const _this = this;
  return new Promise((resolve, reject) => {
    if (_this.status === "pending") {
      _this.callbacks.push({
        onResolved() {
          try {
            const result = onResolved(_this.data);
            if (result instanceof Promise) {
              result.then(resolve, reject);
            } else {
              resolve(result);
            }
          } catch (error) {
            reject(error);
          }
        },
        onRejected() {
          try {
            const result = onRejected(_this.data);
            if (result instanceof Promise) {
              result.then(resolve, reject);
            } else {
              resolve(result);
            }
          } catch (error) {
            reject(error);
          }
        }
      });
    } else if (_this.status === "fulfilled") {
      /**
       * 1. 如果抛出异常，return 的 promise 就会失败，reaseon 就是 error
       * 2. 如果回调函数返回的是非 promise，return 的 promise 就会成功，value 就是返回值
       * 3. 如果回调函数返回的是 promise，return 的 promise 的结果就是这个 promise 的结果
       */
      setTimeout(() => {
        // 抛出异常
        try {
          const result = onResolved(_this.data);
          if (result instanceof Promise) {
            // 回调函数返回 promise
            result.then(resolve, reject);
          } else {
            // 回调函数返回非 promise
            resolve(result);
          }
        } catch (error) {
          reject(error);
        }
      }, 0);
    } else {
      setTimeout(() => {
        try {
          const result = onRejected(_this.data);
          if (result instanceof Promise) {
            result.then(resolve, reject);
          } else {
            resolve(result);
          }
        } catch (error) {
          reject(error);
        }
      }, 0);
    }
  });
};
```

然后再次通过上面三个测试，大成功。

### 实现 catch()

相较之下，catch() 的实现就要简单得很多了。

```js
/**
 * Promise 原型对象的 catch()
 * 执行失败的回调函数
 * 返回一个新的 Promise
 */
Promise.prototype.catch = function(onRejected) {
  return this.then(undefined, onRejected);
};
```

### 实现 resolve()

```js
/**
 * Promise 函数对象的 resolve()
 * 返回一个指定结果的成功的 promise（不一定）
 */
Promise.resolve = function(value) {
  return new Promise((resolve, reject) => {
    if (value instanceof Promise) {
      value.then(resolve, reject);
    } else {
      resolve(value);
    }
  });
};
```

### 实现 reject()

```js
/**
 * Promise 函数对象的 reject()
 * 返回一个指定结果为失败的 promise
 */
Promise.reject = function(reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
};
```

### 实现 all()

```js
/**
 * Promise 函数对象的 all()
 * 返回一个 promise，只有当所有的 promise 都成功时成功，否则失败
 */
Promise.all = function(promises) {
  const values = new Array(promises.length);
  let resolvedCount = 0;
  return new Promise((resolve, reject) => {
    promises.forEach((p, index) => {
      // 这里使用 Promise.resolve 的原因是因为传入的 p 可能不是 Promise。
      Promise.resolve(p).then(
        value => {
          resolvedCount++;
          values[index] = value;
          // 当全部返回成功时才返回成功
          if (resolvedCount === promises.length) {
            resolve(values);
          }
        },
        error => {
          reject(error);
        }
      );
    });
  });
};
```

### 实现 race()

```js
/**
 * Promise 函数对象的 race()
 * 返回一个 promise，其结果由第一个完成的 promise 决定
 */
Promise.race = function(promises) {
  return new Promise((resolve, reject) => {
    promises.forEach(p => {
      Promise.resolve(p).then(resolve, reject);
    });
  });
};
```

### 完整代码

该实现的功能上面都实现了，接下来就把它们整合起来。

```js
(function (window) {
  const PENDING = 'pending'
  const FULFILLED = 'fulfilled'
  const REJECTED = 'rejected'

  function Promise(excutor) {
    const _this = this
    _this.status = PENDING
    _this.data = undefined
    _this.callbacks = []

    function resolve(value) {
      if (_this.status !== PENDING) return
      _this.status = FULFILLED
      _this.data = value
      if (_this.callbacks.length > 0) {
        setTimeout(() => {
          _this.callbacks.forEach(callbacksObj => {
            callbacksObj.onResolved(value)
          })
        }, 0);
      }
    }

    function reject(reason) {
      if (_this.status !== PENDING) return
      _this.status = REJECTED
      _this.data = reason
      if (_this.callbacks.length > 0) {
        setTimeout(() => {
          _this.callbacks.forEach(callbacksObj => {
            callbacksObj.onRejected(reason)
          })
        }, 0);
      }
    }

    try {
      excutor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }
  Promise.prototype.then = function (onResolved, onRejected) {
    // 防止传入的 onResolved 不是函数
    onResolved = typeof onResolved === 'function' ? onResolved : value => value
    // 值得一提，这里的 onRejected 可以不传，所以我们必须要给个默认值
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason}
    const _this = this
    return new Promise((resolve, reject) => {
      function handle(callback) {
        try {
          const result = callback(_this.data);
          if (result instanceof Promise) {
            result.then(resolve, reject);
          } else {
            resolve(result);
          }
        } catch (error) {
          reject(error);
        }
      }
      if (_this.status === PENDING) {
        _this.callbacks.push({
          onResolved() {
            handle(onResolved)
          }, onRejected() {
            handle(onRejected)
          }
        })
      } else if (_this.status === FULFILLED) {
        setTimeout(() => {
          handle(onResolved)
        }, 0);
      } else {
        setTimeout(() => {
          handle(onRejected)
        }, 0);
      }
    })
  };
  Promise.prototype.catch = function (onRejected) {
    return this.then(undefined, onRejected)
  };
  Promise.resolve = function (value) {
    return new Promise((resolve, reject) => {
      if (value instanceof Promise) {
        value.then(resolve, reject)
      } else {
        resolve(value)
      }
    })
  };
  Promise.reject = function (reason) {
    return new Promise((resolve, reject) => {
      reject(reason)
    })
  };
  Promise.all = function (promises) {
    const values = new Array(promises.length)
    let resolvedCount = 0
    return new Promise((resolve, reject) => {
      promises.forEach((p, index) => {
        p.then(
          value => {
            resolvedCount++
            values[index] = value
            if (resolvedCount === promises.length) {
              resolve(values)
            }
          },
          reason => {
            reject(reason)
          }
        )
      })
    })
  };
  Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
      promises.forEach((p, index) => {
        p.then(resolve, reject)
      })
    })
  };
  window.Promise = Promise;
})(window);
```

### class 代码

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class Promise {
  constructor(excutor) {
      const _this = this
      _this.status = PENDING
      _this.data = undefined
      _this.callbacks = []
      function resolve(value) {
        if (_this.status !== PENDING) return
        _this.status = FULFILLED
        _this.data = value
        if (_this.callbacks.length > 0) {
          setTimeout(() => {
            _this.callbacks.forEach(callbacksObj => {
              callbacksObj.onResolved(value)
            })
          }, 0);
        }
      }
      function reject(reason) {
        if (_this.status !== PENDING) return
        _this.status = REJECTED
        _this.data = reason
        if (_this.callbacks.length > 0) {
          setTimeout(() => {
            _this.callbacks.forEach(callbacksObj => {
              callbacksObj.onRejected(reason)
            })
          }, 0);
        }
      }
    try {
      excutor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }


  then(onResolved, onRejected) {
    // 防止传入的 onResolved 不是函数
    onResolved = typeof onResolved === 'function' ? onResolved : value => value
    // 值得一提，这里的 onRejected 可以不传，所以我们必须要给个默认值
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason}
    const _this = this
    return new Promise((resolve, reject) => {
      function handle(callback) {
        try {
          const result = callback(_this.data);
          if (result instanceof Promise) {
            result.then(resolve, reject);
          } else {
            resolve(result);
          }
        } catch (error) {
          reject(error);
        }
      }
      if (_this.status === PENDING) {
        _this.callbacks.push({
          onResolved() {
            handle(onResolved)
          }, onRejected() {
            handle(onRejected)
          }
        })
      } else if (_this.status === FULFILLED) {
        setTimeout(() => {
          handle(onResolved)
        }, 0);
      } else {
        setTimeout(() => {
          handle(onRejected)
        }, 0);
      }
    })
  };
  catch(onRejected) {
    return this.then(undefined, onRejected)
  };
  static resolve = function (value) {
    return new Promise((resolve, reject) => {
      if (value instanceof Promise) {
        value.then(resolve, reject)
      } else {
        resolve(value)
      }
    })
  };
  static reject = function (reason) {
    return new Promise((resolve, reject) => {
      reject(reason)
    })
  };
  static all = function (promises) {
    const values = new Array(promises.length)
    let resolvedCount = 0
    return new Promise((resolve, reject) => {
      promises.forEach((p, index) => {
        p.then(
          value => {
            resolvedCount++
            values[index] = value
            if (resolvedCount === promises.length) {
              resolve(values)
            }
          },
          reason => {
            reject(reason)
          }
        )
      })
    })
  };
  static race = function (promises) {
    return new Promise((resolve, reject) => {
      promises.forEach((p, index) => {
        p.then(resolve, reject)
      })
    })
  };
  
}

exports.Promise = Promise
```

## 结束

啊~ 终于写完了，其实里面内容都是抄的，我能怎么办，我也在学呀！

又又失业了，已经快要一个月没有上班了，感觉整个废人一样，只能靠学点东西过日子，但是没得钱，好难过。

呐，未来的我，你在干什么呢？

## 参考

[Promise 教程](https://www.bilibili.com/video/BV1MJ41197Eu)
