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
**6.仿函数（functors）**
下面的代码片段展示了这六大部件
```cpp {.line-numbers}
#include <vector>
#include <algorithm>
#include <functional>
#include <iostream>

using namespace std;

int main()
{
    int ia[6] = {27, 210, 12, 47, 109, 83};
    vector<int, allocator<int>> vi(ia,ia+6); //vector-容器，allocator-分配器
    cout<< counter_if(vi.begin(),vi.end(),   //begin,end - 迭代器，counter_if-算法
                        not1(bind2nd(less<int>,40)));//not1,bind2nd - 算法适配器，less - 仿函数
    return 0;
}
```
- 11行：用数组数据创建容器
- 12行：输出数组中所有符合条件的数
- 13行：`bind2nd(less<int>,40)` 意思是`return (num_in < 40)`将小于号的第二参数绑定为20，加上之前的`not1`取反，意为统计所有大于40的数。
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
# P3.容器之分类与各种测试（一）
- Sequence Containers - 序列式容器 : **Array** (fixed number), Vector，Deque，List，**Forward-List**
    - array是把语言的数组包装成class
    - List双向链表:内部有向前、向后两根指针；Forward-List单项链表:内部只有一个指针。单向链表占用空间更少
- Associative Containers - 关联式容器：Set/MultiSet，Map/MultiMap，
    - 内部是用红黑树实现，自平衡，避免某一侧过长
    - set 只有value，map有key和value
    - MultiMap 和 MultiSet 里面内容可以重复
- **Unordered Containers** - 不定序容器(底层是HashTable)：Unordered Set/MultiSet，Unordered Map/MultiMap
    - hashtable 常见做法 Separate Chaining，里面放链表
- (加粗的是C++11新增特性)，
## 测试 Array
1. 创建一个长度为`ASIZE`的数组，随机放入0-65535之间数值，输出所需时间，数组大小，首尾数据，和数据地址。对应`3 ~ 14 行`
2. 输入一个想要查找的数。查找前先对数组做快速排序(qsort),再输出上一步输出的那些信息。对应`16 ~ 28 行`
3. 对排序好的数组进行二分查找，输出消耗时间和结果。对应`30-37行`

```cpp {.line-numbers}
void test_array()
{
    cout << "\ntest_array()......... \n";
    array<long, ASIZE> c;//const long ASIZE = 1000000;
    clock_t timeStart = clock();
    for (long i = 0; i < ASIZE; ++i) {
        c[i] = rand() % 65535;
    }

    cout << "milli-seconds:" << (clock() - timeStart) << endl;
    cout << "array.size()= " << c.size() << endl;
    cout << "array.front()= " << c.front() << endl;
    cout << "array.back()= " << c.back() << endl;
    cout << "array.data()= " << c.data() << endl;

    long target = get_a_target_long();//输入一个数
    timeStart = clock();
    cout << "-------------------before qsort: --------------" << endl;

    qsort(c.data(), ASIZE, sizeof(long), compareLongs);
    
    cout << "qsort(), milli-seconds: " << (clock() - timeStart) << endl;
    cout << "---------------------after qsort:------------------ " << endl;

    cout << "array.size()= " << c.size() << endl;
    cout << "array.front()= " << c.front() << endl;
    cout << "array.back()= " << c.back() << endl;
    cout << "array.data()= " << c.data() << endl;

    timeStart = clock();
    long* pItem = (long*)bsearch(&target, (c.data()), ASIZE, sizeof(long), compareLongs);
    cout << "bsearch(), milli-seconds: " << (clock() - timeStart) << endl;
    
    if (pItem != NULL)
        cout << "found, " << *pItem << endl;
    else
        cout << "not found!" << endl;
}
```

# P4.容器之分类与各种测试(二)
## 使用容器vector
- 使用.push_back()从后端放入数据
- vector 空间总是2倍增长，如放入入5个元素时，vector会先拓展成8个，再放入第五个。因此容量总是≥元素个数
- 当空间装不下要容纳的数据的时会自动拓展，并将原来的数据拷贝到新的空间，然后释放旧的空间
- vector.size() - 元素个数， vector.capacity() - 容量
- `12-19行`使用`try-catch`因为可能内存分配失败。
- `31-40行`使用`::find()`直接遍历查找；`43-53行`使用sort先排序再二分查找
- 排序通常很耗时间，有时直接查找更快。
```cpp {.line-numbers}
namespace jj02
{
void test_vector(long& totalCount)
{
    cout << "\ntest_vector()......... \n";

    vector<string> c;
    char buf[10];
    clock_t timeStart = clock();
    for (long i = 0; i < totalCount; ++i) 
    {
        try {
        snprintf(buf, 10, "%d", rand() % 65535);
        c.push_back(string(buf));
        } catch(std::exception& e) {

        cout << "i=" << i << e.what() << endl;
        // 曾经最高 i=58389486 then std::bad_alloc
        abort();
        }
    }

    cout << "milli-seconds:" << (clock() - timeStart) << endl;
    cout << "vector.size()= " << c.size() << endl;
    cout << "vector.front()= " << c.front() << endl;
    cout << "vector.back()= " << c.back() << endl;
    cout << "vector.data()= " << c.data() << endl;
    cout << "vector.capacity()= " << c.capacity() << endl;

    string target = get_a_target_string();
    {
    timeStart = clock();
    auto pItem = ::find(c.begin(), c.end(), target);
    cout << "::find(), mill-seconds: " << (clock()-timeStart) << endl;

    if (pItem != c.end())
        cout << "found, " << *pItem << endl;
    else
        cout << "not found! " << endl;
    }	

    {
    timeStart = clock();
    
    sort(c.begin(), c.end());
    
    string* pItem = (string*)bsearch(&target, (c.data()), c.size(), sizeof(string), compareStrings);
    cout << "sort()+bsearch(), milli-seconds: " << (clock() - timeStart) << endl;
    
    if (pItem != NULL)
        cout << "found, " << *pItem << endl;
    else
        cout << "not found!" << endl;
    }
}
}
```

# P5.容器之分类与各种测试(三)
## 使用容器 list
- list有最大容量，可以调用`list.max_size()`得到（不同的电脑不同）
- list自带成员函数sort()（class内自带的`sort`通常比全局的`sort`效率高）
- 具体测试程序与vector类似

## 使用容器 forward_list
- forward_list 只允许前端放入 .push_front(),查看前端 forward_list.front()
- 不提供查询大小和末位数据。(没有forward_list.back(), forward_list.size())
- forward_list含有sort()
- gnu中还有 `slist` 用法与其相同(gnu 很早之前在拓展库中加入slist，C++11才加入forward_list)

## 使用容器 deque
- deque 可以向两边扩充，但是在底层是分段(buffer)连续的，通过操作符重载来判断是否到达 buffer 的边界并进入到下一个 buffer。
- 每次扩充一个 buffer (不同于vector每次翻倍，list每次增加一个)
- deque 没有sort函数，需要调用 `::sort()`
- deque 可以双向进出
- 容器 `stack` 和 `queue` 也是调用 `deque`实现的，因此也有人将`stack` 和 `queue` 成为 `Container Adapters` 容器适配器