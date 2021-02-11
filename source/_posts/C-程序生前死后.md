---
title: C++ 程序生前死后
date: 2021-02-09 19:35:57
tags:
categories:
---
解密 CRT Startup code
(CRT: C run time library

# 1. Startup code

- 启动码函数比main()函数更早执行，其中会初始化一些静态变量，再调用main()函数，一般是由链接器完成，不推荐个人编写

# 2. heap allocate

- 在启动代码中申请一大块内存，用一定的结构进行不同大小的内存管理

# 3. before main

- 在进入main 之前，还处理了命令行参数和系统的环境变量

- io init 建立stdin，stdout,stderr