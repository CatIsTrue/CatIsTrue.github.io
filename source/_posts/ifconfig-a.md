---
title: ifconfig指令配置网卡信息
date: 2019-11-29 11:21:39
tags: [linux, 配置网卡]
categories: 技术折腾
---

# ifconfig指令配置网卡信息

## 背景
linux/unix系统下，输入ifconfig指令，发现只有lo本地回环，没有网卡信息，为了能够正常上网，需要配置一下网卡信息

当输入指令

```javascript
ifconfig eth0 192.168.1.100
(可以自行设置ip地址）
```

会提示报错信息

```javascript
SIOCSIFADDR: No such device
eth0: ERROR while getting interface flags: No such device
```

此时，我们会发现设置网卡eth0失败了，提示我们没有这个网卡。那么，很有可能，我们的网卡不叫“eth0”这个名字，而不是我们真的没有网卡。

## ifconfig -a
输入指令

```javascript
ifconfig -a
```

此时，我们可以看到系统列出来我们所有的网卡名，如下图：

![ifconfig-a指令显示所有网卡](/images/ifconfig-a.jpg)

## 配置网卡的ip
接下来，我们发现自己系统的网卡果然没有eth0这个网卡名，但是有en0，所以，接下来我们设置来配置这个网卡的ip

```javascript
ifconfig en0 192.168.1.100
(可以自行设置ip地址）
```
然后回车后，就发现设置ip成功啦～

## 总结
一般情况下，因为我们常见的linux下的网卡名字都是eth0，所以习惯性的就直接配置eth0的ip，但是有的系统并不是用eth0当网卡名，所以还是要先输入ifconfig -a指令，来查看一下我们当前的系统下，网卡的名字都是什么，然后再去配置ip