---
title: cjson库版本不一致，导致解析失败
tags:
  - cjson
  - 教程
categories: 技术折腾
abbrlink: ae350693
date: 2018-12-15 11:00:39
---

# cjson库版本不一致，导致解析失败

## 现象

在编译一个程序demo的时候，需要继承一个第三方库libexample.so，第三方库用到了cjson，本身这个程序也用到了cjson，由于两者用的cjson的版本不一致，导致json解析失败……

## 旧版本cjson
第三方库libexample.so使用的旧版本的cjson，cjson-types截图如下：
![cjson_old_version.jpg](/images/cjson_old_version.jpg)

## 新版本cjson
程序demo使用的是新版本的cjson，cjson-types截图如下：
![cjson_new_version.jpg](/images/cjson_new_version.jpg)

## 具体现象
用旧版本的cJSON源码编译到自己的代码里，编译出libexample.so库；
程序demo已经使用过新版本的cJSON源码，但是又连接了上面编译出来的libexample.so的库，再次进行json解析，会发现libexample.so里面解析cJSON_Number类型的节点的值会失败；
然后重新用新版本的cJSON源码编译出libexample.so库，再集成到上面的demo里面，即可解析成功。

## 分析
可以从上面两个不同版本的cjson源码截图的cjosn-types看出来：
这两个版本的cJSON Types的值不一样，比如cJSON_Number类型节点的值，旧版本的值是3， 新版本的值是8，
所以用旧版本编译的libexample.so库，集成到demo里的时候，解析到cJSON_Number节点的时候，错误的使用值8而不是3，所以导致解析失败

## 结论
代码里一定要保持一个版本的cjson；
版本混乱很容易造成奇怪的问题，而且这种问题往往还不容易排查！