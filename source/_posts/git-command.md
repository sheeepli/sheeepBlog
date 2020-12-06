---
title: git 指令
date: 2020-05-12 13:41:58
tags: git
categories: 编程
---

日常会用到的一些 git 指令。

<!-- more -->

初始化 git 仓库

```shell
git init
```

添加修改值存储中

```shell
git add < file | .>
```

如果指定名称，则只提交相对应文件，如果使用 `.` 则存储所有修改。

提交本次修改

```shell
git commit -m < what's new >
```

上传本次提交

- 普通推送

  ```shell
  git push
  ```

- 强制推送

  ```shell
  git push -f origin master
  ```

下载最新的版本

```shell
git pull
```

创建分支

```shell
git branch sheeep
```

切换分支

```shell
git checkout sheeep
```

创建并切换分支

```shell
git checkout -b sheeep
```

查询版本号

```shell
git log
```

切换版本号

```shell
git checkout < 版本号的前六位 | -数字（前几个版本号）->
```

查看远程

```shell
git remote
origin ## 如果有多个会全部显示出来
```

查看远程地址

```shell
git remote get-url origin
<远程的地址>
```

删除指定远程

```shell
git remote rm origin
```

添加远程地址

```shell
git remote add origin < url >
```

主要难理解的是分支那一块内容

但是这些指令肯定还有参数补充，可以参考[官网](https://git-scm.com/docs)
