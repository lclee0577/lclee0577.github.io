---
title: STL与泛型编程
date: 2020-12-03 08:48:24
tags:
categories: C++
toc: true
---

# 资源

<https://www.bilibili.com/video/av48068999?p=1&t=18>

# P1.认识heades、版本、重要资源

`Generic Programming`(泛型编程,GP)就是使用`template`(模板)为主要工具来编写程序，STL是泛型编程最成功的作品
查询网站 [cppReference](https://en.cppreference.com/w/), [CPlusPlus](https://cplusplus.com), [gcc.gnu](https://gcc.gnu.org)

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
- (加粗的是C++11新增特性)

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
- `stack` 先进后出 和 `queue` 先进先出 的特性，因此不提供 `iterator`，也无法查找和排序。

# P6.容器之分类与各种测试(四)

## 使用容器 multiSet

- 调用 `insert()`放入数据，红黑树会找到其合适的位置。
- 数据在放入的时候已经做好排序,插入的时候慢一些
- 类内部的find()函数远远快于全局的::find()

## 使用容器multiMap

- `multimap<long,string> c;` 初始化 key 和 value 的类型
- `c.insert(pair<long,string>(i,buf))`, 要插入`pair`类型数据
- multimap 不可使用[] 做 insertion

## 使用容器 unordered_multimap

- 在以前的`gnu c`中提供的名称叫做`hash_multimap`（底层由散列表实现）
- bucket 一定比 element 多。一旦元素数量达到 bucket 数量时，就会重新申请大一倍的空间，将元素重新打散，放到篮子里
- 不一定每一个bucket里都有元素，有的bucket有多个，有的为空。（但是 bucket 中存放的链不能太长）

## 使用容器set，map

- 底层由红黑树实现
- 放入key不能重复
- 对set来说，key和value相同
- map 可以使用 `c[i] = string(buf)` 进行插入，内部会将`i，buf`组成一个 pair 类型数据

## 同样还有 unordered_set 和 unordered_map

- 底层由hash table实现，特性与set和map相同

# P7.分配器之测试

- 容器背后需要 allocator 来支持对内存的使用，容器有默认的分配器，通常我们不需要关心。具体的内存管理在另一门课详细介绍
- allocator 负责内存的申请与释放，不推荐个人使用。可以直接使用 `new` 和 `delete`

```cpp
template<typename _Tp, typename _Alloc = std::allocator<_Tp>>
    class vector:protected_Vector_base<_Tp, _Alloc>
```

# P8.源码分布(vc,gcc)

- vc 的源码文件在`\include`, gnu c++ 在 `\4.9.2\include`

# P9.OOP(面向对象编程) vs. GP(泛型编程)

- **OOP** 企图将 `datas` 和 `methods` 关联在一起，例如 `list.sort()`

- **GP** 却是将 `datas` 与 `methods` 分开来,  `sort(vec.begin(), vec.end())` 而不是 `vector.sort()`

  - 采用 GP 的优点： `Containers` 和 `Algorithms` 团队可以各自工作，通过 `Iterators` 沟通

- 为什么 `list` 不能用 `::sort` ? 全局函数`::sort` 对迭代器的类型有一定的要求，如实现指针+长度的地址跳转，而链表是通过指针相连，在内存中的地址并不连续，无法满足使用 `::sort` 的条件因此需要使用class内部自带的 `sort()` 。

- 所有 `Algorithms`，器内最终涉及元素本身的操作，无非就是比大小。

# P10.技术基础：操作符重载 and 模板（泛化，全特化，偏特化）

- 不是所有的操作符都能重载，需要重载之前最好先去查一下reference。（像一般的加减乘除大小，都是可以重载的，对于特殊的符号要去查）

- 函数模板
  
  ```cpp
  class stone
  {
      public:
        stone(int w, int h, int we) : _w(w), _h(h), _weight(we) {}

        bool operator<(const stone& rhs) const
        { return _weight < rhs._weight; }

      private:
        int _w, _h, _weight; 
  };

  template <class T>
  inline
  const T& min(const T& a, const T& b)
  { return b < a : b , a; }

  stone r1(2,3,6), r2(3,3,9), r3；
  r3 = min(r1, r2);
  ```

- 当调用 `min(r1, r2)` 时，编译器会对 `function template` 进行实参推导 ( argument deduction )，结果可知 `T` 的类型为 `stone`，于是调用 `stone::operator<()`

- 空心的 `template<>` 就是模板特化，具体特化的例子详见C++笔记

- 偏特化（部分特化）特化就是将`template<···>`尖括号中的参数指定，减少泛化的参数。（个数和范围的特化） 具体特化的例子详见C++笔记

# P11.分配器

- `operator new()` 底层都是调用 `malloc()` 来实现

- `malloc()` 分配的内从往往比我们需要的多，还有一些对齐，管理的额外开销(当我们存入的元素很小时，开销可能比元素更占空间)

- 分配器的底层都是调用 `operator new()`

- vc6，gnu c2.9 bc++ 标准库中的 `allocator` 没有任何独特设计，就是调用 `malloc()， free（）`来分配和释放内存。

- 分配器释放内存时，不仅仅要传递指针，还要传递大小，因此不适合我们去使用它，但是却有利于容器来使用（后面讲解）

- 由于标准库中的 `allocator` 会带来巨大的额外开销，gnu c2.9中并没有采用它来实现容器，而是使用了自己定义的分配器 `alloc`

- 由于容器中的元素大小都是一样的，因此不必为每一个元素都添加开销（也就是说不是每个元素都要去调用 `malloc` 申请内存），gnu c2.9 中的 `alloc` 向系统申请整块大内存，在分割成不同大小，给容器使用，来减小额外开销。（内部具体特性在内存管理课程讲解）

- 处于某种原因在gnu c4.9中没有使用高效的alloc作为容器的默认分配器，但是我们仍然可以使用它，它被改名为`__pool_alloc`, 例如`vector<string,__gnu_cxx::__pool_alloc<string>> vec;`

# P12.容器之间的实现关系与分类(1)

- 简单回顾容器详见P3。
- 在不同版本下容器的大小sizeof()不同

# P13.深度搜索list (1) 上

- `operator++()` 是 `prefix form` 前置型 例如 ++i， `operator++(int)` 是 `postfix form` 后置类型，例如i++

- list's iterator gnu gcc 2.9版本

    ```cpp {.line-numbers}
    template<class T>
    struct __list_node{
        typedef void* void_pointer;
        void_pointer prev;
        void_pointer next;
        T data;
    };

    template<class T, class Ref, class Ptr>
    struct __list_iterator{
        typedef __list_iterator<T, Ref, Ptr> self;
        typedef bidirectional_iterator_tag iterator_category;//(1)
        typedef T value_type;                                //(2)
        typedef Ptr pointer;                                 //(3)
        typedef Ref reference;                               //(4)
        typedef __list_node<T>* link_type;
        typedef ptrdiff_t difference_type;                   //(5)

        link_type node;

        reference operator*() const{return (*node).data;}
        pointer operator->() const{return &(operator*());}
        self& operator++() {node = (link_type)((*node).next); return *this;}
        self operator++(int){self tmp = *this; ++*this, return tmp;}
        ···
    }
    ```

- 几乎所有的容器迭代器要进行(1)-(5)的 `typedef`

- `25`行 `self temp = *this` *不是操作符重载，这里是变量声明，解释为构造函数的参数;以及后面的 `++*this`也不是，`*this`是重载操作符`operator++()`的参数

- 前置++和后置++ 返回类型不同是因为要与 int 的操作保持一致。int 不允许 `(i++)++` 但可以 `++(++i)`

- 重载 `operator*()` 是为了取出数据 `21`行

# P14.深度搜索list (1) 下

- 在 gcc4.9 中做了一些改进，如第`__list_iterator`行只需传入一个模板参数，__list_node 中的指针类型也不在时void，而是指向自己本身这种类型。

```cpp
    template<typename _Tp>
    struct _List_iterator {
        typedef _Tp* pointer;
        typedef _Tp& reference;
        ···
    }

    struct _List_node_base {
        _List_node_base* _M_next;
        _List_node_base* _M_prev;
    }

    struct _List_node:public _List_node_base
    {
        _Tp _M_data;
    }
```

- 双向链表内部其实是环状的，由不能访问节点数据end()连接两头形成环状结构
