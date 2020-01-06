---
title: 19年第52周
date: 2019-12-24 19:53:02
categories: "周记"
---

## bilibili

### 关于暴露 webpack 配置

如果我们直接使用 `npx create-react-app xxx` 脚手架创建项目的话，我们通常是看不到 webpack 配置的。这是 react 帮我们隐藏了配置文件夹，当然它也写好了如何暴露 webpack 配置。

```json
// package.json
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
```

在 package.json 文件中，有这么一个语句 eject，这是帮我们暴露 webpack 配置的语句，我们只需要执行 `npm run eject` 就可以了。

有一个问题就是在执行这条语句之前，不能对文件进行修改，不然会报错（其实就是合并失败）。

### 按需加载 babel-plugin-import

### 关于 react 项目 public 目录

/public/ 是项目最后通过 build 打包生成的一个目录，所有源码都会生成在这个文件夹下面，然会把public下的所有子文件部署到服务器上运行，所以一般会把静态文件或者图片会保存在这里。

/public/ 并不会部署，只是它的子文件部署。

在项目中引用 /public/ 中的文件可以直接 /xxx 引入。
