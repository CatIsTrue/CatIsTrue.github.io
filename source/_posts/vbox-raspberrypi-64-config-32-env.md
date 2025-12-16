---
title: 树莓派64位系统配置32位运行环境
date: 2020-11-15 11:21:39
tags:  [raspberrypi,交叉编译]
categories: 交叉编译
---

# 树莓派64位系统配置32位运行环境

## 配置libc6:armhf

树莓派本身安装了64位系统的情况下，需要配置32位程序的运行环境，首先安装依赖库，操作步骤如下，以下操作都要在`root`用户下进行
![4b_armhf](/images/4b_armhf.jpg)

## 配置32位程序的依赖库环境

```bash
git clone git://github.com/raspberrypi/tools.git
```

将`arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/`路径下的`lib`库都拷贝到`/usr/local/lib32`路径下，这个路径可以自己创建，自定义路径名，专门用来存放底层32位依赖库(如libstdc++.so.6)

## 总结
其实最重要的，就是从官方提供的交叉编译工具链，把32位库给获取到，然后放到树莓派的自定义路径下，之后自己编译的32位执行程序/库，都要指定链接这个路径的基础库，否则会报错（找不到依赖库）

本身64位的树莓派系统是不带32位基础库的，所以必须从官方的交叉编译工具链里获取（目前，无法通过`apt-get`直接获取到所有的32位依赖库）

## 参考资料
https://raspberrypi.club/148.html