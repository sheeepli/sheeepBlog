---
title: vue 和 react 的差异
date: 2019-12-09 15:13:48
tags: [vue, react]
categories: "前端"
---

就学者而言，总会比较好坏，慢慢总结 vue 和 react 的差异。

<!-- more -->

## 相同

1. 组件只能有最外层元素
2. 两者在渲染列表时都需要指定 key，虽然不是强制要求，但是最好有，因为当我们更新这个列表是，Vue 和 react 需要知道哪些项发生了改变。

## 不同

* 创建组件方式不同

  vue 创建组件可以直接创建一个 `.vue` 文件，在里面写代码

  ```html
  // child.vue
  <template>
    <div>

    </div>
  </template>

  <script>
    export deafult{
      name: 'child'
    }
  </script>
  ```

  创建 react 组件时，可以直接创建一个 js 文件，并 return（生成） 一个 React"元素"。
  
  react 有两种方式可以创建组件，一种是纯函数，一种是构造函数创建组件。

  ```js
  // child.js
  // 方式一
  function Child(props) {
    return (
      <div></div>
    )
  }
  // 方式二
  class Child extends React.component{
    render() {
      return (
        <div></div>
      )
    }
  }
  export default Child
  ```

* react 使用的语法时 JSX 语法，类似于 XML。vue 更类似于 html。

* vue 是双向数据绑定，而 react 是单向数据绑定。

* 初始化数据的方式

  ```js
  // vue 初始化数据的方式
  new Vue({
    data() {
      return {
        /* some data */
      }
    }
  })

  // react 初始化数据的方式
  class Children extends React.component{
    constractor(props) {
      super(props)
      state = {
        /* some data */
      }
    }
    render(
      return (<div>{this.state.XXX}</div>)
    )
  }
  ```

* 传递数据的方式

  * 父 -> 子

    两者的父传子方式差不多，都是通过 props 传递。不同的是 vue 需要在子组件中定义所传的 props，而 react 允许通过 this.props 直接使用使用。

    ```js
    new Vue({
      data() {
        return {

        }
      },
      props: [] // 允许使用对象，当使用对象时，可以指定传入 props 格式
    })

    class Children extends React.component{
      render (
        return (
          <div>{this.props.XXX}</div>
        )
      )
    }
    ```

  * 子 -> 父

    在 vue 中，如果需要将数据传给父级组件，需要使用 $emit 触发一个自定义事件，并传递参数。

    在 react 中，允许直接通过 this.props 去触发父组件的事件，即父组件将事件以 props 的方式传递给子组件，让子组件去触发他。

  * 兄弟 & 其他

    vue 中的兄弟传值与子传父相同，都需要借助额外函数帮助。

    在 react 兄弟之间，共享状态各自状态时，需要用到状态提升，即将两个子组件的各自数据保存在父组件，之后所要做的事情与子传父相同

  其实，在 vue 和 react 中都有其各自的状态管理器，分别时 vux 和 redux。无论是在父传子还是子传父或者兄弟其他等等，都是可以使用全局状态的。

  **<p style="color: red">关于这个我还需要加强</p>**

* 生命周期有所不同，这里就不详细说明了，其实也差不多的。

