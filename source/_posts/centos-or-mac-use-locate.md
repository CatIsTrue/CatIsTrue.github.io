---
title: centos或mac下使用locate指令时，报错
tags:
  - linux
  - 教程
categories: 技术折腾
abbrlink: 988cb25a
date: 2019-08-10 11:21:39
---

# centos或mac下使用locate指令时，报错(/var/db/locate.database)

centos或mac下使用locate指令时，出现报错信息The locate database (/var/db/locate.database)

## 在centos系统下，使用locate指令报错
使用locate指令时，出现报错信息’var/lib/mlocate/mlocate.db’:No such file or directory时，处理方案如下：

```javascript
//输入以下指令即可
updatedb
//需要等待一段时间，因为生成数据库需要一段时间
```

稍等一段时间之后，再输入locate指令，即可发现可以使用了

## 在mac系统下，使用locate指令报错
当在mac上使用locate指令时，报错如下：
![使用locate指令时报错](/images/locate_01.jpg)

解决方案如下：

1. 先根据刚才的提醒，输入sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.locate.plist
![根据提示，输入launchtcl指令](/images/locate_02.jpg)


2. 然后，输入指令sudo /usr/libexec/locate.updatedb
![生成数据库](/images/locate_03.jpg)

输入上面这个指令后，会等待好久一段时间，要稍微等待一会儿

3. 最后再次输入locate指令，发现locate指令已经生效啦
![locate指令已经生效啦](/images/locate_04.jpg)

## 参考资料
因为我之前经常在ubuntu下，都没有碰到过locate指令不好用的情况，最近需要在centos和mac下操作，忽然发现居然loacte指令使用失效，于是上网查找了解决办法，经过尝试了一些方案后，终于在centos和mac下可以使用locate指令了，因此做出了以上的总结～

感谢网友们无私分享的解决方案～
参考的网友方案：https://www.jianshu.com/p/d8f4f9e4b58c