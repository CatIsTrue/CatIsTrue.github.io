---
title: linux下安装tcpdump并用其抓包
date: 2020-05-16 11:21:39
tags: [linux, tcpdump, 抓包]
categories: 技术折腾
---

# linux下安装tcpdump并用其抓包

## 背景
有时候需要分析网络协议，这时候抓包看看直接的数据，能够有助于对协议有一个直观的感受，在windows下，可以直接安装Wireshark就能轻松抓包分析了，但是在linux下，没有Wireshark，所以可以安装tcpdump，用tcpdump抓包分析

## 安装tcpdump

安装tcpdump有两种方式，一种是下载tcpdump源码，然后编译安装；另一种是直接用系统安装命令

### 下载源码安装tcpdump

这个可以参考：https://blog.csdn.net/tic_yx/article/details/17012317
这篇文章里记录的很详细
文章内容截图如下：
![源码编译tcpdump](/images/complie_tcpdump.jpg)

### 直接用系统安装命令

在ubuntu下，可以直接使用sudo apt-get install tcpdump
如果这一步安装的时候有报错，可以更新一下下载源，国内清华的下载源还是很好用的

## 用tcpdump抓包

sudo tcpdump -i 网卡 -entXX
![使用tcpdump抓包](/images/use_tcpdump.jpg)

也可以把tcpdump抓到的数据保存到文件里
sudo tcpdump -i 网卡 -entXX -w 文件名.pcap
![tcpdump抓包后保存到文件里](/images/tcpdump_save_file.jpg)
然后将pcap文件从linux里拷贝到windows下，用Wireshark分析数据