---
title: 初学react遇到的不理解的东西
date: 2019-11-28 13:35:56
tags: "react"
categories: "前端"
---

闻道有先后，术业有专攻，如是而已。

<!-- more -->

## 状态提升 & 事件传递

当一个状态（state）在同一父组件下的两个子组件都要使用时，不允许通过操作当前子组件去修改该子组件的 state 间接修改另一子组件的 state。需要将该 state 提升到父组件上。（state 单一化）

通过状态提升之后，原先需要在子组件执行的事件也需要传递到父组件执行（state 在父组件上，不允许子组件修改）。

其实，所谓事件传递，不过就是通过 props 将定义在父组件的事件传递给子组件执行，并没有那么复杂。

```js
function Children(props) {
  return (
    <div onClick={props.onClick}>click me</div>
  )
}

function AnotherChildren(props) {
  return (
    <div>{props.id}</div>
  )
}

class Father extends React.Component{
  constructor(props) {
    super(props);
    this.state = {
      id: 0,
    };
  }

  handleClick(id) {
    this.setState({
      id: id+=1
    })
  }

  render() {
    return (
      <div>
        <Children id={this.state.id} onClick={() => { this.handleClick(this.state.id) }}></Children>
        <AnotherChildren id={this.state.id}></AnotherChildren>
      </div>
    )
  }
}
```

## 组件

组件有两种，函数组件和 class 组件。

当组件不需要有私有 state 时，建议创建函数组件。其所需要的值都可以通过父组件传递过来。

```js
// 函数组件
function Welcome(props) {
  return `<h1>hello {props.name}</h1>`
}

// class 组件
class Welcome extends React.component({
  render() {
    return `<h1>hello {this.props.name}</h1>`
  }
})
```

### props & state

props.children 约等于 vue 中的 slot（插槽）。它包含组件的开始标签和结束标签之间的内容。

props 是传递给组件的（类似于函数的形参），而 state 是在组件内被组件自己管理的（类似于在一个函数内声明的变量）。

大部分写在子组件上的属性都是允许使用 props 读取到的。包括但不限于 className，函数。目前我所知道的只有 `key` 不允许被 props 读取。

setState 是异步的，在某些时候并不会得到我们预想的结果，解决这个问题可以在 setState 中传递一个函数。此时，state 作为第一参数，props 作为第二参数。

```js
this.setState((state) => {
  return {
    count: state.count + 1
  }
})

this.setState((state, props) => ({
  count: state.count + 1
}))
```

其实 this.setState 还给我们提供了第二个参数，一个回调函数，可以解决一部分非预想情况。

```js
this.setState({
  name: 'sheeep'
}, () => {
  {/* something */}
})
```

还可以写多一种普通函数，但是都是一样的。上面这两个函数其实是一样的，不同的地方就是下面的多了个括号，这个关系到箭头函数的语法，为了返回的对象不被解析成函数体添加的括号（ES6 知识点）

> 传递一个函数可以让你在函数内访问到当前的 state 的值。因为 setState 的调用是分批的，所以你可以链式地进行更新，并确保它们是一个建立在另一个之上的，这样才不会发生冲突。

所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。

如果想要修改某些值，以响应用户输入或网络响应，请使用 state。

### 生命周期

关于组件发生什么的时候可以参阅[React.Component](https://zh-hans.reactjs.org/docs/react-component.html)查看发生的步骤

* render（渲染）

  ```js
    render(){
      return (
        {/* 组件 */}
      )
    }
  ```

  render() 方法是 class 组件中唯一必须实现的方法。

  > 在 React.Component 的子类中有个必须定义的 render() 函数。

* constructor（构造函数）

  ```js
    constructor(props) {
      super(props)
    }
  ```

  **如果不初始化 state 或不进行方法绑定，则不需要为 React 组件实现构造函数。**

  > 在 React 组件挂载之前，会调用它的构造函数。在为 React.Component 子类实现构造函数时，应在其他语句之前前调用 super(props)。否则，this.props 在构造函数中可能会出现未定义的 bug。

  constructor被调用是在组件准备要挂载的最一开始，所以此时组件尚未挂载到网页上。

  在 React 中，构造函数仅用于下面两种情况：

  * 通过 this.state 赋值对象来初始化 `内部state`

    ```js
      this.state = {
        stateOne: 0,
      }
    ```

  * 为事件处理函数绑定实例

    ```js
      this.handleClick = this.handleClick.bind(this)
    ```

* mount（挂载）

  ```js
    componentDidMount() {}
  ```
  
  componentDidMount 方法是在组件已经完全被挂在到网页上才会被调用执行，所以可以保证数据的加载。官方设计这个方法是用来加载外部数据的，或处理其他的副作用代码。

* unmount（卸载）

  ```js
    componentDidUnmount() {}
  ```

* update（更新）

  ```js
    componentDidUpdate() {}
  ```

### 不常用生命周期

* shouldComponentUpdate(nextProps, nextState)

根据 shouldComponentUpdate() 的返回值，判断 React 组件的输出是否受当前 state 或 props 更改的影响。

返回值为 false 时告知 React 可以跳过更新。但不会阻止子组件在 state 更改时重新渲染。

* static getDerivedStateFromProps(props, state)

* getSnapshotBeforeUpdate(prevProps, prevState)

* static getDerivedStateFromError()

  此生命周期会在后代组件抛出错误后被调用。 它将抛出的错误作为参数，并返回一个值以更新 state

* componentDidCatch(error, info)

  此生命周期在后代组件抛出错误后被调用。

* componentWillReceiveProps(nextProps)

### 表单

> 在表单组件中，React 的 state 是唯一的数据源。

关于获取用户填入的值，官方例子中，使用的是每次输入都去提交一次 onChange 事件，通过这个事件去修改 state 的值。如果不这么做，不要说修改，连填写都成问题。

正常情况下 input、textarea、select（单选） 的写法相同

select 单选多选

单选只需要配置 value，且 value 是对应值。

多选需要在 select 元素上配置 multiple 属性，值为 true。配置的 value 值为数组。

### 组合

JSX 标签中的所有内容都会作为一个 children

### 事件处理

#### this 问题

在 js 中，class 的方法默认不会绑定 this，像下面写法中，this 的值为 undefined。

```js
class LoggingButton extends React.Component {
  function handleClick() {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

解决方法

1. 使用 class fields 正确绑定回调函数

    ```js
    class LoggingButton extends React.Component {
      // 此语法确保 `handleClick` 内的 `this` 已被绑定。
      // 注意: 这是 *实验性* 语法。
      handleClick = () => {
        console.log('this is:', this);
      }

      render() {
        return (
          <button onClick={this.handleClick}>
            Click me
          </button>
        );
      }
    }
    ```

2. 在构造器中绑定 this（不知道是不是跟上面同一个方法，官方例子使用的大多都是这样写）

    ```js
    class LoggingButton extends React.Component {
      constructor(props) {
        super(props)
        this.handleClick = this.handleClick.bind(this)
      }
      handleClick() {
        console.log('this is:', this);
      }

      render() {
        return (
          <button onClick={this.handleClick}>
            Click me
            </button>
        );
      }
    }
    ```

3. 使用箭头函数

    ```js
    <!-- 一 -->
    class LoggingButton extends React.Component {
      handleClick() {
        console.log('this is:', this);
      }

      render() {
        return (
          <button onClick={() => this.handleClick()}>
            Click me
            </button>
        );
      }
    }

    <!-- 二 -->

    class LoggingButton extends React.Component {
      handleClick = () => {
        console.log('this is:', this);
      }

      render() {
        return (
          <button onClick={this.handleClick}>
            Click me
            </button>
        );
      }
    }
    ```

    > 此语法问题在于每次渲染 LoggingButton 时都会创建不同的回调函数。在大多数情况下，这没什么问题，但如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染。我们通常建议在构造器中绑定或使用 class fields 语法来避免这类性能问题。

#### 向事件处理程序传递参数

在循环中，我们通常会为时间处理函数传递额外的参数。例如，若 id 是你要删除那行的 ID，以下两种方式都可以像事件处理函数传递参数：

```js
<button onClick={e => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

上述两种方式是等价的，分别通过`箭头函数`和`Function.prototype.bind`来实现。

在这两种情况下，React 的事件对象 e 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显示进行传递，而通过 bind 的方式，事件对象以及更多的参数将被隐式的进行传递。

#### context 全局数据

多看...

[动态context](https://reactjs.org/docs/context.html#dynamic-context)

定义一个 context，defaultValue 为默认值。

```js
const MyContext = React.createContext(defaultValue);
```

当组件向上查找没有找到 Provider 时，默认值生效。

```js

const { Provider, Consumer } = React.createContext('defaultValue')

const ProviderComp = (props) => (
  <Provider value={123}>
    {props.children}
  </Provider>
)

const ConsumerComp = () => (
  <Consumer>
    {(value) => <p>{value}</p>}
  </Consumer>
)

// 不嵌套在 ProviderComp 中，defaultValue 生效
ReactDOM.render(
  <ConsumerComp/>,
  document.getElementById('root')
);

// 嵌套在 ProviderComp 中，defaultValue 不生效
ReactDOM.render(
  <ProviderComp><ConsumerComp/></ProviderComp>,
  document.getElementById('root')
);
```

### 受控组件 & 非受控组件

React 推荐使用**受控组件**来处理表单数据。

在一个受控组件中，表单数据是交由 React 组件来管理的。另一种替代方案是使用非受控组件，这时表单数据交友 DOM 节点来处理。

编写一个非受控组件，而不是为每个状态更新都编写数据处理函数，可以使用 ref 从 DOM 节点中获取表单数据。

```js
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.input = React.createRef();
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

## Hooks

因为在项目开发过程中，组件之间复用状态逻辑变得很难，组件的复杂程度使得组件变得难以理解，各种各样的原因便有了 Hook 的诞生。

> Hook 不能在 class 组件中使用 —— 这使得你不使用 class 也能使用 React。

hooks 可以让你在非 class 组件中使用 state。它包括两个模块 `useState` 和 `useEffect`。

你看吧，我又以偏概全了，明明还有很多，甚至还可以自定义 Hook。

### state hooks

```js
import {useState} from 'react'
function Child() {
  // 这是一个纯函数组件
  const [count, setCount] = useState(0)
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  )
}
```

其中 useState 返回两个数据（一个数组？？？），第一个是初始化数据，第二个是更新数据的方法。如上面例子中的 count 和 setCount。setCount 类似于类组件中的 this.setState 方法，但是它不会合并现有数据。也就是说，在更新函数中传入同一个对象是无法更新的。

```js
function Child() {
  // 这是一个纯函数组件
  const [list, setList] = useState([0, 1])
  const emmm = () => {
    list.push(Math.floor(Math.random() * 10))
    console.log(list) // list数据改变
    // setList(list) // 页面没有更新
    setList(list.slice()) // 页面更新了
  }
  return (
    <div>
      <button onClick={() => emmm()}>
        Click me
      </button>
      <ul>
        {list.map((item => {
          return <li key={item}>
            {item}
          </li>
        }))}
      </ul>
    </div>
  )
}
```

### effect hooks

useEffect 类似于类组件的 componentDidMount, componentDidUpdata, componentWillUnmount，只不过是合成了一个函数。

useEffect 允许返回一个函数，用于在**销毁组件**时运行，此时可以做取消订阅等操作。

```js
function Effect(props) {
  useEffect(() => {
    console.log('现在组件已经加载好了')
    return () => {
      alert('我只会在组件销毁时执行哦')
    };
  })
  return (
    <div>{props.source}</div>
  )
}
```

当然，useEffect 可以用来获取数据，可以用来修改 useState 的值等等常规操作。

useEffect 允许添加第二个参数（数组），用来判断是否需要更新。

```js
function Effect(props) {
  useEffect(() => {
    console.log('现在组件已经加载好了')
    return () => {
      alert('我只会在组件销毁时执行哦')
    };
  }, [props.count])
  return (
    <div>{props.source} , {props.count}</div>
  )
}
```

修改一下上面代码，添加一个 props.count，这里的数组只有要给元素，所以当且仅当 props.count 改变时会进行更新。

> 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（[]）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行。这并不属于特殊情况 —— 它依然遵循依赖数组的工作方式。

**useState 和 useEffect 都允许多次使用**这是与类组件不一样的地方。

## 代码分割

> 当使用 Babel 时，你要确保 Babel 能够解析动态 import 语法而不是将其进行转换。对于这一要求你需要 babel-plugin-syntax-dynamic-import 插件。

类似于懒加载的加载模块或组件

React.lazy 函数能让你像渲染常规组件一样处理动态引入（的组件）。

使用之前：

```js
import OtherComponent from './OtherComponent';
```

使用之后：

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```

此代码将会在组件首次渲染时，自动导入包含 OtherComponent 组件的包。

React.lazy 接受一个函数，这个函数需要动态调用 import()。它必须返回一个 Promise，该 Promise 需要 resolve 一个 defalut export 的 React 组件。

## 高阶组件

> 高阶组件是参数为组件，返回值为新组件的函数

## 记录需要多看的地方

* [context](https://reactjs.org/docs/context.html#dynamic-context)
* [传递组件给函数](https://reactjs.org/docs/faq-functions.html)

## 提示

### 不要过度思考

如果你刚刚开始一个项目，不要花超过五分钟在选择项目文件组织结构上。选择上述任何方式（或提出自己的方式）并开始编写代码！因为，在你编写了一些真正的代码之后，你将很有可能会重新考虑它。

如果您感觉完全卡住，请先将所有文件保存在同一个文件夹中。它最终会变得足够大，以至于让你想要将其中一些文件拆分出去。到那时，你将有足够的知识去区分你最频繁编辑的文件。通常，将经常一起变化的文件组织在一起是个好主意。这个原则被称为 “colocation”。

随着项目规模的扩大，人们通常会在实践中混搭使用上述这些方式。因此，在开始时选择“正确”的那个方式并不是很重要。
