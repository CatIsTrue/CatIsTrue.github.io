---
title: 虚拟机安装官方树莓派系统，配置交叉编译链
date: 2020-11-15 11:21:39
tags:  [raspberrypi,交叉编译]
categories: 交叉编译
---

# 虚拟机安装官方树莓派系统，配置交叉编译链

## 1.下载官方镜像

https://www.raspberrypi.org/software/raspberry-pi-desktop/
通过官网，下载raspberry镜像64位系统iso文件之后，可以安装在虚拟机vbox/vmware里

## 2.下载交叉编译工具链

通过github下载最新的官方树莓派交叉编译工具链
git clone git://github.com/raspberrypi/tools.git

## 3.配置交叉编译工具链

将arm-bcm2708文件夹拷贝到/opt/arm-bcm2708下（自定义路径即可）

将上面到交叉编译工具链的路径配置到~/.bashrc文件

```javascript
sudo vim ~/.bashrc
export PATH=$PATH:/opt/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin
```

## 4.测试交叉编译工具链是否安装成功

输入以下指令，如果有打印一些版本信息，那么说明交叉编译环境配置正确
arm-linux-gnueabihf-gcc -v
如图:
![4b_succeed](/images/4b_succeed.jpg)

## 5.假如第4步的时候有报错没有这个文件或目录，但实际是有这个文件的，可以apt-get安装libc6-dev-i386

如果报错提醒如下：
![4b_fail_env](/images/4b_fail_env.jpg)

可以按照以下的解决方案尝试一下：
![4b_deal_fail](/images/4b_deal_fail.jpg)

然后重新输入arm-linux-gnueabihf-gcc -v即可发现打印版本信息

## 6.编写测试demo，然后编译生成执行文件

demo略，编译的指令如下：
![4b_hello_world](/images/4b_hello_world.jpg)

需要注意的是，该可运行文件不能在PC机上运行，只能在树莓派arm板子上运行

## 总结

到此为止，虚拟机上的树莓派arm的交叉编译工具链搭建完成,这个官方的交叉编译工具链还是很靠谱的

## 参考资料

下载镜像参考以下网址
https://www.jianshu.com/p/1a65cb0b8f58
下载安装交叉编译链参考以下网址
https://www.cnblogs.com/zfyouxi/p/3831769.html