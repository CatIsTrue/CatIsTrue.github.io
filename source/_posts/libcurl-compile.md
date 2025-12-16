---
title: libcurl交叉编译
date: 2020-04-25 11:21:39
tags: [linux, 交叉编译, libcurl]
categories: 交叉编译
---

# libcurl交叉编译

# 需要源码交叉编译libcurl操作步骤

## 背景
有时候需要交叉编译libcurl，比如目标机器是32位系统的，但是本地机器是64位系统的，而且由于某些原因，我们无法在32位系统上直接编译，所以需要用到交叉编译

## 编译openssl

libcurl是依赖openssl的，所以先编译openssl的32位库

完整编译选项配置如下：
```bash
setarch i386 ./config -m32 –prefix=/home/muhan/openssl/ –openssldir=/home/muhan/openssl/ -Wl,-rpath,/usr/local/openssl/lib shared
```
详细选项含义如下：
```text
配置-m32 指定编译32位的库
配置–prefix 指定openssl的安装目录
配置–openssldir 指定openssl的头文件目录
配置shared关键字 指定编译时生成动态库（libssl.so/libcrypto.so及其相关软连接）
```

然后再make && make install 即可

## 编译安装zlib

有时候有的系统是默认安装了32位zlib库的，那么就可以跳过这一步，但是有的系统需要自己下载编译zlib-32位库

完整编译选项配置如下：
直接修改CMakeLists.txt文件，增加以下两行
```cmake
set(CMAKE_C_FLAGS “-m32”)
set(CMAKE_CXX_FLAGS “-m32”)
```
详细选项含义如下：
```text
配置CMAKE_C_FLAGS 指定编译32位库环境
配置CMAKE_CXX_FLAGS 指定编译32位库环境
```

然后再mkdir build && cd build && cmake .. && make && make install 即可

## 编译安装libcurl

最后就是编译libcurl

完整编译选项配置如下：
```bash
./configure PKG_CONFIG_PATH=/home/muhan/openssl CFLAGS=”-m32” CPPFLAGS=”-I/home/muhan/openssl/include” LDFLAGS=”-L/home/muhan/openssl/lib”
```

详细选项含义如下：
```text
配置PKG_CONFIG_PATH 指定启动openssl选项(启动这个选项，就会默认链接lssl，lcrypto，lz三个库)
配置CFLAGS 指定编译32位库环境
配置CPPFLAGS 指定链接的库的头文件
配置LDFLAGS 指定链接的库的路径
```

然后再make && make install 即可

## 总结

当编译第三方库的时候，如果有CMakeLists.txt，直接用CMakeLists.txt编译就很方便；
如果只有configure，那么需要先了解编译选项

执行./configure –help 来查看当前支持的编译选项
然后根据提示配置一下我们需要指定的选项，比如自己指定的openssl的版本的库和头文件路径名，比如CC的版本，比如安装路径等等
(当然，如果不需要额外配置这些东西的话，直接走默认配置的话，那么直接执行./config 或者 ./configure 就行)

然后在生成Makefile之后，再make && make install 即可