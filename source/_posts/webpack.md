---
title: webpack 基本配置与第三方构建工具
date: 2020-05-18 11:06:18
tags: 基础
categories: 编程
---

只要我们使用到了 npm，就无法避免会使用到 webpack，当然，gulp 也行，但我主要讲的是 webpack。

这里主要参考的是 v4.43.0 版本

<!-- more -->

## 基本配置

```js
// webpack.config.js

// 因为 webpack 是基于 node 写的，所以可以使用 node 的核心语法
const path = require('path');

// 要使用插件，需要先 require 它
const HtmlWebpackPlugin = require('html-webpack-plugin');

// 配置，四个核心概念 entry output loader plugins
const config = {
  // 模式
  mode: 'production', // development

  // 入口，指示 webpack 应该使用哪个模块，来作为构建其内部*依赖图*的开始
  entry: './src/main.js',

  // output 属性告诉 webpack 在哪里输出它所构建出来的 bundles（由入口文件开始，根据依赖一步步被处理最后生成的文件（集）），以及如何命名。
  // 默认值为 ./dist。基本上，整个应用程序结构，都会被编译到指定的输出路径的文件夹中。
  output: {
    path: path.resolve(__dirname, 'dist'), // dist 前面不需要加 /
    filename: 'webpack.bundle.js' // 这里的名称随便写都行
  },

  // loader 会将所有类型的文件，转换为应用程序的依赖图（和最终 bundle）可以直接引用的模块。
  // 在 webpack 配置中定义 loader 时，要定义在 module.rules 中，而不是 rules 或者 loaders 中。
  module: {
    rules: [
      // 告知 webpack 当遇到 「在 require() / import 语句中被解析为 '.txt' 的路径」时，先用 raw-loader 转换一下再进行编译。
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },

  // 相较于 loader 的转换文件类型，plugin 的功能范围更广，它包括从打包优化和压缩，一直到重新定义环境中的变量，它的功能强大，可以用来处理各式各样的任务。
  plugins: [
    new HtmlWebpackPlugin('./src/index.html')
  ]
}
module.exports = config
```

## loader

loader 用于对模块的源代码进行转换。loader 可以使你在 import 或“加载”模块时预处理文件。
loader 可以将文件从不同的语言（typescript）转换成 javascript，或者将内联图像转换成 data URL。
loader 甚至允许你直接在 javascript 模块中 import CSS文件。

拿 css-loaer 和 ts-loader 举例：

首相安装相对应的 loader：

```shell
yarn add -D css-loader
yarn add -D ts-loader
```

然后指示 webpack 对每个 .css 使用 css-loader，以及对所有 .ts 文件使用 ts-loader：

```js
module.exports= {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
}
```

test 属性用于匹配文件，use 告知 webpack 需要用到哪些 loader。

上面例子中，我们是对两种不同类型的文件进行一次处理，但如果我们想要对一种类型的文件多次处理呢？总不能写很多次匹配叭，太不直观了。

在配置 loader 时，如果我们只做一次处理，那么 use 的值则可以为字符串。需要多次处理时，use 变为一个数组，可以匹配多个 loader。

安装 style-loader：

```shell
yarn add -D style-loader
```

修改 module.rules

```js
module: {
  rules: [
    test: /\.css$/,
    use: [
      { loader: 'style-loader' },
      {
        loader: 'css-loader',
        options: {
          // TODO: 不晓得这里面的内容怎么写，到时候看看叭
          modules: true
        }
      }
    ]
  ]
}
```

loader 的执行顺序是有后往前执行的，所以先执行的是 css-loader，然后执行的才是 style-loader。

### 部分 loader

* image-webpack-loader

* less-loader

* css-loader

* ts-loader

* style-loader

## plugin 插件

从热心网友那边抄来的解释：plugin 是一个扩展器，它丰富了 webpack 本身，针对是的 loader 结束后，webpack 打包的整个过程，它并不直接操作文件，而是基于事件机制工作，会监听 webpack 打包过程中的某些节点，执行广泛的任务。

官网上的剖析（原理）：webpack 插件是一个具有 apply 属性的 JavaScript 对象。apply 属性会被 webpack compiler 调用，并且 compiler 对象可在整个编译生命周期访问。

用法：由于插件可以携带参数/选项，正常情况下在 webpack 配置中，需要向 plugins 属性中传入的是 new 实例对象。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); //通过 npm 安装
const webpack = require('webpack'); //访问内置的插件
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

### 部分插件

这里先只讲基础作用，至于配置需要看文档，可以上[官网](https://www.webpackjs.com/plugins/)，也可以去 npm 上看。

* esbuild 用于构建时加快构建速度，减少内存消耗。

  鉴于当前的版本以及稳定性来说，我还没有使用过。

* compression-webpack-plugin

* uglifyjs-webpack-plugin

* html-webpack-plugin

## 插播一个关于 Babel 的知识

Before we start telling Babel what to do. We need to create a configuration file. All you need to do is create a `.babelrc` file at the root of your project.

这是一个基础的 `.bablerc` 文件

```json
{
  "presets": [],
  "plugins": []
}
```

我们需要安装 `babel-preset-2015` 来将 ES6 编译成 ES5

```shell
npm i --save-dev babel-preset-2015
```

将 `bable-preset-2015` 写入 presets。

注意这里是不需要写 `babel-reset-` 只需要写 `es2015`，编译器会自动识别。

```json
{
  "presets": ["es2015"],
  "plugins": []
}
```

你看到那个 plugins 了吗，那个是用来写插件的，就是那些以 `babel-plugin-` 开头的插件。

先来说说这个按需加载叭 [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import)

其实写法都是一样的，但是插件需要配置一些东西。

```shell
  npm i --save-dev babel-plugin-import
```

修改 .babelrc 文件

```json
{
  "presets": ["es2015"],
  "plugins": [
    [
      "import",
      {
        "libraryName": "antd" // 这里使用到的是 antd 如果使用的是 element，则只需要将 antd 换成 element 就可以了。
      }
    ]
  ]
}
```

我们还可以安装更多的 babel 预设，大部分预设都是以 babel-preset- 开头。

当然我们可以把 .babelrc 配置写在 webpack.config.js 中，但是写在 .babelrc 还是一个最佳的选择。
