---
title: 浅谈生命周期
date: 2020-01-20 11:25:56
tags: [前端, react]
categories: 编程
---

浅谈？？？

谈不上吧，就随便说说关于生命周期的我偶尔会用到的东西。

恰恰是那些不常用的决定了你的人生。

<!-- more -->

## componentWillReceiveProps

为什么我会在一开始就讲这个过时的生命周期呢？还不是因为刚写了这个东西。

在 react 17.0 之前的版本，也就是未来版本，这个生命周期将不再使用，改名为 `UNSAFE_componentWillReceiveProps`。 它接收一个新传过来的 props，通常我们会命名为 `nextProps` 当然这因人而异。

```js
class Father extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 1
    };
  }

  add = () => {
    this.setState(state => ({
      count: state.count + 1
    }));
  };

  render() {
    return (
      <div>
        <input type="button" value="click me" onClick={this.add} />
        <Children count={this.state.count}></Children>
      </div>
    );
  }
}

class Children extends React.Component {
  render() {
    return <div>children: {this.props.count}</div>;
  }
}

ReactDOM.render(<Father />, document.getElementById("root"));
```

看，这是我们一个父子组件的传值，它渲染出来的页面大概是这个样子（因浏览器而异）。

![复制粘贴出奇迹](will-receive-props-01.jpg)

当我们每次点击按钮时相对应的下面的数值就会增加 1。这也是我们一开始学习 react 组件的时候会学到的。

可是这跟我们的 componentWillReceiveProps 有什么关系吗？答案是没有，肯定没有啊，我们的 componentWillReceiveProps 就不是这么用的好不好。不然我们要那个 nextProps 干嘛是吧。

我们来先看一下官网的解释：

> React doesn’t call UNSAFE_componentWillReceiveProps() with initial props during mounting. It only calls this method if some of component’s props may update. Calling this.setState() generally doesn’t trigger UNSAFE_componentWillReceiveProps().

大致意思就是在子组件初始化的时候不会去调用这个钩子，而是在更新传入的 props 时调用。这里的更新，这里虽然说是更新，但无论传入的 props 有无变化他都会去调用这个钩子。

回到我们的代码，为了能够更好的看出效果，我们稍微修改一下。

```js
// Father
// 模拟不更新 props 效果
add = () => {
  this.setState((state) => ({
    count: state.count
  }))
}

// Children
componentDidMount() {
  console.log(`DidMount: ${this.props.count}`)
}

UNSAFE_componentWillReceiveProps(nextProps) {
  console.log(`WillReceiveProps: ${JSON.stringify(nextProps)}`)
}
```

这时它的渲染效果是这样的，并在控制台打印出了一点东西。

![自己动手丰衣足食](will-receive-props-02.jpg)

再连续点击几次出现的效果是这样的：

![03.jpg](will-receive-props-03.jpg)

我知道，这时候又会有人说这有什么用，我直接修改 state，更新组件不香吗，为什么还要多此一举？

可以，当然可以，这样的写法也最好不过，只要能力足够，只要程序合理。

试想一下，我们父组件可能有多个子组件吧，每个它要接收一个参数（对象/数组），按照上面的逻辑可以直接让父组件传，父组件就会越来越臃肿，然后我们就进行拆分……是不是只要能力足够就可以了，但是如果程序逻辑方面不允许你这么做呢？

~~这里来模拟一下请求：~~

```js
// Father
add = () => {
this.setState((state) => ({
  count: state.count + 1
}))
}

// Children
constructor(props) {
  super(props)
  this.state = {
    count: 1
  }
}

UNSAFE_componentWillReceiveProps(nextProps) {
  if (this.props.count !== nextProps.count) {
    this.setState({
      count:nextProps.count * 2
    })
  }
}
```

这里的效果就不放了，大概就是每点击一次按钮，子组件的数值就会是传入值的两倍。

这里仅仅只是转变数值，如果换成请求数据，如果从父组件传入数据我们要做的事情就会变得很多，而且还可以降低耦合。

但是我们请求传递的数据肯定不是纯数字或者基础数据类型，而是对象或者数组，那我们就会遇到这样一个问题：**在 componentWillReceiveProps 中，nextProps 和 this.props 相同**。这里就是因为引用的问题，解决方法就是浅拷贝。

直接用控制台来看吧

![控制台](will-receive-props-04.jpg)

就像这样。对于基础数据类型是不会出现引用问题的，但是对于对象、数组就会。

还是使用我们之前的代码：

```js
class Father extends React.Component{
  constructor(props) {
    super(props)
    this.state = {
      counts: []
    }
  }
  add = () => {
    const {counts} = this.state
    counts.push(Math.floor(Math.random() * 10))
    this.setState((state) => ({
      counts
    }))
  }
  render (){
    return (
      <div>
        <input type="button" value="click me" onClick={this.add} />
        <Children counts={this.state.counts}/>
      </div>
    )
  }
}

class Children extends React.Component{
  UNSAFE_componentWillReceiveProps(nextProps) {
    console.log(nextProps)
    console.log(this.props)
  }
  render(){
    return (
      <div></div>
    )
  }
}
```

这里我们每一次点击按钮都会打印出 nextProps 和 this.props。

![控制台](will-receive-props-05.jpg)

会发现我们的 counts 每一组都是一样的，这就是因为对象的引用相同。要解决这个问题也很简单，我们只需要浅拷贝一下就可以了，也就是用解构传参。

```js
// Father
render() {
  return (
    <div>
      <input type="button" value="click me" onClick={this.add} />
      <Children counts={[...this.state.counts]}/>
    </div>
  )
}
```

运行结果：

![结果](will-receive-props-06.jpg)

这样我们就可以避免前后两个 props.counts 的值一样了。

其实在我写项目之前，我也没有去注意过这个钩子，仅仅把它当成过时的不常用钩子看待。在写项目的时候，需要像上面一样由父组件点击，子组件发起请求，最开始一直在 componentDidMount 和 componentDidUpdate 上面徘徊，直到我看到了这个钩子。

## componentWillMount

你以为我这里就要开始正经了？不你错了，我只是随便写写。

这个方法同上面的 `UNSAFE_componentWillReceiveProps` 都是属于过时的不常用钩子（Legacy Lifecycle Methods），所以他们的未来版本写法都是在前面添加一个 `UNSAFE_` 前缀，之后的过时钩子全都是如此写法。

边看文档边写博客简直美滋滋，虽然我试想直接抄，但是用自己的言语组织过的对于自己来说更好理解。

componentWillMount 有几个特性：

* 它在页面挂载（didMount）之前被调用；
* 它在 render() 之前被调用，所以这时候调用 setState() 并不会重新渲染页面；
* 它是服务器上渲染的唯一生命周期。

但也由于这几个特性，它变得可有可无。

* 对于副作用、订阅、请求数据等行为，建议写在 componentDidMount 中；
* 如果需要初始化数据，推荐使用 constructor；
* 比较少用到服务器渲染。

因为种种特性得不到重视或者被更好的钩子代替了，所以这个钩子就比较少用到了，但也不是不能用，就是推荐不要而已。

## componentDidMount

componentDidMount 是使用次数仅次于 render 的一个钩子。它的常常被用来订阅、请求数据、setState。

componentDidMount 会在页面被加载进 Dom tree 的时候立即被调用，所以这个时候特别适合请求数据，并 setState。

componentDidMount 同样适合订阅东西，但需要在 componentWillUnmount 时取消订阅。

官网还提到一个问题（pattern），在调用 componentDidMount 时立即执行 setState，页面会渲染两次，但是这两次渲染会发生在屏幕更新之前，也就是用户看不到渲染两次的效果。

## componentWillUpdate

该生命周期在未来版本写作 UNSAVE_componentWillUpdate。

从文字上看，我们能够知道它是在组件更新之前执行的，我们能够在组件更新前执行这个钩子去执行一些操作。

还有啊，这个钩子不会在创建组件的时候执行。

还有一件事，当 shouldComponentUpdate 返回 false 时，该钩子不执行。

## shouldComponentUpdate

提到 componentWillUpdate 就不能提到 shouldComponentUpdate 这个钩子了。当然，还有一个 componentDidUpdate 钩子，等会再说。

shouldComponentUpdate **必须**返回一个 Boolean，当返回 true 时执行 componentWillUpdate 以及  componentDidUpdate，反之则不执行。

```js
shouldComponentUpdate(props, state) {
  return Math.floor(Math.random()*10) %  2 === 0
}

componentWillUpdate() {
  console.log(`It's componentWillUpdate`)
}

componentDidUpdate() {
  console.log(`It's componentDidUpdate`)
}
```

输出结果有两种情况：

1. 不执行 componentWillUpdate 和 componentDidUpdate
2. 输出 It's componentWillUpdate 之后输出 It's componentDidUpdate

## componentDidUpdate

上面我们也提到了这个钩子，它是在组件更新完毕之后执行的。同样，我们初始化组件时它并不会执行。

切记！在 componentDidUpdate 不要执行 setState，如果非要执行，则需要一个结束条件，否则无限循环直至报错。

```js
componentDidUpdate() {
  if (this.state.count < 10) {
    this.setState((state) => ({
      count: state.count + 1
    }))
  }
}
```

同样，该钩子在 shouldComponentUpdate 返回 false 时不执行。

## render

render 生命周期是每个类组件必须拥有的一个钩子，该生命周期返回一个最外层只有一个标签的 jsx 元素。

在返回的 jsx 中，如果我们需要加入变量或者函数进去，可以使用 `{ }` 包住。但是有几个需要注意的地方：

1. 当包裹的内容是 Booleans 或 null 时

    ```js
    render() {
      return (
        <>
        true: {true}
        false: {false}
        null: {null}
        </>
      )
    }
    ```

    并不会渲染除任何内容。

2. 想要渲染组件时

    `{ }` 允许我们直接返回一整个数组的 jsx 元素，react 会把这个数组渲染进去组件中。

    ```js
    render() {
      <ul>
        {
          [1, 2, 3, 4].map((item) => <li>{item}</li>)
        }
      </ul>
    }
    ```

    上面代码所示，我们就可以渲染除一个列表的 li 标签。但是呢会报错的说，毕竟列表中的每个元素都必须要有自己的 key 值，如果没有，react 会根据下标给每个元素添加 key。具体内容可去看 react 官网的列表。


