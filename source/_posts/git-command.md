---
title: git 指令
date: 2020-05-12 13:41:58
tags: git
categories: 编程
---

日常会用到的一些 git 指令。

<!-- more -->

创建一个 git 仓库

git init

添加修改值存储中

git add < file | .>

提交本次修改

git commit -m < what's new >

上传本次提交

* 普通提交
  git push

* 强制提交
  git push -f origin master

下载最新的版本

git pull

创建分支

git branch sheeep

切换分支

git checkout sheeep

查询版本号

git log

切换版本号

git checkout < 版本号的前六位 | -数字（前几个版本号）->

主要难理解的是分支那一块内容

但是这些指令肯定还有参数补充，可以参考[官网](https://git-scm.com/docs)
