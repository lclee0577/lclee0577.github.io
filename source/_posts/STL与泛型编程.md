---
title: STL与泛型编程
date: 2020-12-03 08:48:24
tags:
categories: C++
toc: true
---

<!-- markdownlint-disable MD025 -->
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
    cout<< count_if(vi.begin(),vi.end(),   //begin,end - 迭代器，count_if-算法
                        not1(bind2nd(less<int>(),40)));//not1,bind2nd - 算法适配器，less - 仿函数
    return 0;
}
```

- 11行：用数组数据创建容器
- 12行：输出数组中所有符合条件的数
- 13行：`bind2nd(less<int>(),40)` 意思是`return (num_in < 40)`将小于号的第二参数绑定为20，加上之前的`not1`取反，意为统计所有大于40的数。

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

# P13.深度探索list (1) 上

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

# P14.深度探索list (1) 下

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

# P15.迭代器的设计原则和 Iterator Traits的作用与设计

- 迭代器是沟通算法与容器的桥梁，Iterator 必须提供5种 `associated types`才能使算法正确工作，(就是P13标出那5种)

- 如果传入的不是 iterator，而是一个 native pointer (可以理解为一个退化的迭代器)，因此需要插入一个中间层 `Iterator Traits` 利用偏特化来分离class iterator 和 non-class iterator。

    ```cpp {.line-numbers}
    template <class I>
    struct iterator_traits{ //traits 是特征的意思
        typedef typename I::value_type value_type
    };

    //两个 partial specialization
    template <class T>
    struct iterator_traits<T*>{
        typedef T value_type;
    };

    template <class T>
    struct iterator_traits<const T*>{
        typedef T value_type; //注意是T而不是const T
    }
    ```

  - 注意第14行是`T`而不是`const T`，原因是在算法中这些类型要用来声明变量，若为`const`就无法进行后续的处理

- 于是当需要知道`I`的value type 时便可以向下放这么写,当I是`class iterator`时，调用上方`1-4`行；若是pointer to T，则调用`7-10`行；若是pointer to const T，则调用`12-15`行

    ```cpp
    template<typename I,···>
    void algorithm(···){
        typename iterator_traits<I>::value_type v1;
    }
    ```

- 上面举例一个特性，其他的5个也是相同的操作。指针类型偏特化时，一般是随机读取型tag `typedef random_access_iterator_tag iterator_category`

- 除了 iterator traits 还有其他各式各类的traits，如 type traits，char traits， allocator traits ···

# P16.vector深度探索(1)

- vector 的扩充不是原地扩充，而是申请一块更大的内存，再把原来的数据复制过去，实现扩充。

- vector 由三个指针来维护，`start`， `finish`，`end_of_storage`。因此 `size` = `finish` - `start`， `capacity` = `end_of_storage` - `start`。

- 当调用`push_back`时，若`finish`==`end_of_storage`，则调用`insert_aux()`插入辅助函数。`insert_aux()`会重新申请内存，拷贝原有数据，同时还要拷贝当前数据之后的数据。（`insert_aux()`还会被`insert()`函数调用，vector中间插入后扩充，还要复制插入点之后的元素），再释放原来的vector，重新调整`start`， `finish`，`end_of_storage`三根指针指向新的位置。

## vector's iterator

- 由于vector是连续内存空间，因此迭代器无需设计的一个`class`，使用一个指针即可。（上一章讲的链表在内存中不连续）

```cpp {.line-numbers}
//G2.9
template<class T, class Alloc = alloc>
class vector{
public:
    typedef T value_type;
    typedef value_type* iterator;//T*
···
}

vector<int> vec;
···
vector<int>::iterator ite = vec.begin();
iterator_traits<ite>::iterator_category
```

- 上面最后行就是通过萃取机 `iterator_traits` 获取指针类型的5中特征。（P15中的指针类型萃取example）

- 在G4.9中 vector的实现非常复杂，但是功能完全一致，多了很多类的继承和复合

# P17. array，forward_list深度探索(1)

## 容器array

- 数组是C就已经提供的数据结构，将其封装成容器，是为了方便算法的调用。

```cpp
//TR1  c++技术报告1，在c++98 和 C++11 之间的版本
template<typename _Tp, std::size_t _Nm>
struct array{
    typedef _Tp         value_type;
    typedef _Tp*        pointer;
    typedef value_type* iterator;//是native pointer G2.9也是如此

    value_type _M_instance[_Nm ? _Nm :1];

    iterator begin(){ return iterator(& _M_instance[0]);}
    iterator end(){ return iterator(& _M_instance[_Nm]);}
}
```

- 与普通的数组一样 array没有ctor 和dtor.

- G4.9版本的array有复杂的继承关系，但实质没变。之前介绍过双向链表list，forwa_list 不在多说。

# P18.deque、queue 和 stack深度探索(上)

- deque 表现为一个双向可拓展的结构，其实在底层使用vector保存各个缓冲区的信息（这个vector相当于一个控制中心），各个缓冲区在内存中其实是不连续的，通过vector中保存的信息实现不同缓冲区的切换，对用户来说就像是连续空间一样。

- 比较优秀的一点是deque的insert()在插入时会判断插入位置离哪一端更近，从而减少移动元素带来的开销

# P19.deque、queue 和 stack深度探索(下）

- deque 模拟连续空间都是 deque iterators 的功劳。 iterator中有4个成员：`cur`-当前元素，`first`-缓冲区开始地址，`last`-缓冲区结尾地址，`node`-vector中的下一个节点

```cpp
difference_type operator-(const self& x) const
{
    return difference_type(buffer_size())*(node-x.node-1)+(cur-first)+(x.last-x.cur);
}
```

- 计算两个元素像差的地址：像差缓冲区数量*缓冲区大小+起始buffer中元素数量+结束buffer中元素数量。

- 后置自增/减会调用前置自增/减。

- 当控制中心vector需要扩充时，原来的数据复制到新的中间位置，为两头留下增长余量。

- queue 和 stack 内含了一个deque，减少了某些功能实现的

- queue 和 stack 都可以选择list 和deque作为底层结构，默认deque。

- queue 和 stack 不允许遍历，也不提供迭代器。

- stack 可以选择 vector 作为底层，queue则不能。

- queue 和 stack 都不能选择set或map做底层

# P20.RB-tree深度探索

- Red-Black tree(红黑树) 是高度平衡二分搜索树，排列规则有利于 search 和 insert，并且保持平衡，不会使任何一个节点太深。

- 不应该使用 rb_tree 的 iterator 改变元素值（因为元素有严谨的排序），但是在变成层面并未禁止此事。这是为了后续的map考量，map 中允许修改元素的 data，只有元素的 key 才是不可修改的。

- rb_tree 提供两种插入操作： `insert_unique()` 和 `insert_equal()`, 前者表示插入的key独一无二，后者表示插入的key可重复，

- 红黑树中的 value 是包含 key 和 data 的整体。

- G2.9 - G4.9 容器的实现都变得复杂，包含了多个类的继承与复合。虽然复杂不利于学习但是遵循了一个面向对象的原则 `handle and body` 桥接模式。抽象部分与实现部分分离。

# P21.set multiset 深度探索

- set/multiset 以 rb_tree 为底层结构，因此有元素自动排序的特性。set/multiset 元素的 value 和 key 合一；value 就是 key

- set/multiset 提供遍历操作，按照正常规则 ++ite 遍历可以获得排序状态

- set/multiset 无法通过 iterator 修改元素值（不同于rb_tree）

- set 的 key 必须是独一无二的，底层调用的是红黑树的 `insert_unique()`; multiset 的key 可以重复，底层调用的是红黑树的 `insert_equal()`

- set 内的 iterator 是 `const_iterator` 类型，防止用户通过迭代器修改set的内容。

- set所有的操作都是调用底层的红黑树完成，从这个意义上看，set也能称之为 Container Adapter

# P22.map,multimap 深度探索

- map 底层也是由红黑树实现，`typedef pair<const Key, T> value_type` 来实现只允许改 data，不允许改 Key

- map 独特的 `operator[](const key_type& __k)`, 与python中的字典相同，若key存在则返回data，若不存在则创建（multimap 不允许用[]）。

# P23.hashtable 深度探索(上)

- obj 通过`散列函数-hash function` 计算出`编号-hash code`，再对 hashtable 的长度求余数，放到 hashtable bucket中。若出现重复则使用链表串联。

- `Separate Chaining` 虽然 list 是线性搜索空间，如果list足够小，搜索速度任然很快。

- hashtable 的长度通常为质数，在gnu c中默认值一般为53.

- 当 obj 个数大于hashtable的长度时，hashtable 将扩充到两倍容量相近的那个质数的大小，最接近 53*2 的质数为97（stl里有质数表，直接查表就即可），因此hashtable扩充到 97，在对所有元素重新打散 rehashing

- 与 deque 一样，`hashtable`内部的控制中心是 vector 类型的 buckets， `iterator` 在进行 `++`的操作，指向链表的末位时，能回到控制中心，再指向下一个元素。

# P23.hashtable 深度探索(下)

```cpp
hashtable<const char*,
        const char*,
        hash<const char*>,
        identity<const char*>,
        eqstr
        alloc>
ht(50,hash<const char*>(),eqstr());
ht.insert_unique("kiwi");
ht.insert_unique("plum")
ht.insert_unique("apple")

struct eqstr{
    bool operator()(const char* s1,const char* s2) const { return strcmp(s1,s2)==0;}
}
```

- 比较 `c-string` 是否相等可用`strcmp`，但返回值是`-1,0,1`，不是bool，需要重新包装。

- 默认的类型都有特化版本的 hash 函数。对于自定义的类型要自己设计 hash-function（使hash足够乱，足够随机）。注意标准库没有提供 `hash<std::string>`

# P25.这节课的视频跟之前重复了

# P26.unordered 容器概念

- 之前所有 hash 开头的数据结构在c++11中 都以 unordered_ 开头。

- 可以调用 `.bucket_size(i)` 查看第 `i` 个 bucket 中的元素数量。

到此为止第二讲容器的探讨全局完成

---

# P27.算法的形式

- 算法 - function template， 目的是为了处理容器的内容

- 算法无法看到容器的所有内容，需要通过迭代器来进行访问。

# P28.迭代器的分类

- 五种 iterator category,表示迭代器的移动方式。除了output外，其他四个自下而上继承
  - input_iterator_tag, output_iterator_tag
  - forward_iterator_tag
  - bidirectional_iterator_tag
  - random_access_iterator_tag

- input_iterator_tag, output_iterator_tag 对应 istream_iterator，ostream_iterator 后面的课程会深入讲解

- 可以通过 `typeid(ite).name()` 来查看迭代器的类型，但是输出还会附加一些编译过程的其他信息，取决于编译器的实现。

# P29.迭代器分类对算法的影响

- 如需要求两个指针之间的距离，对于可随机访问的迭代器来说就是两个地址相减，若不是可随机访问的，则需要一直`++`使两个指针地址相同来计算距离。

- 又例如前进`advance()`这个函数，若是可随机访问，则直接地址+偏移量，否则就要一个一个跳转

- 迭代器种类的继承关系使得在不同函数重载时，便于确定是哪一种类型。（有的函数只有`random_access`和`input`类型的两种版本，那么forward和bidirectional也就会归为`input`这一类。

- function template 没有所谓的特化，用的都是重载的手法。例如常用的copy，若是const char* 这种类型则调用 `memmove()`-(速度极快)，若是普通的迭代器，且构造函数不重要，那么也调用`memmove()`，否则就要一个一个创建。

- 算法源码中对 `iterator_category`的暗示: 原则上算法可以接受任意类型的迭代器，但是算法参数的声明可能会暗示我们要输入的类型

    ```cpp
    template<class RandomAccessIterator> 
    sort(RandomAccessIterator first,RandomAccessIterator last)
    ```

# P30.算法源码剖析

- 标准库中的算法前两个参数通常是容器的两个迭代器（指针-退化的迭代器）

```cpp
template<class InputIterator,class T>
T accumulate(InputIterator first, InputIterator last, T init){
    for(; first != last; ++first)
        init = init + *first;//将元素值累加到 init 上
    return init；
}

template<class InputIterator,class T,class BinaryOperation)
T accumulate(InputIterator first, InputIterator last, T init, BinaryOperation binary_op)//binary_op 接受两个参数的函数
    for(; first != last; ++first)
        init = binary_op(init + *first);//将元素值累加到 init 上
    return init；
```

- 以上是标准库的一个算法，第二个重载版本可以调用自定义的函数进行累计算操作。

```cpp
int myfunc(int x,int y){return x+2*y;}

struct myclass{
    int operator()(int x, int y){return x+3*y;}
}myobj;//仿函数

int init = 100;
int num[] = {10,20,30};
cout << accumulate(nums,nums+3,init);//160
cout << accumulate(nums,nums+3,init,minus<int>());//40
cout << accumulate(nums,nums+3,init,myfunc)；//220
cout << accumulate(nums,nums+3,init,myobj);//280 
```

- binary_search 之前要验证是否已经排好序。

```cpp
template<class ForwardIterator, class T>
bool binary_search(ForwardIterator first, ForwardIterator last, const T& val){
    first = std::lower_bound(first, last, val);
    return(first!=last && !(val<*first));
}
```

- lower_bound 返回的是找到元素的最小的位置（里面可能有重复，找最大的位置为upper_bound）

- 需要判断目标位置不是end，目标值也不小于首元素

- 其实应该先判断不小于首元素在继续二分查找

# P31.仿函数和函数对象

- functors 是在平时编程中最有可能需要自己编写的部分

```cpp
template <class T>
struct plus:public binary_funcation<T,T,T>{
    T operator()(const T& x,const T& y) const {return x + y;}
}
```

- 因为需要把加减乘除等这些操作传给算法，因此需要将其变成函数。

- 上一讲 struct 没有继承 因此无法融入到stl的体系中（当前的函数可以调用，但是标准库的其他函数不一定能调用，需要一些继承关系），我们自己写的仿函数往往没有继承。

- STL规定每个 Adaptable Function 都应挑选适当的继承。(函数接受几个参数，以及参数种类),这种继承没有带来额外的开销（占用数据空间），但是有利于适配。

```cpp
template <class Arg, class Result>
struct unary_function{//一个参数
    typedef Arg argument_type;
    typedef Result result_type;
};

template <class Arg1,class Arg2, class Result>
struct binary_function{//两个参数 
    typedef Arg1 first_argument_type;
    typedef Arg2 second_argument_type;
    typedef Result result_type;
};
```

- 仿函数就是重载小括号 `()`, 其实是一个对象，但是像是一个函数。

# P32.存在多种Adapters

- 对现有的函数或数据结构进行简单的修饰，如三个参数变为两个（在特定条件下固定一个参数），修改函数名称等等。

- 有两种实现方式：继承 和 复合

- 容器适配器 stack 和 queue 内含一个 deque，底层功能都是由 deque 实现。

```cpp
template <class T, class Sequence=deque<T>>//底层容器默认使用双向队列
class stack{
···
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::size_type size_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
protected:
    Sequence c;//底层容器
public:
    bool empty() const {return c.empty();}
    size_type size() const {return c.size();}
    reference top() {return c.back();}
    const_refernece top() const {return c.back();}
    void push(const value_type& x){c.push_back(x);}
    void pop(){c.pop_back();}

}
```
  
# P33.函数适配器：bind2nd

```cpp
    int ia[6] = {27, 210, 12, 47, 109, 83};
    vector<int, allocator<int>> vi(ia,ia+6); //vector-容器，allocator-分配器
    cout<< count_if(vi.begin(),vi.end(),   //begin,end - 迭代器，count_if-算法
                        not1(bind2nd(less<int>(),40)));//not1,bind2nd - 算法适配器，less - 仿函数
```

- 找到vector中所有不小于40的数，现在来介绍 `bind2nd` 

---

- 辅助函数，让使用者可以更加方便使用 `binder2nd<Op>`,因为函数模板可以进行实参推到，编译器会自动推导出 `Op` 的类型。

```cpp
template<class Operation, class T>
inline binder2nd<Operation> bind2nd(const Operation& op, const T& x){
    typedef typename Operation::second_argument_type arg2_type;//从为声明对象的类中访问子对象类型 需要加 typename
    return binder2nd<Operation>(op,arg2_type(x)); //类型（参数），构造函数 创建对象
}

```

- 模板类，一般难以直接写出 `Operation` 的类型，因此采用辅助函数。

```cpp
template <class Operation>
class bind2nd: public unary_function<typename Operation::first_argument_type,typename Operation::second_argument_type>{

    protected:
    Operation op;// 内部成员，记录函数和第二实参
    typename Operation::second_argument_type value;

    public:
    //构造函数
    binder2nd(const Operation& x,const typename Operation::second_argument_type& y)
    :op(x),value(y){}//记录操作函数和实参

    typename Operation::result_type //这个类中有很多这样的类型，是为了保持适配后的类型与源操作一致
    operator()(const typename Operation::first_argument_type& x) const{
        return op(x,value);//调用函数并固定第二实参
    }
}
```

- 在绑定第二参数后，仍有一个参数，应当继承 `unary_function`,以便还要被适配，以兼容STL体系

- `bind2nd(less<int>(), 40))`生成一个仿函数对象，记录操作和实参。由于重载小括号可以在 `count_if` 算法中调用

- 上式中 `less<int>()` 为创建对象的构造函数而非操作符重载

```cpp
 template <class InputIterator, class Predicate>
 typename iterator_traits<InputIterator>::difference_type
 count_if(InputIterator first,InputIterator,last,Predicate pred){
     typename iterator_traits<InputIterator>::difference_type n = 0;
     for(;first != last; ++first)
        if(pred(*first))//这里调用 重载小括号，绑定第二参数
            ++n;
    return n;
 }
```

- 现在 `bind2nd`、`binder2nd` 已经过时，现在由 `bind` 取代。

# P34. 函数适配器 not1

```cpp
//辅助函数，便于使用
template <class Predicate>
inline unary_nagete<Predicate> not1(const Predicate& pred){
    return unary_negate<Predicate>(pred);//创建对象
}

template <class Predicate>
class unary_negate: public unary_function<typename Predicate::argument_type,bool>{
    protected:
        Predicate pred;
    public:
        explicit unary_negate(const Predicate& x):pred(x){}//构造函数
        bool operator()(const typename Predicate::argument_type& x) const{
            return !pred(x);//结果取否
        }
}
```

- 由于 `bind2nd` 继承 `unary_function`, `not1` 可以继续适配.

# P35.新型适配器 bind c++11

- std::bind 可绑定：

    1. functions

    2. function objects

    3. member functions, _1必须是某个object地址(_1是占位符)

    4. data members, _1必须是某个object地址

```cpp
//使用仿函数也是同理 std::divides<double> my_divide;
double my_divide(double x,double y)
    {return x/y ;}

struct MyPair{
    double a,b;
    double multiply() { return a*b; }
}

using namespace std::placeholders;// adds visibility of _1, _2 

//绑定函数
auto fn_five = bind(my_divide,10,2); //返回10/5
cout << fn_five() << '\n';           //5   

auto fn_half = bind(my_divide, _1, 2); //返回x/2
cout << fn_hale(10) << '\n';           //5

auto fn_invert = bind(my_divide,_2,_1);//返回y/x
cout << fn_invert(10,2)<<'\n';          //0.2

auto fn_rounding = bind<int>(my_divide,_1,_2); //返回int(x/y)
cout << fn_rounding(10,3)<<'\n';               //3

//绑定成员
MyPair ten_two {10,2}; // C++11 新加入的初始化方式

//成员函数都含有隐藏的 this指针
auto bound_menfn = bind(&MyPair::multiply,_1); //return x.multiply()
cout << bound_menfn(ten_two) <<'\n';           //20

auto bound_memdata = bind(&MyPair::a, ten_two); //return ten_two.a
cout <<bound_memdata() <<'\n';                  //20

auto bound_memdata2 = bind(&MyPair::a, _1);     //return x.a
cout << bound_memdata2(ten_two) <<'\n';         //2

// 重新实现两章的内容
vector<int> v{15,37,94,50,73,58,28,98};
cout << count_if(v.cbegin(),v.cend(),not1(bind2nd(less<int>(),50))) <<endl;//之前的用法仍旧兼容

auto fn_= bind(less<int>(),_1,50)
cout << count_if(v.cbegin(),v.cend(),fn_) << endl;
```

# P36.迭代器适配器 reverse_iterator

- 逆向取迭代器

```cpp
template <class Iterator>
class reverse_iterator{
    protected:
        Iterator current;
    public:
        //逆向迭代器5种相关的类型 与正向相对应
        typedef typename iterator_traits<Iterator>::iterator_category iterator_category;
        typedef typename iterator_traits<Iterator>::value_type value_type;
        ···
        typedef Iterator iterator_type;             //表示正向迭代器
        typedef reverse_iterator<Iterator> self;    //表示逆向迭代器
    public:
        explicit reverse_iterator(iterator_type x):current(x){}
                 reverse_iterator(const self& x):current(x.current){}
        iterator_type base() const {return current;}//取出对应正向迭代器
        reference operator*() const{Iterator tmp = current; return *--tmp;}//逆向迭代器退一位取值
        pointer operator->()const {return &(operator*());}

        //前进改后退，后退改前进
        self& operator++() {--current;return *this;}
        self& operator--() {++current;return *this;}
        self operator+(difference_type n) const {return self(current - n)}
        self operator-(difference_type n) const {return self(current + n)}
        //头变尾 尾变头
        reverse_iterator rbegin() { return reverse_iterator(end());}
        reverse_iterator rend() { return reverse_iterator(begin());}
}
```

# P37.迭代器适配器 inserter

```cpp
int myints[] = {10,20,30,40,50,60,70};
vector<int> myvec(7);
copy(myints,myints+7,myvec.begin());
```

- 首先看一段从数组拷贝到 `vector` 的程序，其中 `copy` 的主要代码如下所示

```cpp
template<class InputIterator,class OutputIterator>
OutputIterator copy(InputIterator first, InputIterator last, OutputIterator result){
    while(first!=last){
        *result = *first;
        ++result;++first;
    }
    return result
}
```

- 可以看出是循环赋值，并没有检查边界。因此在 `vector` 中要留好足够的空间。

- 下面是一段插入的程序片段

```cpp
list<int> foo, bar;
for(int i =1, i <=5, i++)
    {foo.push_back(i);bar.push_back(i*10);}
// 初始化，foo 为1-5， bar 为10-50

list<int>::iterator it foo.begin();
advance(it,3) // list为不连续空间，使用advance 一个一个跳转

copy(bar.begin(), bar.end(),inserter(foo,it))
//1,2,3,10,20,30,40,50,4,5
```

- 在 `copy` 中是一个一个赋值动作，并没有重新申请空间。inserter 使用重载等号的方法实现在 `copy` 中扩容。

- 在inserter 中 等号被重载为容器的插入操作,(容器默认提供了插入操作，会自动申请空间)

```cpp
template<class Container, class Iterator>
inline inserter_iterator{
    protected:
        Container* container;
        typename Container::iterator iter;
    public:
        inserter_iterator(Container& x, typename Container::iterator i)
            :container(&x),iter(i){}
        
        inserter_iterator<Container>&
        operator=(const typename Container::value_type& value){
            iter = container->insert(iter,value);//这一步是关键，变移动为插入。这里的返回值指向新插入的元素
            ++iter;//同时 iter 紧跟目标移动，指向下一个位置
            return *this;
        }
}
```

- inserter_iterator 的辅助函数 实现类型推导

```cpp
template<class Container, class Iterator>
inline inserter_iterator<Container>
insert(Container& x, Iterator i){
    typedef typename Container::iterator iter;
    return inserter_iterator<Container>(x,iter(i))
}
```

# P38. X适配器 ostream_iterator

```cpp
vector<int> myvector;
for(int i =1; i < 10; ++i) myvector.push_back(i*10);

ostream_iterator<int> out_it(cout,",") //构造 类型为cout， 分隔符为，
copy(myvector.begin(),myvector.end(),out_it);
//打印出 10,20，30,40,50,60,70,80,90
```

- 通过操作符重载，在不改变 `copy` 程序(见上一章)的情况下，实现对输出的适配(对硬件也就是屏幕输出的绑定)。

- `copy` 中的赋值操作变成输出字符，迭代器自增不变

```cpp
template<class T, class charT = char, class traits = char_traits<chatT>>
class ostream_iterator:
public iterator<output_iterator_tag,void,void,void,void>
{
basic_ostream<chatT,traits>* out_stream;
const chatT* delim;

public:
    typedef charT char_type;
    typedef traits traits_type;
    typedef basic_ostream<charT,traits> ostream_type;
    ostream_iterator(ostream_type& s):out_stream(&s),delim(0){}
    ostream_iterator(ostream_type& s, const charT* delimiter)
        :out_stream(&s),delim(delimiter){}
    ostream_iterator(const ostream_iterator<T,charT,traits>& x)
        :out_stream(x.out_stream),delim(x.delim){}
    ~ostream_iterator(){}

    ostream_iterator<T,charT,traits>& operator=(const T& value){
        *out_stream << value;          //重载等号为输出
        if(delim != 0) *out_stream << delim;//若有分隔符，一起输出
        return *this;
    }
    ostream_iterator<T,charT,traits>& operator*(){return *this}//对地址的操作全部返回自身
    ostream_iterator<T,charT,traits>& operator++(){return *this}
    ostream_iterator<T,charT,traits>& operator++(int){return *this}
}
```

# P39. X适配器 i stream_iterator

- 对输入的绑定，相当于绑定键盘

```cpp
double value1,value2;
cout<< "please insert two values:";
istream_iterator<double> eos;//end of stream_iterator, 没有参数作为结束标志位
istream_iterator<double> iit(cin) //stdin iterator 创建迭代器时已经要求输入了，相当于cin>> value;详见下方的定义
if(iit!=eos) value1 = *iit;//相当于return value

++iit;//自增相当于读取下一个
if(iit!=eos) value2 = *iit;
```

```cpp {.line-numbers}
template<class T,class charT= char,class traits = char_traits<charT>,class Distance=ptrdiff_t>
class istream_iterator
    :public iterator<input_iterator_tag,T,Distance,const T*,const T&>
{
basic_istream<chatT,traits>* in_stream;
T value;
public:
    typedef charT char_type;
    typedef traits traits_type;
    typedef basic_istream<charT,traits> istream_type;

    istream_iterator():in_stream(0){}
    istream_iterator(istream_iterator& s):in_stream(&s){++*this}//值得注意的是：一旦创建立即读取，见下面操作符重载 18行
    istream_iterator(const istream_iterator<T,charT,traits,Distance>& x):in_stream(x.in_stream),value(x.value){}
    ~istream_iterator(){}
    const T& operator*() const {return value;}
    const T& operator->() const {return &value;}
    istream_iterator<T,charT,traits,Distance>& operator++(){
        if(in_stream&&!(*in_stream>>value)) in_stream=0;
        return *this;
    }
    istream_iterator<T,charT,traits,Distance>& operator++(int){//后++
        istream_iterator<T,charT,traits,Distance> tmp = *this;
        ++*this;
        return tmp;
    }
}
```

```cpp
istream_iterator<int>iit(cin),eos;

copy(iit,eos,inserter(c,c.begin()))
```

- 使用 `copy` 将输入存储在vector中

# P40.一个万用的 hash function

- 计算一个自定义类型的hash（由常用的类型组成），假设 Custom 类 含有 `string fname,lname;long no;` 

- 常见的类型标准库自带 hash function

- 自定义hash function 有三种方法

```cpp
//方式一：创建可调用类型
class CustomerHash
{
public:
    std::size_t operator()(const Customer& c) const{
        return ......
    }
};
unordered_set<Customer, CustomerHash> custest;
 
//方式二：创建哈希函数
size_t customer_hash_func(const Customer& c){
    return ......
}
unordered_set<Customer, size_t(*)(const Customer&)> custest(20, customer_hash_func); 
//注意这里调用了构造函数的不同版本,size_t(*)(const Customer&) 是hash function 的类型
 
//方式三：创建默认值hash<T>的特化版本
namespace std
{
template<>
struct hash<Customer>
{
    size_t
    operator()(const Customer& c) const noexcept
    { return ...... }
};
}
unordered_set<Customer> custest;
```

- 一共有三种方法创建自定义hash function
    1. 创建可调用类型
    2. 创建哈希函数，但是创建对象时需要声明函数类型和函数地址，有一定难度
    3. nordered_set 默认值`hash<T>`，我们只需创建自定义类型的`hash<T>`的特化版本即可

- 标准库提供了 `hash_combine`(下面21行)，这个函数并没有什么数学推导，其中`0x9e3779b9`为黄金分割率。

- 使用 variadic templates 变参模板（C++11），类似于递归每次计算第一个参数直到结束。

- 用户调用第2行，实际调用的版本是滴9行，将参数逐个拆解计算，于第16行读取完所有参数终止计算。由于是传递引用，每次计算参数都在修改 `seed`，最后 `seed` 就是我们的哈希值。

```cpp {.line-numbers}
template <typename... Types>
inline size_t hash_val(const Types&... args){        //提供给使用者的重载版本
    size_t seed = 0;
    hash_val(seed, args...);                         //实际调用的版本
    return seed;
}
 
template <typename T, typename... Types>
inline void hash_val(size_t& seed,                   //实际调用函数的主体
                     const T& val, const Types&... args){
    hash_combine(seed, val);
    hash_val(seed, args...);
}
 
template <typename T>
inline void hash_val(size_t& seed, const T& val){    //实际调用函数的边界条件
    hash_combine(seed, val);
}
 
template <typename T>
inline void hash_combine(size_t& seed, const T& val){
    seed ^= std::hash<T>()(val) + 0x9e3779b9
            + (seed<<6) + (seed>>2);
}
 
//下面是对于 class Customer 的实际使用
class CustomerHash{
public:
    std::size_t operator()(const Customer& c) const{
        return hash_val(c.fname, c.lname, c.no);
    }
};
```

# P41.tuple 用例

- `tuple` 可以将任意多个类型作为一组作为一种新的类型来声明变量。不同于 `vector` 和 `list` 创建对象时要固定变量的类型  

```cpp
tuple<string,int,int,complex<double>> t;
cout<< "sizeof =" << sizeof(t)<<endl; //32 不是28 4+4+4+(8+8)=28 应该要跟最大的元素对齐,所以应是16的倍数

tuple<int,float,string>t1(41,6.3,"nico")
cout << "t1: "<< get<0>(t1) <<" "<< get<1>(t1)<< " "<<get<2>(t1)<<endl;//分别取出元素

auto t2 = make_tuple(22,44,"stacy");

get<1>(t1) = get<1>(t2);//成员之间可以互相赋值

if(t1<t2) //之间可以互相比较
    cout<<"t1<t2"<<endl;
else
    cout<<"t1>=2"<<endl;

t1 = t2;//可以互相赋值

tuple<int,float string> t3(77,1.1,"more light")
int i1;
float f1;
string s1;
tie(i1,f1,s1) = t3; //批量赋值，t3 在之后的改动并不会影响前面三个变量

typedef tuple<int,float,string> TupleType;
cout << tuple_size<TupleType>::value << endl;// 3
cout << tuple_element<1,TupleType>::type f1 = 1.0; //取出第第二个的类型来声明变量，也就是float f1=1.0
```

- 可以使用 `tuple` 和类型直接组合 创建元祖，也可以使用 `make_tuple` (内含类型推导)

- 如同普通类型一样可以进行大小比较，赋值

- 可以用 `get`获取元祖成员的数据,`tie`进行批量赋值，`tuple_size` 查询元祖成员数量，`tuple_element`

```cpp {.line-numbers}
template<typename... Values> class tuple;
template<> class tuple<> {};
 
template<typename Head, typename... Tail>
class tuple<Head, Tail...> :private tuple<Tail...>
{
    typedef tuple<Tail...> inherited;
public:
    tuple() {}
    tuple(Head v, Tail... vtail):m_Head(v), tuple<Tail...>(vtail...) {}
 
    typename Head::type head() { return m_Head; }
    inherited& tail() { return *this; }
protected:
    Head m_Head;
};
```

- 递归继承，例如 `tuple<int,float,string>` 的继承顺序 `tuple<>` ← `tuple<string>` ← `tuple<float,string>`←`tuple<int,float,string>`，

- 递归继承终止与第二行，一个空的元祖

- 因此声明一个元祖，就可以将其分为头和尾，尾又可以再分成头尾，以此下去，直到最后一个元素的尾指是空的元祖。第 `10` 行 `tuple<Tail...>(vtail...)`重新调用构造函数，将其分成 head 和 tail

- 第 `13` 行 `return *this` 但是类型被转换成 `inherited` 获得的是尾部的元祖

```cpp
tuple<int,float,string> t(41,6.3,"nico");
t.head() //41
t.tail().head()//6.3
```

# P42.type traits

- POD Plain Old ，c 风格的数据，如struct里没有函数

- c++11 提供了一系列萃取机来获得类的特性,是不是整数，类，仿函数，是否含有指针，构造，析构函数是否重要等等

```cpp
template <typename T>
void type_traits_output(const T& x)
{
    cout<< "is_void\t"<< is_void<T>::value <<endl;
    cout<< "is_integral\t"<< is_integral<T>::value <<endl;
}
```

# P42.type traits实现

- 都是使用一些模板的特化和偏特化实现

```cpp
template <typename _Tp>
struct remove_const
{ typedef _Tp type; };

template <typename _Tp>
struct remove_const<_Tp const>
{ typedef _Tp type; };//特化版本 如果含有const，则定义为无const 版本 volatile 同理

template <typename _Tp>
struct remove_cv
{ typedef typename 
    remove_const<typename remove_volatile<_Tp>::type>::type type; }; //先移除无关的 const， volatile


template<typename>
struct __is_void_helper : public false_type { }; 

template<>
struct __is_void_helper<void> : public true_type { }//只有为void 才继承真


template<typename _Tp>
struct is_void : public __is_void_helper<typename remove_cv<_Tp>::type>::type
{ };

```

- 对于一些负载的类型萃取，如 is_class 则无法看到源码，推测应该是编译器帮忙完成一些类型的推导。