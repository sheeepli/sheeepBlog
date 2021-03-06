---
title: 学习 vue
date: 2020-04-15 11:31:27
tags: [前端, vue]
categories: 编程
---

也不是第一次学习 vue 了，也不是最后一次学习 vue。

<!-- more -->

## vuex

vuex 的 state 在刷新页面之后就回去清除设置好的值，变回默认值。

**那 vuex 能做什么呢？**

1. 组件之间全局共享的数据
2. 通过后端异步请求的数据

    把通过后端异步请求的数据都纳入 vuex 状态管理，在 Action 中封装数据的增删改查等逻辑，这样可以一定程度上对前端的逻辑代码进行分层，使组件中的代码更多地关注页面交互与数据渲染等视图层的逻辑，而异步请求与状态数据的持久化等则交由 vuex 管理。

    demo:

    ```js
    // State
    const state = {
      userInfo: {}
    }
    // Mutaion
    const mutations = {
      UPDATE_USER_INFO (state, payload) {
        state.userInfo = payload
      }
    }
    // Action
    export const fetchUserInfo = async ({commit}) => {
    // ... 请求用户数据
    // 调用 Mutaion 写入用户数据
      commit('UPDATE_USER_INFO', userInfo)
    }
    // Component// 在组件中引入 Action
    ...mapAction({
      fetchUserInfoAction: `fetchUserInfo`
    })
    // 在 method 中调用 Action
    let res = self.fetchUserInfoAction()
    ```

3. 可以通过登录者的 token 获取路由，并用 vuex 来存储路由，我们只需要写基本的路由配置，然后可以判断 vuex 中是否有路由判断是否要重新获取路由（如果有路由，则不用获取），而 token 的存储方式是在本地存储。[路由守卫](/2020/04/15/learn-vue/#路由守卫)

## vue

### 基础

Vue.set(target, property, value) / vm.$set(target, proprety, value) 这个用于添加当前 data 列表下的某个对象（数组）的元素。

直接赋值也是可以的，但是没法做到视图更新。

**target 不能是根属性**。

```js
export default {
  data() {
    obj: {

    }
  },
  create() {
    this.$set(this.obj, 'a', 1)
  }
}
```

$event 的使用

在 input 框或者其他的需要用到 change 之类的事件时，默认传参都是 e，我们是没法传第二个参数的，因为那样指定不了 e。

这个时候就需要用到 $event 了

```html
<input @change="handleChange($event, 'emmm')" value="" />

<script>
export default {
  methods: {
    handleChange(e, tag) {
      console.log(e) // 默认的 e
      console.log(tag) // 传入的 emmm
    }
  }
}
</script>
```

### 传值的方式

- 父传子
  1. 通过 props 将数据传递给子组件

    ```html
      // Parent.vue
      <template>
        <Child :parentData="parentData" />
      </template>
      <script>
        exprot default {
          data() {
            parentData: 'this is parent\'s data'
          }
        }
      </script>

      // Child.vue
      <template>
        {{parentData}} <!-- this is parent's data -->
      </template>
      <script>
        export default {
          props: ['parentData'] // 这里必须使用 props 声明从父组件传入的 key
        }
      </script>
    ```

- 子传父
  1. 通过在子组件中触发事件实现向父组件传递数据

    ```html
      // Parent.vue
      <template>
        <Child @getChildData="getChild" />
      </template>
      <script>
        exprot default {
          methods: {
            getChild(data) {
              console.log(data) // -> this is child\' data
            }
          }
        }
      </script>

      // Child.vue
      <template>
        <input type="button" @click="handleClick" />
      </template>
      <script>
        export default {
          methods: {
            handleClick() {
              this.$emit('getChildData', 'this is child\' data') // 第一个参数为父组件传入的函数名称（key），之后的参数为传入该函数的参数
            }
          }
        }
      </script>
    ```

- 其他（包括父子、子父、兄弟、祖孙）
  1. Event Bus 看下面的详细
  2. [发布订阅模式](/2020/03/24/simgle-pubsub/)
  3. vuex 看上面

### event bus

非父子组件之间的通信一般通过一个空的 Vue 实例作为 中转站，也可以称之为 事件中心、event bus。

```js
// 创建事件中心实例
let bus = new Vue()

// 在组件 A 中触发事件
bus.$emit('test', 1)

// 在组件 B 中接受事件
bus.$on('test', (id) => {
// ...
})
```

## vue-router

route 给用户提供了一些属性的，如 name、path、params、query 等
router 提供了修改路由的方法，例如 push、replace 等

### 路由守卫

1. 前置守卫 beforeEach

  通常用来做全局守卫，判断我们账号当前是否登录，然后能否跳转到相对应的页面

  ```js
    // 路由配置

    // 全局守卫
    router.beforeEach((to, from ,next) => {
      // 需要用到账号的页面
      const nextRoute = ['user']
      const loginRoute = ['login']
      // 当前是否登录
      const hasLogin = localStorage.getItem('hasLogin')
      // 未登录且前往需要账号的页面，则跳转到登录页
      if (!hasLogin && nextRoute.indexOf(to.name) > -1) {
        router.push('/login')
      } else if (hasLogin && loginRoute.indexOf(to.name) > -1) {
        router.push('/')
      } else {
        // 其余的正常进行
        next()
      }
    })
    // 导出路由
  ```

  beforeEach 甚至可以用来动态加载路由。

  **思考：是否可以写在全局的 router 中，还是写在单独页面中？如果是全局 router 则有可能每次跳转页面都需要去调用这个 router.beforeEach 函数，但是写在单独页面中需要每个页面都写，或者必要的页面都需要写，会造成冗余。是否可以在登录点击的时候去获取这个路由列表，然后将这个路由列表放入 router 中，并保存在 vuex 中，但是每次重新打开页面都会清空 vuex 中的数据，所以什么时候要再去获取这个路由列表呢，还是保存在本地存储中呢？**。

  ```js
    // 基础路由配置

    // 全局守卫
    router.beforeEach((to, from ,next) => {
      // 根据登录信息（token）获取相关路由
      // 根据获取到的路由信息添加路由
      router.addRoutes(/* 获取到的路由-数组*/)
      next()
    })
    // 导出路由
  ```

### 混入

混入 (mixin) 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。一个混入对象可以包含任意组件选项。

当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。

当组件本身的选项名称与混入对象名称相同时，使用组件本身的选项。

* 全局混入

  ```js
  new Vue({
    mixins: []
  })
  ```

* 局部混入

  ```js
  export default {
    name: '',
    mixins: []
  }
  ```

## vue-cli

vue-cli 脚手架用于快速搭建基础的模块。

```shell
vue create project
```

当我们输入这行指令之后，允许我们去选择所需要的模块等内容，然后根据这些内容从指定的 github 仓库下载代码。

讲 vue-cli 主要讲的应该是他的 webpack 配置。

为什么要配置？还不是为了减少页面初始化的时间，防止首屏加载时间太长，以及拆包啊，代码加密啊，控制台打印啊等一系列东西。

默认使用 vue-cli 搭建的项目是没有 vue.config.js 这个文件的，所以需要新建一个在根目录。

众所周知，webpack 是基于 node 的，所以我们可以在配置文件中去使用 node 的语法。

```js
module.exports = {

}
```

我们必须要将模块导出，这是语法的规定啊。

vue-cli 还内置了一个 add 指令，允许我们直接去加载一些框架，比如 element-ui

```shell
vue add element-ui
```

当然，vue-cli 还有之前的 init 指令等很多指令，只需要 `-h` 一下就可以知道了。

### vue-cli 配置

vue-cli 内置了很多配置，像各种对 css 的预处理 loader。

还可以使用 configureWebpack，chainWebpack 分别对接 webpack 的配置和 webpack-chain

**当使用的是 ts 的时候，怎么写这个配置文件还需要研究一下**。

## 关于 vue 3.0 bate 我想说

我真的学不来了，你们不要更新那么快。
