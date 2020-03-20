---
title: 防抖（debounce）和节流（throttle）
date: 2020-03-19 23:16:48
tags: 基础
categories: 编程
---

我自问这辈子没做过坏事，为什么要被生活勒住喉咙。—— 网易云热评《父亲的散文诗》

关于防抖和节流这一部分功能，最难的地方也是最重要的地方就是闭包的使用。

<!-- more -->

## 防抖 debounce

防抖，就是指在触发事件之后 n 秒内只能执行一次，如果 n 秒内又触发了事件，则会重新计算函数执行时间。

想象这样一种情况，当你点击一个按钮时，你本来想点一次，但是鼠标接触过于优良连点了两次，就会造成两次数据提交。这种时候就需要用到我们的防抖了。

所以防抖一般常用在提交表单时。

防抖有两种，一种是非立即执行版，一种是立即执行版

### 非立即执行

非立即执行版的意思是触发事件后函数不会立即执行，而是在 n 秒后执行，如果在 n 秒内又触发了事件，则会重新计算函数执行时间。

```html
<body>
    <button id="btn">submit</button>
    <script>
        function debounce(fn, wait) {
            let timeout = null
            return function() {
                if (timeout !== null) {
                    // 作用：清除上一次的 timeout，不然每次点击都会延时触发函数
                    clearTimeout(timeout)
                }
                timeout = setTimeout(fn, wait)
            }
        }

        function submit() {
            console.log(2)
        }
        document.getElementById('btn').onclick = debounce(submit, 1000)
        // 注意这里是直接调用函数，并不是写 function(){ debounce(submit, 1000) }
    </script>
</body>
```

### 立即执行

立即执行版的意思是触发事件后函数会立即执行，然后 n 秒内不触发事件才能继续执行函数的效果。

```js
function debounce(fn, wait) {
  let timeout = null
  return function() {
    if (timeout !== null) {
      clearTimeout(timeout)
    }
    let callNow = !timeout
    timeout = setTimeout(function() {
      timeout = null
    }, wait)
    if (callNow) fn()
  }
}
```

在这里我们发现，无论是非立即执行还是立即执行，我们都需要去清除上一次的 setTimeout，如果不清除，到了约定时间它还是会执行。

我们还可以给 debounce 函数加多一个参数用来控制非立即执行还是立即执行，以方便我们的不同需求。

```js
function debounce(fn, wait, immediate) {
  let timeout = null
  return function() {
    if (timeout !== null) {
      clearTimeout(timeout)
    }
    if (immediate) {
      let callNow = !timeout
      timeout = setTimeout(function() {
          timeout = null
      }, wait)
      if (callNow) fn()
    } else {
      timeout = setTimeout(fn, wait)
    }
  }
}
```

## 节流 throttle

节流，就是指连续触发事件，但是 n 秒内只执行一次函数。节流会稀释函数的执行频率。

相较起来，节流写起来会比防抖容易些。

```js
function throttle(fn, wait) {
  let timeout = null
  return function() {
    if (!timeout) {
      timeout = setTimeout(() => {
        timeout = null
        fn()
      }, wait);
    }
  }
}
```

就酱，告辞。
