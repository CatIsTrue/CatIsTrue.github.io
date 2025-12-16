---
title: 将linux下源码文件，直接放在windows下vs编译的时候，关于undeclared identifier的奇怪报错
date: 2018-08-31 11:21:39
tags: [linux, 跨平台移植, windows]
categories: 跨平台移植
---


## 背景
我的一个工程里的源文件就叫他a.cpp,在linux下可以正常的编译，由于工程需要移植到windos下，所以我把a.cpp源文件移动到windows下的工程里，然后编译，编译的时候，发现有两个特别诡异的报错，报错内容如下：

![直接将cpp从Linux下挪到windows下报错](/images/copy_cpp_error.jpg)

更为奇怪的是，如果我在windows操作系统下，将a.cpp文件，用sublime.txt打开，然后再全选内容，拷贝到vs到工程里报错的a.cpp文件里，然后发现可以成功编译！

## 原因
经过一系列脑壳疼的排查，最终发现，这是因为，linux下和windows下的换行符不一样！！！
如果用sublime.txt把报错的源文件a.cpp的所有\n替换成windows下的\r\n,然后再保存文件，把保存修改的文件，挪到vs里直接编译，发现阔以了！！！

vs是不支持把linux下的换行符主动转成windows下的换行符的！！！巨坑

## 总结
后来，查看了网友将linux和windows换行符的区别，讲的特别好
链接：https://blog.csdn.net/stpeace/article/details/45767245

讲解的特别精彩的截图片段
![linux与windows换行符的区别](/images/huanhangfu.jpg)

## 吐槽
不得不说，像换行符这种巨坑的问题，排查起来简直特别特别坑……

