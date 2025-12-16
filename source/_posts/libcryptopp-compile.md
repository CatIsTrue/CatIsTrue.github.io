---
title: libcryptopp源码编译
date: 2022-11-06 16:00:20
tags: [linux, 交叉编译, libcryptopp]
categories: 技术折腾
---

# libcryptopp源码编译

## 详细步骤
1.下载源码，解压
2.修改GNUmakefile
因为是64位编译32位的库，还增加了-m32选项

```javascript

CXXFLAGS += -pipe -fPIC
CXXFLAGS += -m32

```

如下图：
![cryptopp_complie](/images/cryptopp_complie.jpg)

3.然后make

```javascript

make libcryptopp.a libcryptopp.so cryptest

```
![cryptopp_complie_2](/images/cryptopp_complie_2.jpg)

4.然后安装make install

```javascript

可以指定安装路径，如不想安装到默认路径
make install PREFIX=/usr/local/selfpath

```

![cryptopp_complie_3](/images/cryptopp_complie_3.jpg)

## 参考链接
https://www.lwlwq.com/post-cryptopp.html
