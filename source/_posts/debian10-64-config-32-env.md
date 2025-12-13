---
title: debian10系统原本为64位系统，如何配置32位编译and运行环境
tags:
  - linux
  - 教程
  - debian
categories: 技术折腾
abbrlink: 71d17bfd
date: 2020-04-25 10:21:39
---

# debian10系统原本为64位系统，如何配置32位编译and运行环境

## 背景

原本安装的是debian10的64位系统，但是因为有些第三方程序是32位的，需要在这个系统上编译运行，那么需要配置一下必要的环境

## 当32位程序放在debian64位系统里无法编译or运行时，如何判断是因为没有配置32位编译or运行环境

一般来说，编译32位程序时，如果出现以下报错,可以认为没有配置32位编译or运行环境

```javascript
fatal error: bits/libc-header-start.h: No such file or directory
```

运行32位程序时，如果出现以下报错，也可以认为没有配置32位编译or运行环境

```javascript
//明明有执行程序test（32位的），但是执行./test的时候却报错
test： 
     no such file or directory
```

出现以上两种情况的任何一个时，一般可以判断时由于64位debian系统下，没有配置32位程序的编译or运行环境，需要执行以下两个步骤

## 解决方案

### 一、执行sudo apt-get install gcc-multilib

安装gcc-multilib

```javascript
//安装gcc-multilib
sudo apt-get install gcc-multilib
```

### 二、执行sudo apt-get install g++-multilib

安装g++-multilib

```javascript
//安装g++-multilib
sudo apt-get install g++-multilib
```

## 总结

因为从官网直接下载下来的debian10的64位安装镜像在安装完成后，原始的debian系统是不支持32位程序运行的，所以需要对环境进行配置，所以做个记录，免得下次忘记了