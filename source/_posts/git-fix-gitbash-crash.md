title: (记录)解决git bash启动时报错的问题
date: 2015-02-05 15:09:19
categories: Git
tags: [git]
---

解决git bash启动时报"couldn't reserve space for cygwin's heap"的错误。

<!--more-->

# 错误描述

双击git bash后，打开的窗口一闪而过就自动关闭了。 

为了看清错误信息，执行cmd命令打开控制条，然后直接执行gitbash的启动命令

	D:\Users\sky.ao\AppData\Local\Programs\Git\bin\sh.exe --login -i

输出如下：


	Microsoft Windows [版本 6.1.7601]
	版权所有 (c) 2009 Microsoft Corporation。保留所有权利。
	
	D:\Users\sky.ao>D:\Users\sky.ao\AppData\Local\Programs\Git\bin\sh.exe --login -i

      0 [main] us 0 init_cheap: VirtualAlloc pointer is null, Win32 error 487
	AllocationBase 0x0, BaseAddress 0x68570000, RegionSize 0x140000, State 0x10000
	D:\Users\sky.ao\AppData\Local\Programs\Git\bin\sh.exe: *** Couldn't reserve spac
	e for cygwin's heap, Win32 error 0

# 解决方法

google了一下，找到解决方法，执行一下rebase命令：

	cd D:\Users\sky.ao\AppData\Local\Programs\Git\bin
	rebase.exe -b 0x50000000 msys-1.0.dll

问题解决。

PS：这是第二次遇到这个错误，第一次忘了怎么解决了，这次决定记录下来备忘，估计以后还得遇到。