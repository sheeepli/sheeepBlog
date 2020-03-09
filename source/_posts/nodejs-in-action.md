---
title: nodejs 实战第二版
date: 2020-02-15 22:29:28
tags: ['后台', '书籍']
categories: 编程
---

确认生命中的荒诞感绝不可能是一个终点，而恰恰是一个开始。 ——加廖《局外人》

一篇关于 Nodejs 实战的抄写。

<!-- more -->

## 目录

第三章为我们提供了一个完整的 Node Web 应用程序搭建教程。
第四章专门介绍前端构建系统，以及 npm 脚本、Gulp 和 Webpack 的用法
第六章介绍了几个重要的 Node 服务器端框架。
第七章专门介绍了 Express 和 Connect。
第八章是 Web 应用程序模板
第十二章详细介绍如何使用 Electron 写程序。
附录B抓取网页

## 核心模块

Node 不仅有文件系统库（fs、path）、TCP 客户端和服务端库（net）、HTTP库（http和https）和域名解析库（dns），还有一个经常用来写测试的断言库（assert），以及一个用来查询平台信息的操作系统库（os）。

### 文件系统

```js
const fs = require('fs')
const zlib = require('zlib')
const gzip = require('gzip')
const outStream = fs.createWriteStream('output.js.gz')

fs.createReacStream('./index.js')
  .pipe(gzip)
  .pipe(outStream)
```

上述代码中，使用了 Node 的 fs 模块创建了读和写流，然后把他们通过另外一个流（gzip）连接起来传输数据，就这个例子而言，就是压缩。

### 网络

使用 http 模块写 Hello World

```js
const http = require('http')
const port = 8080

const server = http.createServer((req, res) => {
  res.end('Hello World')
})

server.listen(port, () => {
  console.log('Server is running on http://localhost:%s', port)
})
```

在 Node 中搭一个服务器只需要加载 http 模块，然后给它一个函数。这个函数有两个参数，即请求和响应。

## 主流的 Node 程序

Node 程序主要可以分成三种类型：Web 应用程序、命令行工具和后台程序、桌面程序。

提供单页应用的简单程序、REST 微服务以及全栈的 Web 应用都属于 Web 应用程序。

命令行工具类似于 npm、Gulp 和 Webpack。后台程序就是后台服务，比如 PM2 进程管理器。

桌面程序一般是用 Electron 框架写的软件，Electron 用 Node 作为基于 Web 的桌面应用的后台。Atom 和 VSC 文本编辑器都属于这一类。

### Web 应用程序

一个简单的 Web 应用程序。

```js
// index.js
const express = require('express')
const app = express()

app.get('/', (req, res) => {
  res.send('Hello Express')
})

app.listen(3000, () => {
  console.log(`Express is running`)
})
```

控制台执行 node ./index.js 之后会输出 'Express is running'，打开 <http://localhost:3000> 就可以看到页面上显示着 Hello Express

因为常用的也就是应用程序，所以只写这个，如果需要，可再次阅读第一章第五节

## 编程基础

### 如何组织代码

在创建程序时，不管是用 Node 还是其他工具，基本不可能把所有代码都放到一个文件中。

当出现这种情况时，传统的方式是按照逻辑相关性对代码分组，将包含大量代码的单个文件分解成多个文件，也叫做包或者模块。

* 创建模块

  模块既可以是一个文件，也可以是包含一个或多个文件的目录。
  
  如果模块是目录，Node 通常会在这个目录下找一个叫 index.js 的文件作为模块的入口。之所以说通常，是因为这个入口是可以配置的，只需要在当前目录下创建要给 package.json 文件，通过配置 main 指向的文件路径就可以改变入口。

  在 ES6 之前的年代里，我们都是在通过 requireJS 去引入其他的模块。

  ```js
  // modules a
  function fun(num) {
    return num * 2;
  }
  module.exports=fun;
  // module.exports.fun=fun;

  // modules b
  const fun = require('./a');
  // const {fun} = require('./a');
  const res = fun(100); // 此时结果为 200
  ```

  导出两种写法，如上都可。

  而在现在，我们可以直接使用 ES6 的语法引入模块。

  ```js
  // modules a
  function fun(num) {
    return num * 2;
  }
  export default fun;

  // modules b
  import fun from './a';
  const res = fun(100); // 此时结果为 200
  ```

  注：以上代码需要使用 babel 转换成 ES5 才能运行。

* 模块搜索

  模块搜索的规则（不指定目录）

```flow
  st=>start: 开始搜索
  e1=>end: 返回模块
  e2=>end: 抛出异常
  cond1=>condition: 当前目录下？
  cond2=>condition: 核心模块？
  cond3=>condition: 当前目录 node_modules下？
  cond4=>condition: 存在父目录？
  cond5=>condition: 在环境变量 NODE_PATH 指定的目录下？

  st->cond1(yes)->e1
  cond1(no)->cond2(yes)->e1
  cond2(no)->cond3(yes)->e1
  cond3(no)->cond4(yes)->cond3
  cond4(no)->cond5(yes)->e1
  cond5(no)->e2
```

  <!-- TODO: Node 能把模块作为对象缓存起来。怎么写 -->

### 如何实现异步编程

类似于前端的点击触发的逻辑等都属于异步编程。服务端也一样：事件发生会触发响应逻辑。

在 Node 中存在两种响应逻辑管理方式：回调和事件监听。

**回调**通常用来定义一次性响应的逻辑。比如对数据库查询，可以指定一个回调函数来确定如何处理查询结果。这个回调函数可能会显示数据库查询结果，根据这些结果做些计算，或者一查询结果为参数执行另一个回调函数。

**事件监听器**本质上也是要给回调，不同而是，它跟一个概念实体（事件）相关。例如，当有人在浏览器中点击鼠标时，鼠标点击就是一个需要处理的事件。在 Node 中，当有 HTTP 请求过来时，HTTP 服务器会发出一个 request 事件。可以监听那个 request 事件，并添加一些响应逻辑。在下面例子中，因为用 EventEmitter.prototype.on 方法在服务器上绑定了一个监听器，所以每当有 request 事件发出时，服务器就会调用 handleRequest 函数：

```js
server.on('request', handleRequest)
```

一个 Node HTTP 服务器实例就是一个事件发射器，一个可以继承、能够添加事件发射及处理能力的类（EventEmitter）。Node 的很多核心功能都继承自 EventEmitter，也能创建自己的事件发射器。

1. 如何响应一次性事件

    响应一次性事件我们通常使用**回调**来处理。

    ```js
    http.createServer((req, res) => {
      if (req.url === '/') {
        fs.readFile('./title.json', (err, data) => {
          if (err) {
            console.error(err)
            res.end('server error')
          } else {
            const titles = JSON.parse(data.toString());
            fs.readFile('./index.html', (err, data) => {
              if (err) {
                console.error(err)
                res.end('Server Error')
              } else {
                const tmpl = data.toString()
                const html = tmpl.replace('%', titles.join('</li><li>'))
                res.writeHead(200, {'Content-Type': 'text/html'})
                res.end(html)
              }
            })
          }
        })
      }
    })
    ```

    如上代码，我们每次 readFile 函数都需要创建一个回调函数。在这样的代码中，我们会嵌套很多回调，这样会使代码看起来很乱，重构和测试会变得困难，所以需要限制回调的嵌套层级。

    我们可以把每层的回调嵌套提取出来做成命名函数。

    ```js
    getTitle(res) {
      /* some code */
    }
    getTemp(res) {
      /* some code */
    }
    ```

    不单单是提取出函数，在众多的 if/else 条件中，我们可以使用 return 减少嵌套

    ```js
    fs.readFile('./title.json', (err, data) => {
      if (/* ... */) return
      /* some code */
    })
    ```

    **注意** Node 中的大多数内置模块在使用回调时都会带两个参数：第一个用来放可能会发生的错误；第二个用来放结果。错误参数经常缩写为 err。

2. 如何处理重复性事件

    page 28

 3. 如何让异步逻辑顺序执行

## web 程序

### 使用 npm

web 程序的搭建少不了 npm 的协助，现在也可以使用 yarn 来安装模块。

```shell
mkdir later
cd later
npm init -fy
npm i express --save
```

前两行代码分别是创建 later 目录和进入 later 目录。

第三行代码是初始化当前目录同时会创建一个 package.json 文件用来存放一些配置。

第四行代码是用来安装模块的，当前安装的是 express 模块，--save 是将这个模块写入 package.json 中，方便模块管理和安装。

### 搭建一个 RESTful Web 服务

设计 RESTful 服务时，要想要需要哪些操作，并将它们映射到 Express 里的路由上。

假如我们需要实现保存文章、获取文章、获取包含所有文章的列表和删除不再需要的文章这几个功能，我们可以对应下面这些路由：

* POST /articles —— 创建新文章
* GET /articles/:id —— 获取指定文章
* GET /articles —— 获取所有文章
* DELETE /articles/:id —— 删除指定文章

以下是用 express 实现的部分代码：

```js
const express = require('express')
const bodyParser = require('body-parser')

const app = express()

const articles = [{title: 'Example'}]

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

app.set('port', process.env.PORT || 3000)

app.get('/', (req, res, next) => {
  res.send(articles)
})

app.post('/articles', (req, res, next) => {
  let str = ''
  articles.push({title: req.body.title})
  res.send("ok");
})

app.get('/articles/:id', (req, res, next) => {
  const id = req.params.id
  res.send(articles[id])
})

app.delete('/articles/:id', (req, res, next) => {
  const id = req.params.id;
  delete articles[id]
  res.send({message: 'deleted'})
})

app.listen(app.get('port'), () => {
  console.log(`app is running at ${app.get('port')}`)
})
```

关于处理 post 请求需要消息体解析，所以引入了官方的解析器：[body-parser](https://www.npmjs.com/package/body-parser)。

我们给程序添加了两个功能：JSON 消息体解析（`app.use(bodyParser.json())`）和表单解码消息体解析（`app.use(bodyParser.urlencoded({extended: true})`）。

### 数据库

按照书本，这里我也使用 sqlite3
