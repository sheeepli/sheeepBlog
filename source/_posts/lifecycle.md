---
title: 浅谈生命周期
date: 2020-01-20 11:25:56
tags: "react"
categories: "前端"
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

其实在我写项目之前，我也没有去注意过这个钩子，仅仅把它当成过时的不常用钩子看待。在写项目的时候，需要像上面一样由父组件点击，子组件发起请求，最开始一直在 componentDidMount 和 componentDidUpdate 上面徘徊，直到我看到了这个钩子。
