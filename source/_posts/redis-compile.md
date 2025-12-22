---
title: redis交叉编译arm版本
date: 2022-11-02 23:32:31
tags: [第三方库,交叉编译]
categories: 交叉编译
---

# redis交叉编译arm版本

## 背景
遇到以下报错时，记得更新redis版本，用最新的redis代码编译
![redis_error](/images/redis_error.jpg)

## 编译环境
在预先已经配置好交叉编译环境的虚拟机里，编译`arm`版本的`redis`

## 编译步骤
1.修改`redis/deps`的`Makefile`，在`jemalloc`编译的时候，在`./configure`增加 `--host=arm-linux`选项

![redis_makefile](/images/redis_makefile.jpg)

![redis_configure](/images/redis_configure.jpg)

2.在`redis`下直接执行`makefile`，如果之前编译过，先执行`make distclean`在彻底清除之前编译的残留，出现以下内容，就是编译完成了

![redis_make](/images/redis_make.jpg)

3.查看编译出来的执行文件，比如`redis-cli` ，可以发现编译出来的`redis-cli`目标机器是`arm`的，交叉编译结束

![redis_check](/images/redis_check.jpg)
