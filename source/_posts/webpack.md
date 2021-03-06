---
title: webpack 基本配置与第三方构建工具
date: 2020-05-18 11:06:18
tags: 基础
categories: 编程
---

> 我们竟也平凡到自命不凡。

只要我们使用到了 npm，就无法避免会使用到 webpack，当然，gulp 也行，但我主要用的是 webpack。

这里主要参考的是 v4.43.0 版本

<!-- more -->

## 基本配置

```js
// webpack.config.js

// 因为 webpack 是基于 node 写的，所以可以使用 node 的核心语法
const path = require("path");

// 要使用插件，需要先 require 它
const HtmlWebpackPlugin = require("html-webpack-plugin");

// 配置，四个核心概念 entry output loader plugins
const config = {
  // 模式
  mode: "production", // development

  // 入口，指示 webpack 应该使用哪个模块，来作为构建其内部*依赖图*的开始
  entry: "./src/main.js",

  // output 属性告诉 webpack 在哪里输出它所构建出来的 bundles（由入口文件开始，根据依赖一步步被处理最后生成的文件（集）），以及如何命名。
  // 默认值为 ./dist。基本上，整个应用程序结构，都会被编译到指定的输出路径的文件夹中。
  output: {
    path: path.resolve(__dirname, "dist"), // dist 前面不需要加 /
    filename: "webpack.bundle.js", // 这里的名称随便写都行
  },

  // loader 会将所有类型的文件，转换为应用程序的依赖图（和最终 bundle）可以直接引用的模块。
  // 在 webpack 配置中定义 loader 时，要定义在 module.rules 中，而不是 rules 或者 loaders 中。
  module: {
    rules: [
      // 告知 webpack 当遇到 「在 require() / import 语句中被解析为 '.txt' 的路径」时，先用 raw-loader 转换一下再进行编译。
      { test: /\.txt$/, use: "raw-loader" },
    ],
  },

  // 相较于 loader 的转换文件类型，plugin 的功能范围更广，它包括从打包优化和压缩，一直到重新定义环境中的变量，它的功能强大，可以用来处理各式各样的任务。
  plugins: [new HtmlWebpackPlugin("./src/index.html")],
};
module.exports = config;
```

## loader

loader 用于对模块的源代码进行转换。loader 可以使你在 import 或“加载”模块时预处理文件。
loader 可以将文件从不同的语言（typescript）转换成 javascript，或者将内联图像转换成 data URL。
loader 甚至允许你直接在 javascript 模块中 import CSS 文件。

拿 css-loaer 和 ts-loader 举例：

首相安装相对应的 loader：

```shell
yarn add -D css-loader
yarn add -D ts-loader
```

然后指示 webpack 对每个 .css 使用 css-loader，以及对所有 .ts 文件使用 ts-loader：

```js
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: "css-loader" },
      { test: /\.ts$/, use: "ts-loader" },
    ],
  },
};
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

- image-webpack-loader

- less-loader

- css-loader

- ts-loader

- style-loader

## plugin 插件

从热心网友那边抄来的解释：plugin 是一个扩展器，它丰富了 webpack 本身，针对是的 loader 结束后，webpack 打包的整个过程，它并不直接操作文件，而是基于事件机制工作，会监听 webpack 打包过程中的某些节点，执行广泛的任务。

官网上的剖析（原理）：webpack 插件是一个具有 apply 属性的 JavaScript 对象。apply 属性会被 webpack compiler 调用，并且 compiler 对象可在整个编译生命周期访问。

用法：由于插件可以携带参数/选项，正常情况下在 webpack 配置中，需要向 plugins 属性中传入的是 new 实例对象。

```js
const HtmlWebpackPlugin = require("html-webpack-plugin"); //通过 npm 安装
const webpack = require("webpack"); //访问内置的插件
const path = require("path");

const config = {
  entry: "./path/to/my/entry/file.js",
  output: {
    filename: "my-first-webpack.bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: "babel-loader",
      },
    ],
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({ template: "./src/index.html" }),
  ],
};

module.exports = config;
```

### 部分插件

这里先只讲基础作用，至于配置需要看文档，可以上[官网](https://www.webpackjs.com/plugins/)，也可以去 npm 上看。

- esbuild 用于构建时加快构建速度，减少内存消耗。

  鉴于当前的版本以及稳定性来说，我还没有使用过。

- compression-webpack-plugin 预先准备的资源压缩版本

- uglifyjs-webpack-plugin

- html-webpack-plugin

- clean-webpack-plugin 自动清理构建目录

```js
const CleanWebpackPlugin = require("clean-webpack-plugin");
module.exports = {
  // ... 其他配置
  plugins: [new CleanWebpackPlugin()],
};
```

- postCSS 自动补齐 CSS3 前缀

```js
module.exports = {
  module: {
    rules: [
      {
      test: /\.less$/,
        use: [
          'style-loader',
          'css-loader',
          'less-loader',
          {
            loader: 'postcss-loader',
            options: {
              // postCSS 属于 loader
              // autoprefixer 属于 plugin
              plugins: () => [
                require('autoprefixer')({
                  browsers: ["last 2 version", "> 1%", "iOS 7"]
                })
              ]
            }
          }
        ]
      }
    ]
  }
};

作者：前端小佬
链接：https://juejin.im/post/5ec90fb46fb9a047f2297f07
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

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

## 基于 vue-cli 的 webpack 配置

这个是我使用在鼎坚仓储管理系统的配置。

> vue.config.js 是一个可选的配置文件，如果项目的 (和 package.json 同级的) 根目录中存在这个文件，那么它会被 @vue/cli-service 自动加载。

```js
// vue.config.js

const path = require("path");
const UglifyJsPlugin = require("uglifyjs-webpack-plugin");
const CompressionPlugin = require("compression-webpack-plugin");
const { HashedModuleIdsPlugin } = require("webpack");

function resolve(dir) {
  return path.join(__dirname, dir);
}

// cdn 预加载使用
const externals = {
  vue: "Vue",
  "vue-router": "VueRouter",
  vuex: "Vuex",
  axios: "axios",
};

const cdn = {
  build: {
    js: [
      "https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.min.js",
      "https://cdn.jsdelivr.net/npm/vue-router@3.0.1/dist/vue-router.min.js",
      "https://cdn.jsdelivr.net/npm/vuex@3.0.1/dist/vuex.min.js",
      "https://cdn.jsdelivr.net/npm/axios@0.18.0/dist/axios.min.js",
    ],
  },
};

// vue.config.js
module.exports = {
  /*
    Vue-cli3:
    Crashed when using Webpack `import()` #2463
    https://github.com/vuejs/vue-cli/issues/2463
   */
  // 如果你不需要生产环境的 source map，可以将其设置为 false 以加速生产环境构建。
  productionSourceMap: false,

  // 打包app时放开该配置
  // publicPath: './',
  configureWebpack: (config) => {
    const plugins = [];
    if (process.env.NODE_ENV === "production") {
      plugins.push(
        new UglifyJsPlugin({
          uglifyOptions: {
            output: {
              comments: false, // 去掉注释
            },
            warnings: false,
            compress: {
              drop_console: true,
              drop_debugger: false,
              pure_funcs: ["console.log"], // 移除 console.log
            },
          },
        })
      );

      // 代码压缩，服务器也要相应开启
      plugins.push(
        new CompressionPlugin({
          algorithm: "gzip",
          test: /\.(js|css)$/, // 匹配文件名
          threshold: 10000, // 对超过10k的数据压缩
          deleteOriginalAssets: false, // 不删除源文件
          minRatio: 0.8, // 压缩比
        })
      );

      // 用于根据模块的相对路径生成 hash 作为模块 id, 一般用于生产环境
      plugins.push(new HashedModuleIdsPlugin());

      // 开启分离js
      config.optimization = {
        runtimeChunk: "single",
        splitChunks: {
          chunks: "all",
          maxInitialRequests: Infinity,
          minSize: 1000 * 60,
          cacheGroups: {
            vendor: {
              test: /[\\/]node_modules[\\/]/,
              name(module) {
                // 排除node_modules 然后吧 @ 替换为空 ,考虑到服务器的兼容
                const packageName = module.context.match(
                  /[\\/]node_modules[\\/](.*?)([\\/]|$)/
                )[1];
                return `npm.${packageName.replace("@", "")}`;
              },
            },
          },
        },
      };

      // 取消webpack警告的性能提示
      config.performance = {
        hints: "warning",
        // 入口起点的最大体积
        maxEntrypointSize: 1000 * 500,
        // 生成文件的最大体积
        maxAssetSize: 1000 * 1000,
        // 只给出 js 文件的性能提示
        assetFilter: function (assetFilename) {
          return assetFilename.endsWith(".js");
        },
      };

      // 打包时npm包转CDN
      config.externals = externals;

      return { plugins };
    }
  },
  chainWebpack: (config) => {
    if (process.env.NODE_ENV === "production") {
      // 压缩图片
      // config.module
      //     .rule('images')
      //     .test(/\.(png|jpe?g|gif|svg)(\?.*)?$/)
      //     .use('image-webpack-loader')
      //     .loader('image-webpack-loader')
      //     .options({ bypassOnDebug: true })

      // webpack 会默认给 commonChunk 打进 chunk-vendors，所以需要对 webpack 的配置进行 delete
      config.optimization.delete("splitChunks");

      config.plugin("html").tap((args) => {
        if (process.env.NODE_ENV === "production") {
          args[0].cdn = cdn.build;
        }
        if (process.env.NODE_ENV === "development") {
          args[0].cdn = cdn.build;
        }
        return args;
      });

      if (process.env.npm_config_report) {
        config
          .plugin("webpack-bundle-analyzer")
          .use(require("webpack-bundle-analyzer").BundleAnalyzerPlugin)
          .end();
      }
      // 移除 prefetch 插件，prefetch 插件会事先加载所有的模块
      config.plugins.delete("prefetch");
      config.plugins.delete("preload");
    }

    config.resolve.alias
      .set("@$", resolve("src"))
      .set("@api", resolve("src/api"))
      .set("@assets", resolve("src/assets"))
      .set("@comp", resolve("src/components"))
      .set("@views", resolve("src/views"))
      .set("@layout", resolve("src/layout"));

    // 配置 webpack 识别 markdown 为普通的文件
    config.module
      .rule("markdown")
      .test(/\.md$/)
      .use()
      .loader("file-loader")
      .end();
  },

  css: {
    loaderOptions: {
      less: {
        modifyVars: {
          /* less 变量覆盖，用于自定义 ant design 主题 */
          "primary-color": "#1890FF",
          "link-color": "#1890FF",
          "border-radius-base": "4px",
        },
        javascriptEnabled: true,
      },
    },
  },

  devServer: {
    port: 3000,
    proxy: {
      "/api": {
        target: "http://localhost:3000", //mock API接口系统
        ws: false,
        changeOrigin: true,
      },
    },
  },

  lintOnSave: undefined,
};
```

## TODO

用自己的项目打包呀
