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

- `operator new()` 可以写出多个版本，需要声明独特的参数列，且第一参数必须为 `size_t`,调用的时候括号里直接写第二个参数即可(第二个代码块)

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

# P12. per-class allocator

- 当需要多次 new obj 时，可以设计接管 operator new() 一次申请一大块空间，需要时直接分配，无需每次 `malloc`。同时可以减少维护的cookie，提高内存利用率。

- 版本一：类内变量由指针和数据构成，首次获取一大块内存，用指针连城链表进行维护

```cpp
 class Screen
  {
 public:
    Screen(int x) : i(x){};
    int get() { return i; }

    void *operator new(size_t);
    void operator delete(void *, size_t); //(2)
    //! void  operator delete(void*);      //(1) 二擇一. 若(1)(2)並存,會有很奇怪的報錯 (摸不著頭緒)

  private:
    Screen *next;
    static Screen *freeStore;
    static const int screenChunk;

  private:
    int i;
  };
  Screen *Screen::freeStore = 0;
  const int Screen::screenChunk = 24;

  void *Screen::operator new(size_t size)
  {
    Screen *p;
    if (!freeStore)
    {
      //linked list 是空的，所以攫取一大塊 memory
      //以下呼叫的是 global operator new
      size_t chunk = screenChunk * size;
      freeStore = p =
        reinterpret_cast<Screen *>(new char[chunk]);
      //將分配得來的一大塊 memory 當做 linked list 般小塊小塊串接起來
      for (; p != &freeStore[screenChunk - 1]; ++p)
        p->next = p + 1;
      p->next = 0;
    }
    p = freeStore;
    freeStore = freeStore->next;
    return p;
  }

  //! void Screen::operator delete(void *p)    //(1)
  void Screen::operator delete(void *p, size_t) //(2)二擇一
  {
    //將 deleted object 收回插入 free list 前端
  (static_cast<Screen *>(p))->next = freeStore;
    freeStore = static_cast<Screen *>(p);
  }

    //! void Screen::operator delete(void *p)    //(1)
  void Screen::operator delete(void *p, size_t) //(2)二擇一
  {
    //將 deleted object 收回插入 free list 前端
    (static_cast<Screen *>(p))->next = freeStore;
    freeStore = static_cast<Screen *>(p);
  }

```

# P13. per-class allocator 2

- 使用 `struct` 包装所有数据（第 `7` 行），使用 `union` 将数据和指针进行联合（`14`行），这种设计成为嵌入式指针

- 初始化后，未使用数据用指针相连形成链表

- 相较于上个版本，使用 `union` 联合，除去维护用的指针（未被使用的内存用指针串联，需要分配对象的内存用户会声明指针变量，因此数据和指针不会同时存在可用 `union`节省内存空间）

- 第 `44` 行 判断是否发生继承

- 这个版本的缺陷在于 `operator delete()` 中 没有 `free`，（程序 new 100000 个 obj，用完之后delete，但是没有将内存还给操作系统）

```cpp {.line-numbers}
  //ref. Effective C++ 2e, item10
  //per-class allocator

  class Airplane
  { //支援 customized memory management
  private:
    struct AirplaneRep
    {
      unsigned long miles;
      char type;
    };

  private:
    union
    {
      AirplaneRep rep; //此針對 used object
      Airplane *next;   //此針對 free list
    };

  public:
    unsigned long getMiles() { return rep.miles; }
    char getType() { return rep.type; }
    void set(unsigned long m, char t)
    {
      rep.miles = m;
      rep.type = t;
    }

  public:
    static void *operator new(size_t size);
    static void operator delete(void *deadObject, size_t size);

  private:
    static const int BLOCK_SIZE;
    static Airplane *headOfFreeList;
  };

  Airplane *Airplane::headOfFreeList;
  const int Airplane::BLOCK_SIZE = 512;

  void *Airplane::operator new(size_t size)
  {
    //如果大小錯誤，轉交給 ::operator new()
    if (size != sizeof(Airplane))
      return ::operator new(size);

    Airplane *p = headOfFreeList;

    //如果 p 有效，就把list頭部移往下一個元素
    if (p)
      headOfFreeList = p->next;
    else
    {
      //free list 已空。配置一塊夠大記憶體，
      //令足夠容納 BLOCK_SIZE 個 Airplanes
      Airplane *newBlock = static_cast<Airplane *>(::operator new(BLOCK_SIZE * sizeof(Airplane)));
      //組成一個新的 free list：將小區塊串在一起，但跳過
      //#0 元素，因為要將它傳回給呼叫者。
      for (int i = 1; i < BLOCK_SIZE - 1; ++i)
        newBlock[i].next = &newBlock[i + 1];
      newBlock[BLOCK_SIZE - 1].next = 0; //以null結束

      // 將 p 設至頭部，將 headOfFreeList 設至
      // 下一個可被運用的小區塊。
      p = newBlock;
      headOfFreeList = &newBlock[1];
    }
    return p;
  }

  // operator delete 接獲一塊記憶體。
  // 如果它的大小正確，就把它加到 free list 的前端
  void Airplane::operator delete(void *deadObject,
                   size_t size)
  {
    if (deadObject == 0)
      return;
    if (size != sizeof(Airplane))
    {
      ::operator delete(deadObject);
      return;
    }

    Airplane *carcass =
      static_cast<Airplane *>(deadObject);

    carcass->next = headOfFreeList;
    headOfFreeList = carcass;
  }
```

# P14. static allocator

- 可以为每个需要的类写之内存管理，但是每个类都会有大量重复的代码，因此可以将这种操作独立出来写一个类

```cpp
class allocator
  {
  private:
    struct obj
    {
      struct obj *next; //embedded pointer
    };

  public:
    void *allocate(size_t);
    void deallocate(void *, size_t);
    void check();

  private:
    obj *freeStore = nullptr;
    const int CHUNK = 5; //小一點方便觀察
  };

  void *allocator::allocate(size_t size)
  {
    obj *p;

    if (!freeStore)
    {
      //linked list 是空的，所以攫取一大塊 memory
      size_t chunk = CHUNK * size;
      freeStore = p = (obj *)malloc(chunk);

      //cout << "empty. malloc: " << chunk << "  " << p << endl;

      //將分配得來的一大塊當做 linked list 般小塊小塊串接起來
      for (int i = 0; i < (CHUNK - 1); ++i)
      { //沒寫很漂亮, 不是重點無所謂.
        p->next = (obj *)((char *)p + size);
        p = p->next;
      }
      p->next = nullptr; //last
    }
    p = freeStore;
    freeStore = freeStore->next;

    //cout << "p= " << p << "  freeStore= " << freeStore << endl;

    return p;
  }
  void allocator::deallocate(void *p, size_t)
  {
    //將 deleted object 收回插入 free list 前端
    ((obj *)p)->next = freeStore;
    freeStore = (obj *)p;
  }
  void allocator::check()
  {
    obj *p = freeStore;
    int count = 0;

    while (p)
    {
      cout << p << endl;
      p = p->next;
      count++;
    }
    cout << count << endl;
  }
  //--------------

  class Foo
  {
  public:
    long L;
    string str;
    static allocator myAlloc;

  public:
    Foo(long l) : L(l) {}
    static void *operator new(size_t size)
    {
      return myAlloc.allocate(size);
    }
    static void operator delete(void *pdead, size_t size)
    {
      return myAlloc.deallocate(pdead, size);
    }
  };
  allocator Foo::myAlloc;

  class Goo
  {
  public:
    complex<double> c;
    string str;
    static allocator myAlloc;

  public:
    Goo(const complex<double> &x) : c(x) {}
    static void *operator new(size_t size)
    {
      return myAlloc.allocate(size);
    }
    static void operator delete(void *pdead, size_t size)
    {
      return myAlloc.deallocate(pdead, size);
    }
  };
  allocator Goo::myAlloc;

  //-------------
  void test_static_allocator_3()
  {
    cout << "\n\n\ntest_static_allocator().......... \n";

    {
      Foo *p[100];

      cout << "sizeof(Foo)= " << sizeof(Foo) << endl;
      for (int i = 0; i < 23; ++i)
      { //23,任意數, 隨意看看結果
        p[i] = new Foo(i);
        cout << p[i] << ' ' << p[i]->L << endl;
      }
      //Foo::myAlloc.check();

      for (int i = 0; i < 23; ++i)
      {
        delete p[i];
      }
      //Foo::myAlloc.check();
    }

    {
      Goo *p[100];

      cout << "sizeof(Goo)= " << sizeof(Goo) << endl;
      for (int i = 0; i < 17; ++i)
      { //17,任意數, 隨意看看結果
        p[i] = new Goo(complex<double>(i, i));
        cout << p[i] << ' ' << p[i]->c << endl;
      }
      //Goo::myAlloc.check();

      for (int i = 0; i < 17; ++i)
      {
        delete p[i];
      }
      //Goo::myAlloc.check();
    }
  }
```

- 相较于之前的设计，这里application classes 与内存不再纠缠不清，所有与内存相关的细节都让`allocator`去操作。

- 这样无需为每个类单独设计内存管理，这里的内存管理是类内的一个静态变量