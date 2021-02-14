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

# P4. 基本构建一 new delete expression 上

`Complex* pc = new Complex(1,2);` 会被编译器转为以下程序

```cpp
Complex *pc;
try{
    void *mem = operator new(sizeof(Complex));
    pc = static_cast<Complex*>(mem);
    pc->Complex::Complex(1,2);//只有编译器可以这样直接调用ctor
}
catch(std::bad_alloc){}
```

- operator new() 可以被重载，实现自定义的内存管理

- 默认的 operator new() 中调用 `malloc` 申请内存，若申请失败会调用 `cllnewh` ，这个函数用户可以用来定义一些可以释放的变量，以便进行申请内存。

- 直接调用ctor 要用placement new `new(p) Complex(1,2);`

# P5. 基本构建一 new delete expression 中

- delete 被编译器转为 析构函数和 operator delete，operator delete内部调用free

```cpp
Complex* pc new Complex(1,2);
...
delete pc;
```

被编译器转为下面

```cpp
pc->~Complex();
operator delete(pc);//内部调用free
```

我们不能直接调用构造函数，但是可以直接调用析构函数。

# P6. 基本构建一 new delete expression 下

- 测试 无法直接调用ctor

```cpp
A* pA = new A(1);         //ctor. this=000307A8 id=1
cout << pA->id << endl;   //1
//pA->A::A(3);                //in VC6 : ctor. this=000307A8 id=3
                            //in GCC : [Error] cannot call constructor 'jj02::A::A' directly
```

# P7. Array new

- array new 的构造函数要有默认值，在array new 中无法赋初值

```cpp
Complex* pca = new Complex[3];//构造函数要有默认值，无法借由参数给予初值
delete[] pca;//调用三次析构函数
```

- array new 需要搭配 array delete,否则只会调用一次析构函数。

- 对于不含指针的对象没有什么影响，对含有指针的对象会造成内存泄漏（只释放了一个对象申请的内存，二删除了整个数组，导致这个部分内存无法再被使用）

- 若 array new 的对象的dtor需要释放内存，则申请的内存空间中会记录对象的数目，array delete 时依次析构。

# P8. placement new

- placement new 定点的new，允许我们将 `obj` 构建在已经分配好的内存中,下面第二行

```cpp
char* buf = new char[sizeof(Complex)*3];
Complex* pc new(buf)Complex(1,2);
...
delete[] buf;
```

# P9. 重载

- 一般的应用程序 `new` `-> ``operator new()` `->` `::operator new()` 可以在类内重载 `Foo:: operator new()` 实现自定义的内存管理，一般很少去动用全局的 `::operator new()`

```cpp
class Foo{
public：
    void* operator new(size_t);
    void operator delete(void* size_t)//第二参数为可选参数
}
```

# P10. 重载示例（上）

- operator new 需要是静态的，出于人性化考虑可以不加

```cpp
class Foo
{
public:
  int _id;
  long _data;
  string _str;
  
public:
  static void* operator new(size_t size);
  static void  operator delete(void* deadObject, size_t size);
  static void* operator new[](size_t size);
  static void  operator delete[](void* deadObject, size_t size);
  
  Foo() : _id(0)      { cout << "default ctor. this="  << this << " id=" << _id << endl;  }
  Foo(int i) : _id(i) { cout << "ctor. this="  << this << " id=" << _id << endl;  }
  //virtual 
  ~Foo()              { cout << "dtor. this="  << this << " id=" << _id << endl;  }
  
  //不加 virtual dtor, sizeof = 12, new Foo[5] => operator new[]() 的 size 參數是 64, 
  //加了 virtual dtor, sizeof = 16, new Foo[5] => operator new[]() 的 size 參數是 84, 
  //上述二例，多出來的 4 可能就是個 size_t 欄位用來放置 array size. 
};

void* Foo::operator new(size_t size)
{
    Foo* p = (Foo*)malloc(size);  
 cout << "Foo::operator new(), size=" << size << "\t  return: " << p << endl;   

   return p;
}

void Foo::operator delete(void* pdead, size_t size)
{
 cout << "Foo::operator delete(), pdead= " << pdead << "  size= " << size << endl;
 free(pdead);
}
```

# P11. 重载示例 （下）

- `operator new()` 可以写出多个版本，需要声明独特的参数列，且第一参数必须为 `size_t`

- `operator delete()` 也可以写出多个版本，但是只有相对应的 `operator new()` 构造抛出异常时，才会调用对应的 `delete`

- 原则上 每个版本的 `operator new()` 都要有与之对应的 `operator delete()`,但是没有写也不会报错，表示放弃对ctor的异常处理。（有的平台没有写对应版本的delete 会有警告）

```cpp
{
  class Foo
  {
  public:
    Foo() { cout << "Foo::Foo()" << endl; }
    Foo(int)
    {
      cout << "Foo::Foo(int)" << endl;
      // throw Bad();
    }

    //(1) 這個就是一般的 operator new() 的重載
    void *operator new(size_t size)
    {
      cout << "operator new(size_t size), size= " << size << endl;
      return malloc(size);
    }

    //(2) 這個就是標準庫已經提供的 placement new() 的重載 (形式)
    //    (所以我也模擬 standard placement new 的動作, just return ptr)
    void *operator new(size_t size, void *start)
    {
      cout << "operator new(size_t size, void* start), size= " << size << "  start= " << start << endl;
      return start;
    }

    //(3) 這個才是嶄新的 placement new
    void *operator new(size_t size, long extra)
    {
      cout << "operator new(size_t size, long extra)  " << size << ' ' << extra << endl;
      return malloc(size + extra);
    }

    //(4) 這又是一個 placement new
    void *operator new(size_t size, long extra, char init)
    {
      cout << "operator new(size_t size, long extra, char init)  " << size << ' ' << extra << ' ' << init << endl;
      return malloc(size + extra);
    }
}
```

```cpp
    Foo *p1 = new Foo;        //op-new(size_t)
    Foo *p2 = new (&start) Foo;    //op-new(size_t,void*)
    Foo *p3 = new (100) Foo;    //op-new(size_t,long)
    Foo *p4 = new (100, 'a') Foo; //op-new(size_t,long,char)

    Foo *p5 = new (100) Foo(1);     //op-new(size_t,long)  op-del(void*,long)
    Foo *p6 = new (100, 'a') Foo(1); //
    Foo *p7 = new (&start) Foo(1);   //
    Foo *p8 = new Foo(1);       //
```