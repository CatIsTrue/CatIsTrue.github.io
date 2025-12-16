---
title: linux插件系统加载器-使用dl库
date: 2019-09-01 11:21:39
tags:
  - C++
  - linux
categories: C++
---

## 背景

在 C/C++ 开发中，项目里需要在程序运行期间加载一个外部的代码库（而不是在编译时链接），而且根据不同的配置，需要加载不同的代码库，进行热更新，那直接考虑的就是插件架构。

操作系统提供的原生 API 是完全不同的，如果是在linux下，使用libdl库。一般使用这个库就是为了做插件系统加载器（可以说是它最主要的用途了）。


## 主要代码

```cpp
#include <dlfcn.h>

short loadLibDemo::realLoadLib(const string& libPath)
{
	if(libPath.length() <= 0)
		return -1;

	libHandle = dlopen(libPath.c_str(), RTLD_GLOBAL | RTLD_NOW);
	if(NULL==libHandle)
	{
		printf("dlopen fail %s\n",dlerror());
		return -1;
	}

	pOne = (PFUN_APIOne)dlsym(libHandle, "one_fun_api");
	if(NULL == pOne)
	{
		printf("[PFUN_APIOne] dlsym fail %s\n",dlerror());
		return -1;
	}

	pTwo = (PFUN_APITwo)dlsym(libHandle, "two_fun_api");
	if(NULL == pTwo)
	{
		printf("[PFUN_APITwo] dlsym fail %s\n",dlerror());
		return -1;
	}

	pThree = (PFUN_APIThree)dlsym(libHandle, "three_fun_api");
	if(NULL == pThree)
	{
		printf("[PFUN_APIThree] dlsym fail %s\n",dlerror());
		return -1;
	}

	pFour = (PFUN_APIFour)dlsym(libHandle, "four_fun_api");
	if(NULL == pFour)
	{
		printf("[PFUN_APIFour] dlsym fail %s\n",dlerror());
		return -1;
	}

	pFive = (PFUN_APIFive)dlsym(libHandle, "five_fun_api");
	if(NULL == pFive)
	{
		printf("[PFUN_APIFive] dlsym fail %s\n",dlerror());
		return -1;
	}

	pSix = (PFUN_APISix)dlsym(libHandle, "six_fun_api");
	if(NULL == pSix)
	{
		printf("[PFUN_APISix] dlsym fail %s\n",dlerror());
		return -1;
	}

	return 0;
}
```

完整代码已经放在了https://github.com/TreeAndFlower/loadlibdemo-linux

## 注意点

主要是，要对导出的动态库里的API，记得“extern c”

当没有extern c的时候，编译出来的动态库里API如下：

![默认是g++编译，所以有前缀后缀](/images/wu_extern_c.jpg)

当extern c添加了以后，编译出来的动态库API如下：

![相当于选择了gcc编译，没有多余的Z11前缀v后缀等内容](/images/you_extern_c.jpg)

PS:

这个示例代码，仅在linux平台下生效，因为链接的`dl`库，头文件是`#include <dlfcn.h>`，这些是linux平台的；
windows下用`Kernel32.lib`库，头文件是`Windows.h`；
