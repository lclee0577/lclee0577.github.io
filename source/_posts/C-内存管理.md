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

# P15. macro for static allocator

- 每个类中都有很多相同的部分，因此用宏展开，代码更为简洁

```cpp
//借鏡 MFC macros.
// DECLARE_POOL_ALLOC -- used in class definition
#define DECLARE_POOL_ALLOC()                                           \
public:                                                                \
  void *operator new(size_t size) { return myAlloc.allocate(size); } \
  void operator delete(void *p) { myAlloc.deallocate(p, 0); }        \
                                                                       \
protected:                                                             \
  static allocator myAlloc;

// IMPLEMENT_POOL_ALLOC -- used in class implementation file
#define IMPLEMENT_POOL_ALLOC(class_name) \
  allocator class_name::myAlloc;

  // in class definition file
  class Foo
  {
    DECLARE_POOL_ALLOC()
  public:
    long L;
    string str;

  public:
    Foo(long l) : L(l) {}
  };
  //in class implementation file
  IMPLEMENT_POOL_ALLOC(Foo)

  //  in class definition file
  class Goo
  {
    DECLARE_POOL_ALLOC()
  public:
    complex<double> c;
    string str;

  public:
    Goo(const complex<double> &x) : c(x) {}
  };
  //in class implementation file
  IMPLEMENT_POOL_ALLOC(Goo)
```

# P16. new handler

- 之前提到内存分配不成功时有补救措施，让使用者释放一些内存进行分配或者终止程序

```cpp
  void noMoreMemory()
  {
    cerr << "out of memory";
    abort();
  }

  set_new_handler(noMoreMemory);
      /*
    int* p = new int[100000000000000];   //well, so BIG!
    assert(p);

    p = new int[100000000000000000000];  //[Warning] integer constant is too large for its type
    assert(p);
*/

```

# P17. vc6 malloc()

- debug 模式下 malloc 内存布局

cookie |
:-:|
 debug header |
 user block |
 debug tail |
 pad |
 cookie|

- 上下cookie 记录使用的分配内存的大小和使用情况，pad 用来补齐使申请的内存块为16的倍数，获得的指针指向用户区块。

# P18. VC6 标准分配器之实现 P19.BC5 分配器

- VC6 中的 `allocator` 仅仅是调用 `::operator new()` 和 `::operator delete()`,没有任何特殊的设计

- vc6 中的容器都是默认使用这个分配器以元素为单位完成内存分配，因此存在着大量的额外cookie开销。

- BC5 的分配器行为与VC6 一致

# P20. G2.9 标准分配器之实现

- G2.9 中的`std::allocator`的行为与 VC6 和 BC5 完全一致，应该只是出于兼容的考虑

- G2.9的`std::allocator`并没有在自带的STL中使用，容器的默认分配器是 `alloc`,接受的是字节大小而不是元素个数。

# P21. G2.9 std::allo vs. G4.9 __pool_alloc

- G4.9标准库中有许多 extented allocators, 包含 __pool_alloc 

- 在G4.9 中 `std::alloc` 改名为 `__pool_alloc`

```cpp
//G4.9
vector<string,__gnu_cxx::__pool_alloc<string>> vec;

//G2.9
vector<string,std::alloc<string>>
```

# P22. G4.9 pool allocator 例用

- G4.9 的标准分配器也和之前的没什么不同（应该是标准库的规定）

- 在创建大量的obj时，推荐使用 __pool_alloc，减少cookie，提高内存利用率

```cpp
/*
template<typename Alloc> 
void cookie_test(Alloc&& alloc, size_t n)  //由於呼叫時以 temp obj (Rvalue) 傳入，所以這兒使用 &&. 只是隨意之下的搭配
                                            //使用 &&，那麼呼叫時就不能以 Lvalue 傳入.  
*/
  //上述, pass by Rvalue reference 是 OK 的.
  //但我不想那麼標新立異, 就改用 pass by value 吧
  template <typename Alloc>
  void cookie_test(Alloc alloc, size_t n)
  {
    typename Alloc::value_type *p1, *p2, *p3; //需有 typename
    p1 = alloc.allocate(n);            //allocate() and deallocate() 是 non-static, 需以 object 呼叫之.
    p2 = alloc.allocate(n);
    p3 = alloc.allocate(n);

    cout << "p1= " << p1 << '\t' << "p2= " << p2 << '\t' << "p3= " << p3 << '\n';

    alloc.deallocate(p1, sizeof(typename Alloc::value_type)); //需有 typename
    alloc.deallocate(p2, sizeof(typename Alloc::value_type)); //有些 allocator 對於 2nd argument 的值無所謂
    alloc.deallocate(p3, sizeof(typename Alloc::value_type));
  }

  cookie_test(std::allocator<int>(), 1);        //相距 10h (表示帶 cookie)
  cookie_test(__gnu_cxx::malloc_allocator<int>(), 1); //相距 10h (表示帶 cookie)
  cookie_test(__gnu_cxx::__pool_alloc<int>(), 1);    //相距 08h (表示不帶 cookie)

```

# P23. std::alloc 运行模式

- std::alloc 包含16条链表，负责8 ~ 16*8字节的数据，大于128字节的数据则有malloc服务

- 每条链表每次申请2\*20个单元的数据（这里的20是经验值），前20个单元用于本条链表，后20个用作储备，如第一次在32个字节的单元申请2*20个空间，当需要用到64字节数据时，可以先取用这些储备单元。

- embedded pointers 嵌入式指针，使用内存块的前4个字节作为单向链表的指针

# P24. G2.9 std::alloc 运行一瞥01-05

- 申请到的内存都先放在pool中（储备区），用指针指明头尾

- 从pool中切割出来准备挂在list上的区块数量总是在1-20之间

- 多次申请内存为例

  - 申请32bytes内存时，由于pool为空，先向pool中注入32\*20\*2+RoundUp(0\>>4)=1280，从中切出一块给客户，剩下的19个区块挂在list#3上，余下640在pool中备用（RoundUp是追加量，每次越要越多）

  - 申请64bytes，由于pool还有余量，故将pool分为640/64=10个区块，第一个给客户，剩下9个挂在list#7上（共申请1280，pool为0个）

  - 申请96bytes,由于pool为空,因此注入（malloc申请）96\*20\*2+RoundUp(1280>>4),第一个给客户，19个给list#11（累计5200，pool：2000）

  - 申请88，由于pool有余量，分割20个，第一个给客户，19个给list#10（累计5200，pool：240）

# P25. G2.9 std::alloc 运行一瞥06-10

- 续上一节例子

  - 连续申请3次88，直接由list#10取出给客户（累计5200，pool：240）

  - 申请8，由于pool有余量，分割20个，第一个给客户，19个给list#0（累计5200，pool：80）

  - 申请104，list#12没有区块，pool余量不足一个，于是先将余下的80个给list#9（**碎片处理**），然后注入（malloc）104*20*2+RoundUp(5400>>4)，分割第一个给客户，19个给list#12（累计申请9688，pool：2408）

  - 申请112，有余量，分割20个，第一个给客户，19个给list#13（累计申请9688，pool：168）

  - 申请48，有余量，分割3个区块，第一个给客户，2个挂在list#5（累计9688，pool：24）

# P26. G2.9 std::alloc 运行一瞥11-13

- 续上一节例子，观察系统的边界行为，修改源码将系统内存设置为10000，

  - 此时申请72，list#8为空，由于pool余量不足一个，于是先把余下24挂在list#2。但是现在已经无法申请到内存，因此需要使用大于list#8的空闲空间，因此取得list#9填会pool，再切出72给客户（累计申请9688，pool：8）

  - 再申请72，list#8为空，pool余量不足一个，于是先把剩下的8个挂在list#0，从最接近的list#10取出一块填到pool，再切出72给客户。（累计9688，pool：16）

  - 此时再申请120，系统已经无法满足需求，但是还有许多未使用的链表，系统还剩余312的空间。

- 合并未使用的链表难度极高，留下一部分内存给别的程序

# P27-29. G2.9 std::alloc 源码剖析

- 第一级分配器主要来处理之前提到的set_new_handler，与内存分配关系不大

- 主要是对照代码讲解

# P30-31. G2.9 std::alloc 观念大整理

- 大部分与第29讲视频相同

- deallocate 没有free，源于设计的缺陷，在一开始没有记录分配的指针，在后面loki部分会有改进

# P32-41. vc6和vc10的malloc比较 （进入第三讲）

- vc6和vc10 main函数启动前的行为，与C++startup课程讲的相同

- debug模式下内存分配会加上额外开销来记录文件行号等信息

- 所有malloc申请的内存都被登记在sbh之中

# P42-44. vc6内存管理free(p)

- 先根据指针的地址判断落在哪一个header（一共16个header，每个header 管理 1M的内存）

- 再讲地址减去header指针 初一32得到group分组，再找出对应的链表

- 将1M内存分成32个group，这种一个小块小块的操作更有可能进行整块内存的回收

- 判断全回收：每个group头部有个计数器，每次malloc就会增加，每次free就会减少，当计数器为0时就全回收,变成最初始的8个page

- 全回收后并没有马上还给操作系统，有两个全回收才还给操作系统一个

- 虽然 `std::allocator` 、`CRT(malloc/free)` 甚至 `heapAlloc`都是类似的使用链表来来进行内存的维护，但是这并不重复，因为`std::allocator`不能预设`CRT(malloc/free)`已经进行了内存管理，`CRT(malloc/free)`也不能预设系统API`heapAlloc`内部有内存管理。

- 但是在vc10中 由于是windows平台，malloc的行为就是直接调用 `heapAlloc` 没有sbh管理

# P45. 上中下3个classes分析

- loki allocator 上中下三层

- 底层 **Chunk**

  - **pData_** : unsigned char* (指针指向一块内存)
  - **firstAvailableBlock_** : unsigned char (第一个可用的区块编号)
  - **blockAvailavle_**: unsigned char (剩余可用的区块数量)

- 中层 **FixedAllocator**
  
  - **chunks_** : vector\<Chunk> (动态数组存放底层)
  - **allocChunk_**: Chunk*
  - **deallocChunk_**: Chunk*  (两根指针用来维护)

- 上层 **SmallObjAllocator**

  - **pool_**: vector\<FixedAllocator> （动态数组存放中层）
  - **pLastAlloc**：FixedAllocator*
  - **plastDealloc**: FixedAllocator*
  - chunkSize: size_t
  - maxObjectSize: size_t

# P46. loki_allocator行为图解

- 申请一块内存分成64个块，借用每块的头部保存下一个index，

- 刚开始时 `firstAvailableBlock`是0，第一个内存空间的头部是1即下一个index

- 当客户需要内存时，先记录`firstAvailableBlock`所在区块的地址作为返回的指针，再取得当前第一个头部数据作为`firstAvailableBlock`的更新值，同时`blockAvailavle_`减一

- 当客户归还内存时，将`firstAvailableBlock`中的值填入释放的地址中，同时`firstAvailableBlock` 和 `blockAvailavle_` 自增
