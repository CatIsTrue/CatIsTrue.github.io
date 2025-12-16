---
title: linux下查看编译的静态库和动态库是32位还是64位
date: 2019-12-15 11:21:39
tags: [linux, 常用命令, 交叉编译]
categories: 常用命令
---

## 背景
有时候可能会需要交叉编译，所以需要知道平台上编译出来的版本到底是64位还是32位

## file指令查看动态库是32位还是64位
如图：file libcurl.so 查看当前编译的libcurl.so是32位还是64位的
![file_so.jpg](/images/file_so.jpg)

## objdump -a指令查看静态库是32位还是64位的
如图：objdump -a libtest.a 查看当前编译的静态库libtest.a是32位还是64位的
![objdupm_a.jpg](/images/objdump_a.jpg)

## readelf -h指令查看静态库or动态库是32位or64位，及编译平台运行平台等信息
如图：readelf -h libssl.so 查看编译的动态库lib
Class字段显示当前库是32位or64位
Machine字段显示当前库运行的目标机器系统

![readelf_h.jpg](/images/readelf_h.jpg)