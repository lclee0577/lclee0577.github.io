---
title: STL与泛型编程
date: 2020-12-03 08:48:24
tags:
categories: C++
---
# 资源
https://www.bilibili.com/video/av48068999?p=1&t=18

# P1.认识heades、版本、重要资源
`Generic Programming`(泛型编程,GP)就是使用`template`(模板)为主要工具来编写程序，STL是泛型编程最成功的作品
查询网站 [cppReference](https://en.cppreference.com/w/), [CPlusPlus](https://cplusplus.com), [gcc.gun](https://gcc.gnu.org)

# P2.STL体系结构基础介绍
## STL 六大部件(Component)
**1.容器  （Containers）**
**2.分配  （Allocators）**
**3.算法  （Algorithms）**
**4.迭代器（Iterators）**
**5.适配器（Adapters）**
**5.仿函数（functors）**

```cpp {.line-numbers}
#include <vector>
#include <algorithm>
#include <functional>
#include <iostream>

using namespace std;

int main()
{
    int ia[6] = {27, 210, 12, 47, 109, 83};
    vector<int, allocator<int>> vi(ia,ia+6);//vector-容器，allocator-分配器
    cout<< counter_if(vi.begin(),vi.end(),//begin,end - 迭代器，counter_if-算法
                        not1(bind2nd(less<int>,40)));//not1,bind2nd - 算法适配器，less - 仿函数
    return 0;
}
```
- 11行：用数组数据创建容器
- 12行：输出数组中所有符合条件的数
- 13行：`bind2nd(less<int>,40)` 意思是`return (num_in < 40)`将小于号的第二参数绑定为20，加上之前的`not1`去反，意为统计所有大于40的数。
---
- `复杂度，Complexity，Big-oh` :不同的算法，不同的数据结构有不同的复杂度，要根据实际情况选择
- 前闭后开区间：容器都是前闭后开区间，`vector.end()`不是容器内的元素，是最后一个的下一个，不能进行`*(c.end())`操作。
    ```cpp
    //通常用法
    Container<T> c;
    ···
    Container<t>::iterator ite = c.begin()
    for(; ite != c.end(); ++ite){···}`
    ```
- 更推荐range-base for (C++11) 语法糖 方便`for`循环 语法如下

    ```cpp
    for(decl:coll)//decl-声明，coll-容器，集合。将容器中的元素一个个取出来 coll：collection
    {
        statement
    }
    ```

    如下面的例子

    ```cpp
    for(int i :{2,3,4,5,6})// {···} 为c++11中新增的容器类型
    {
        cout << i << endl;
    }

    vector<double> vec
    ···//给容器添加一些值
    for(auto elem:vec)
    {
        cout << elem << endl; //传值 pass by value
    }

    for(auto& elem:vec)
    {
        elem *= 3; //传引用 pass by reference 修改容器内部值
    }  
    ```
    