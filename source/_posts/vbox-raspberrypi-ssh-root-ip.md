---
title: 树莓派配置基础环境-ssh-root-静态ip
date: 2020-11-15 11:21:39
tags:  [raspberrypi,交叉编译]
categories: 交叉编译
---

# 树莓派配置基础环境-ssh-root-静态ip

## 配置root用户

在终端进行如下操作即可：
```bash
sudo passwd root
```
然后根据提示输入root用户的密码
再重复输入一次刚刚设置的密码

切换root用户操作如下即可：
```bash
su -
```
输入设置的root用户的密码

## 配置开启ssh
操作如下即可：
```bash
sudo vim /etc/ssh/sshd-config
```
修改 `PermitRootLogin yes`
然后保存修改
然后执行`sudo systemctl restart ssh`
再通过`ss -tnl`查看是否开启成功即可

## 配置静态ip如下
在需要无线网络连接的情况下，配置eth0的静态ip如下：
```bash
sudo vim /etc/network/interfaces
```

然后再文件里增加以下内容
```text
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
address 192.168.0.1 //IP地址
netmask 255.255.255.0 //掩码
```

然后保存
但是此时，如果不设置一下`wlan0`，那么会发现虽然静态ip设置成功了，但是树莓派却无法联网了
所以还要在`interfaces`文件里追加以下内容
```text
auto wlan0
iface wlan0 inet dhcp
wpa_conf /etc/wpa_supplicant/wpa_supplicant.conf
```
然后保存
退出文件
重启树莓派即可

## 如图，配置动态IP
![shu_ip_dhpc](/images/shu_ip_dhcp.jpg)

## 如图，配置静态IP
![shu_ip_static](/images/shu_ip_static.jpg)
