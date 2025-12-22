---
title: openvpn交叉编译
date: 2020-04-11 11:21:39
tags: [第三方库,交叉编译]
categories: 交叉编译
---

# openvpn交叉编译

# 需要源码交叉编译openvpn的操作步骤

## 背景
因为某些原因，只能选择交叉编译，当前平台是64位的，但是目标平台是32位的，目标平台又禁止了apt-get等相关操作，所以必须要源码交叉编译openvpn及其相关依赖库。因为之前交叉编译的时候，大多第三方库都是有现成的CMaKeLists.txt或者是Makefile，所以编译起来倒是也方便，但是，openvpn它只有configure文件，它是需要执行./configure成功设置好各种环境变量配置以后，才能生成Makefile的。于是，一开始踩了很多坑，在此做一个记录……

## 一、预先配置的环境变量

export CC=指定交叉编译时gcc编译版本
export CXX=指定交叉编译时g++编译版本

## 二、编译openssl

完整编译选项配置如下：
`setarch i386 ./config -m32 –prefix=/home/muhan/openssl/ –openssldir=/home/muhan/openssl/ -Wl,-rpath,/usr/local/openssl/lib shared`

详细选项含义如下：
配置-m32 指定编译32位的库
配置–prefix 指定openssl的安装目录
配置–openssldir 指定openssl的目录
配置shared关键字 指定编译时生成动态库（libssl.so/libcrypto.so及其相关软连接）

然后再make && make install 即可

## 三、编译安装lzo

完整编译选项配置如下：
`./configure –enable-shared –prefix=/home/muhan/lzo/usr –with-sysroot=指定交叉编译时的sysroot的路径名`


详细选项含义如下：
配置–enable-shared 指定编译时生成动态库（liblzo2.so及其相关软连接）
配置–prefix 指定lzo安装路径
配置–with-sysroot 指定交叉编译时的sysroot

然后再make && make install 即可

## 四、编译安装openvpn

完整编译选项配置如下：
`./configure CC=指定交叉编译时gcc的那个版本 –prefix=/home/muhan/openvpn/ LZO_CFLAGS=”-I/home/muhan/lzo/usr/include” LZO_LIBS=”-L/home/muhan/lzo/usr/lib -llzo2” OPENSSL_CFLAGS=”-I/home/muhan/openssl/include” OPENSSL_LIBS=”-L/home/muhan/openssl/lib -lssl -lcrypto” –disable-plugin-auth-pam`

详细选项含义如下：
配置CC的版本
配置–prefix 指定openvpn安装目录
配置LZO_CFLAGS 指定lzo的头文件路径
配置LZO_LIBS 指定lzo的库文件路径及所链接的库
配置OPENSSL_CFLAGS 指定openssl的头文件路径
配置OPENSSL_LIBS 指定openssl的库文件路径及所链接的库
配置–disable-plugin-auth-pam 禁掉pam模块

然后再make && make install 即可

## 总结

当编译第三方库的时候，如果没有CMakeLists.txt，也没有Makefile，只有config或者configure，那么需要先了解编译选项

执行./config –help 或者 ./configure –help 来查看当前支持的编译选项
然后根据提示配置一下我们需要指定的选项，比如自己指定的openssl的版本的库和头文件路径名，比如CC的版本，比如安装路径等等
(当然，如果不需要额外配置这些东西的话，直接走默认配置的话，那么直接执行./config 或者 ./configure 就行)

然后在生成Makefile之后，再make && make install 即可