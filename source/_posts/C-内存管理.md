---
title: C++内存管理
date: 2021-02-11 13:02:58
tags:
categories:
---

# P1. overview

- 第一讲 primitives 基础工具
- 第二讲 malloc/free
- 第三讲 std::allocator
- 第四讲 other allocator
- 第五讲 loki::allocator

# P2. 内存分配的每一层面

1. C++ Application

2. C++ Library std::allocator

3. C++ primitives new,new[], new(),::operator new()

4. CRT malloc/free

5. OS.API  HeapAlloc, VirtualAlloc

- 编程一般操作1-4，若调用系统API就跟系统绑定在一起，不具备移植性

# P3.四个层面基本用法

分配                        | 释放                      | 属类      | 可否重载   |
---------                   |----------                 |-----------|---------
 malloc()                   | free()                    | C函数     |不可
 new                        | delete                    | C++表达式 |不可
 ::operator new()           | operator delete()         | C++函数   |可
 allocator\<T>::allocate()  |allocator\<T>::deallocate()| C++标准库|可自由设计并搭配容器

```cpp
void* p1 = malloc(512);//512 bytes
free(p1);

complex<int>* p2 new complex<int>;
delete P2

void p3 :: operator new(512);//512 bytes
::operator delete(p3);

void* p4 = allocator<int>().allocate(7);// 分配7个int
allocator<int>().deallocate((int*)p4,7)

```
