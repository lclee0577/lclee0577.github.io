---
title: C++笔记
date: 2020-11-24 08:50:30
tags:
toc: true
categories: C++
---
<!-- toc -->
# 上篇 

视频网址 https://www.bilibili.com/video/BV1aW411H7Xa?p=1

# P3. 构造函数

``` cpp {.line-numbers}
class complex
{
  public:
    complex (double r = 0, double i = 0): re (r), im (i) { } \\使用: 初始化变量  无需再 {···} 内赋值
    complex& operator += (const complex&); 
    complex& operator -= (const complex&); 
    complex& operator *= (const complex&); 
    complex& operator /= (const complex&); 
    double real () const { return re; }
    double imag () const { return im; }

  private:
    double re, im;
  friend complex& __doapl (complex *, const complex&); 
  friend complex& __doami (complex *, const complex&); 
  friend complex& __doaml (complex *, const complex&); 

}

inline complex&
__doapl (complex* ths, const complex& r)
{
  ths->re += r.re; 
  ths->im += r.im; 
  return *ths; 
}

inline complex&
complex::operator += (const complex& r)
{
  return __doapl (this, r); 
}

```

* 构造函数没有返回值
* 使用`初始列(initialization list)`初始化默认参数 (第`4`行) 更有效。 在大括号内赋值(assignment)就等于放弃了初始列这种
* 构造函数可以有多个 ,根据参数的不同进行重载(overloading),虽然函数名相同,但是编译后的函数名包含参数

# P4.参数传递和返回值

* 大部分构造函数在public区,例外:单例(Singleton): 构造函数(constructor) 放在 private 区 只允许创建一份

## 常量成员函数

* 不改变成员的函数要在函数声明圆括号()后,函数体花括号{}前,用const修饰 (第`9`,`10`行)

## 返回值传递 

* 值传递 pass by value, 引用传递 pass by reference (to const)

  > 值传递 所有参数的值压入函数的栈(如果数据量很大,将很耗资源)
  > 引用传递 (底层是指针实现) 效率更高,但是没有指针的缺点。尽量传引用
  > 当不希望传递的引用被修改，改为常量引用 (第 `5` 行)

* 返回值 返回引用 (第 `5` 行)

## `friend`(友元) 

* 友元函数可以操作私有变量(`14`行)
* 相同class的各个objects互为friends(友元) 一个obj可以操作另一个obj的私有变量

## class body外的各种定义

* 返回对象的是函数内创建的变量时,要用return value (引用在函数结束后被销毁)

# P5.操作符重载和临时变量

* 区别于C语言,C++中操作符也被认为是一种函数,可以进行重载.这就可以对新的类型进行加减乘除操作,例如复数的运算.
* 所有的成员函数默认都有一个`this`指针,指向调用自己的实例.

## operator overloading(操作符重载-1,成员函数) this

```cpp
inline complex&
__doapl (complex* ths, const complex& r)
{
  ths->re += r.re;
  ths->im += r.im;
  return *ths;
}

inline complex&
complex::operator += (const complex& r)
{
  return __doapl (this, r);
}
```

* **传递者**无需知道**接收者**是以 **reference** 的形式接收 (`20-26行`函数`__doapl()` return *this; 但是它的声明却是引用) 
* 操作符重载要考虑连续调用, 因此声明时, 返回类型不能是`void`, 如`c3 += c2 += c1;`

## operator overloading(操作符重载-2, 非成员函数) 无this

`c2 = c1 + c2`这种情况, 同时要考虑复数+实数, 实数+复数, 复数+虚数等情况进行重载

``` cpp
inline double
imag (const complex& x)
{
  return x.imag ();
}

inline double
real (const complex& x)
{
  return x.real ();
}

inline complex
operator + (const complex& x, const complex& y)
{
  return complex (real (x) + real (y), imag (x) + imag (y));
}
```

* 注意 这些函数一定要return by value 不能by reference, 因为返回的是临时变量

---

``` cpp
ostream&
operator << (ostream& os, const complex& x)
{
  return os << '(' << real (x) << ',' << imag (x) << ')';
}
```

* 输出函数重载, 参数os没有使用const是因为 输出字符会改变os的状态
* 虽然不在乎`cout` 的返回值, 但是考虑到连续输出的情况, 即 `cout<< c1 << c2` 因此重载函数的返回类型应是`ostream&`

## 回顾

> 1. 数据一定放在private区, 
> 2. 参数和返回值尽量以引用来传递(根据情况要不要加const)
> 3. 不修改数据的函数要用 `const` 修饰
> 4. 使用 `initialization list` 设置初值

# P7三大函数 拷贝构造 拷贝赋值 析构 

```cpp {.line-numbers}
class String
{
public:                                 
   String(const char* cstr=0); 
   String(const String& str); 
   String& operator=(const String& str); 
   ~String(); 
   char* get_c_str() const { return m_data; }
private:
   char* m_data; 
}; 

```

## Classes 的两个经典分类,是否包含指针

* class 内带指针,需要自己实现拷贝构造和拷贝赋值
* 拷贝构造:构造函数接受的参数类型是这个类型本身(`第5行`)
* 拷贝赋值:操作符重载来实现.接受的类型还是本身(`第6行`)
* 析构函数:`~`构造函数() 离开作用域后删除
* class内含有指针,多半需要动态分配内存,因此需要自己析构

## 拷贝赋值函数

```cpp
inline
String& String::operator=(const String& str)
{
   if (this == &str)
      return *this;

   delete[] m_data;
   m_data = new char[ strlen(str.m_data) + 1 ];
   strcpy(m_data, str.m_data);
   return *this;
}
```

> 经典做法:
> 1. **先检测自我赋值, 否则下一步清空数据将导致没有数据拷贝**
> 2. **再清空自己的数据**
> 3. **再创建需要的空间**
> 4. 拷贝数据

# P8 堆、栈与内存管理

## stack 栈

  stack是存在于某个作用域(scope)的一块内存空间(memory space). 例如当你调用函数时, 函数本身会形成一个stack来放置他所接受的参数, 以及返回的地址. 函数本体被声明的变量都来自stack.(离开作用域自动释放)

## heap 堆

heap是有操作系统提供的一块global的内存空间, 程序可动态分配(dynamic allocated)从某种获得若干区块(block).(需要手动释放 delete)

## 静态对象 static local object

函数内用**stack**修饰变量, 其声明在作用域结束后仍然存在, 直到整个程序结束. 与之相同的还有全局变量(声明在所有函数之外)

---

* new: 先分配memory, 在调用ctor, 底层为:

> 1. 计算class的大小
> 2. 再调用malloc分配内存(默认返回 `void*` )
> 3. 修改指针类型
> 4. 传给构造函数

* delete: 先调用dtor, 再释放memory, 底层为:

> 1. 先调用析构函数, 删除该类型内部动态分配的内存
> 2. 再调用delete删除该类型变量的指针( `free` 实现))

## 动态分配所得的内存块

* 获得的内存的大小一般是16的倍数(不够补齐), 前后有cookie来标记内存的长度, 用cookie的最后一位表示是创建还是回收
* array new 一定好array delete `String* p = new String[3];` 之后一定要`delete []p;`这样才会多次调用dtor 否则只释放了数组的第一个对象.

# P10 拓展补充 类模板, 函数模板等

## static补充

``` cpp
class Account{
  public:
    static double m_rate;
    static void set_rate(const double& x) { m_rate = x;}
};
double Account::m_rate = 8.0;

int main(){
  Account::set_rate(5.0);//通过class name调用
  Account a;
  a.set_rate(7.0);//通过 object 调用
} 
```

* 类内的静态函数只能处理类内静态变量
* 类内静态变量要在所有函数之外先定义

# P11. 组合和继承

## Composition 复合, 表示has-a

* 类内有一个别的类型
* Adapter 适配器模式 一个类经过另一个类的稍加修改实现功能, 例如用deque双端队列实现queue队列
* 编译复合类时, 构造由内而外, 先调用内部类默认的构造函数, 在调用自身的ctor, 析构由外而内.

## Delegation 委托, composition by reference

* 两个类用指针相连, 两个类生命周期不同步
*  Pointer to Implementation    PIMPL模式
    - 有的成员指针，将指针所指向的类的内部实现数据进行隐藏,
    - 降低模块的耦合
    - 降低编译依赖，
    - 提高编译速度接口与实现分离，提高接口的稳定性

## Inheritance 继承 表示is-a

最常见的是public继承

(父类)base class 的dtor必须是virtual, 否则会出现undefined behavior

# P10. 虚函数和多态

## Inheritance with **virtual** functions

* non-virtual函数: 不希望子类 (derived class)重新定义(override, 覆写)它
* virtual 函数: 希望子类重新定义它, 默认已有定义
* pure virtual函数: 子类一定要重新定义, 默认没有定义

``` cpp
class Shape{
  public：
    virtual void draw() const = 0;// pure virtual
    virtual viod error(const std::string& msg);// impure virtual
    int objectID() const;//non-virtual
    ···
}；

class Rectangle:public Shape{···};
class Ellipse:public Shape{···};

```

**设计模式-模板设计(template method)**: 父类设计好大部分程序, 留下其中几个虚函数让子类实现, 使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤, 而将一些步骤延迟到子类中实现

## Delegation(委托) + Inheritance(继承)

``` cpp
   class Subject
  {
      int m_value;
      vector<Observer*>m_views;
    public:
      void attach(Observer* osb)
      {
        m_views.push_pack(obs);
      }
      void set_val(int value)
      {
        m_value = value;
        notify()
      }
      void notify()
      {
        for(int i = 0; i<m_views.size(); ++i)
          m_views[i]->update(this,m_value);
      }
  };

  class Observer
  {
    public:
      virtual void update(Subject* sub, int value) = 0;
  }

```

* 观察者模式: 当对象间存在一对多关系时, 则使用观察者模式(Observer Pattern). 比如, 当一个对象被修改时, 则会自动通知依赖它的对象

# P13. 委托相关设计

## 设计模式-组合设计(Composite)

* 把一组相似的对象当作一个单一的对象. 组合模式依据树形结构来组合对象, 用来表示部分以及整体层次

``` cpp
  class Component
  {
      int value;
    public:
      Component(int val) { value = val; }
      virtual void add( Component* ) { }
  };

  class Primitive: public Component
  {
    public:
      Primitive(int val): Component(val) {}
  };

  class Composite: public Component
  {
      vector <Component*> c;
    public:
      Composite(int val): Component(val) { }
      void add(Component* elem) 
      {
        c.push_back(elem);
      }
  };  
```

## 设计模式-原型模式

父类需要调用子类, 每一个子类都有一个 clone 方法, 通过拷贝这些原型创建新的对象

---

---

# 下篇

C++ 程序设计(Ⅱ) https://www.bilibili.com/video/av19151507?p=1&t=62

## P2.conversion function, 转换函数

``` cpp
class Fraction
{
  public:
    Fraction(int num, int den=1)
    : m_numerator(num), m_denominator(den) {}

    operator double() const{//conversion function: fraction -> double
      return (double)(m_numerator / m_denominator); 
    }
  private:
    int m_numerator;   //分子
    int m_denominator; //分母
}
 
Fraction f(3,5);
double d = 4 + f; //编译器先找有没有重载 fraction + int类型,再找fraction能不能转成double
                  //调用operator double() 将 f 转为 0.6
```

## non-explicit-one-argument ctor

``` cpp
class Fraction
{
  public:
    Fraction(int num, int den=1) 
      : m_numerator(num), m_denominator(den)
        
    Fraction operator+(const Fraction& f) 
    {
      //···
      return f; 
    } 
  private:   
    int m_numerator;    //
    int m_denominator;  //
};
Fraction f(3,5);
Fraction d2 = f + 4;
```

* 分数类中 ctor den已有默认值1, 可以只需输入num的值 -- one argument
* 因为den已有默认值1,         `Fraction d2 = f + 4;`中调用 non-explicit ctor 将4 转为Fraction(4, 1), 然后调用 `operator +`   

``` cpp
class Fraction
{
public:
	explicit Fraction(int num, int den=1) 
	  : m_numerator(num), m_denominator(den){  }
	
 	operator double() const { 
      return (double)m_numerator / m_denominator; 
 	}
 	
 	Fraction operator+(const Fraction& f) {  
	   //··· plus
	   return f; 
	} 

private:   
   int m_numerator;    //
   int m_denominator;  //
};
Fraction f(3,5);
Fraction d2 = f + 4;
```

* 若cotr前没有explicit,         `Fraction d2 = f + 4;`存在歧义 ambiguous, 可以都变成分数, 也可以都变成double
* ctor前有explicit, 意为没有明确写构造函数时, 不要转换(取消隐式转换). 所以`Fraction d2 = f + 4;`中f被转成double后相加, 无法再转回Fraction, 无法通过编译.

# P4.pointer-like classes, 关于智能指针

``` cpp
template<class T>
class shared_ptr
{
  public:
    T& operator* () const
    { return *px; }

    T* operator->() const
    { return px; }

    shared_ptr(T* p) : px(p) { }
  
  private:
    T*    px;
    long* pn;
    //···
};

class Foo
{
  //···
  void method(void){}
}

shared_ptr<Foo> sp(new Foo);

Foo f(*sp);// * 解引用

sp->method();//  转换为 px->method(),  sp-> 重载后 ->会继续作用 即 (sp.operator->())->method 也就是px->method
```

* \* 称之为 解引用
*  `operator->()`重载后 会继续作用 `->`

## pointer-like classes, 迭代器

* 迭代器也是一种智能指针

``` cpp
reference operator*() cosnt {return (*node).data;}
pointer operator->() const {return &(operator*());}
```

* 对迭代器解引用就是取出节点的数据
* 对迭代器访问方法, 就是先取出数据在访问方法

# P5.function-like classes, 仿函数

* 仿函数都是某个类重载`()`, 使其具有函数的特性, 标准库的仿函数详见STL课程

# P6.namespace, 经验谈

* 防止起名冲突, 用namespace加以区分

# P7 and P8 .class template 类模板, function template 函数模板

* 复习之前课程, 没有新内容

# P9. 成员模板

* 类的成员函数是模板函数.

```cpp {.line-numbers}
namespace jj01
{
  class Base1 {  }; 
  class Derived1: public Base1 {  }; 	
  class Base2 {  }; 
  class Derived2: public Base2 {  }; 

  template <class T1, class T2>
  struct pair {
    typedef T1 first_type; 
    typedef T2 second_type; 

    T1 first; 
    T2 second; 
    pair() : first(T1()), second(T2()) {}
    pair(const T1& a, const T2& b) : first(a), second(b) {}

    template <class U1, class U2>
    pair(const pair<U1, U2>& p) : first(p.first), second(p.second) {}
  }; 

  void test_member_template()
  {
    cout << "test_member_template()" << endl;
    pair<Derived1, Derived2> p; 
    pair<Base1, Base2> p2(pair<Derived1, Derived2>()); 
    pair<Base1, Base2> p3(p);   
    //Derived1 will be assigned to Base1; Derived2 will be assigned to Base2.
    //OO allow such assignments since of "is-a" relationship between them.  

    //!	pair<Derived1, Derived2> p4(p3); 

    //error messages as below appear at the ctor statements of pair member template
    // [Error] no matching function for call to 'Derived1::Derived1(const Base1&)'
    // [Error] no matching function for call to 'Derived2::Derived2(const Base2&)'

    Base1* ptr = new Derived1; //up-cast 
    shared_ptr<Base1> sptr(new Derived1); //simulate up-cast
    //Note: make sure your environment support C++2.0 at first.
  }
}
```
- 通常`T1` 是`U1`的父类,`T2`是`U2`的父类.可以实现继承和多态的巧妙使用,如`26`行.反正则不行,如`31`行.
- 父类的指针可以指向子类 如`38`行

# P10.specialization, 模板特化
在模板类中指定某一个类型进行表述
```cpp {.line-numbers}
template <class Key>
struct hash{ };

template <>
struct hash<char>{
  size_t operator()(char x) const { return x;}
};
```
# P11.partial specialization, 模板偏特化
- 个数的偏,指定某个参数
```cpp
template<typename T, typename Alloc=···>
class vector
{
  ···
};

template<typename Alloc=···>
class vector<bool,Alloc>
{
  ···
};
```
在c++中,最小的单元是`char`,用来表示`bool`空间利用率不高，因此需要重新设计针对`bool`的`vector`.

- 范围上的偏,将范围由任意类型变为指针类型
```cpp{.line-numbers}
 template <typename T>
 class C
 {
   ···
 }

 template <typename T>
 class C<T*>
 {
   ···
 };

C<string>  obj1;//调用第2行
C<string*> obj2;//调用第8行
```

# P12.template template parameter,模板模板参数
- 模板的参数也是模板
```cpp
template<typename T,
        template<typename T>
        class Container
        >
class XCls
{
    private:
      Container<T> c;
    public:
      ···
};

template <typename T>
using Lst = list<T,allocator<T>>;//在下一门课程讲解
//XCls <string,list> mylst1; 错误，list 接受两个参数要用上一行所示才能运行
XCls<string,Lst>; mylst2
```
- 私有变量 `mylst2.c`的类型就是 `list<string>`
- 下面这个不是模板模板参数
```cpp
template <class T, class Sequence = deque<T>>
class stack{
  ···
}

stack<int,list<int>> s2;
```
若前面是`int`,后面必须是`list<int>`,因此不是模板模板参数,

p13.介绍标准库, 尽量每个函数都去用一遍

# P14. 三个主题
## variadic template (C++11) 数量不定模板参数
- 注意: `...`是语法的一部分,表示这是一个包(pack)
```cpp{.line-numbers}
void print(){ }

template <typename T, typename... Types>
void print(const T& firstArg, const Types&... args)
{
  cout<< firstArg<<endl;
  print(args...);//递归调用
}
print(7.5,"hello",bitset<16>(377),42)
// 7.5
// hello
// 0000000101111001
// 42
```
- 当执行第`9`行打印，参数被分成`7.5`和后面的一包(pack)，调用第`4`行，打印第一个参数，如此递归至最后一个发现没有参数，调用第`1`行 什么也不做，停止递归。
## auto (C++11)
 - auto 语法糖 用于类型推导
```cpp {.line-numbers}
lsit<string> c;
···
list<string>::iterator ite;
ite = find(c.begin(),c.end(),target);
```

由于写出迭代器的类型比较繁琐可以使用auto进行类型推导
```cpp
auto ite = find(c.begin(),c.end(),target);//代替上面3,4行
```

## range-base for (C++11)
语法糖 方便`for`循环 语法如下
```cpp
for(decl:coll)//decl-元素，coll-容器。将容器中的元素一个个取出来
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
# P15 reference
```cpp
int x = 0;
int* p = &x; 
int& r = x;//r代表x，现在r，x都是0
int x2 = 5;

r = x2;// r不能重新代表其他变量，这句话相当于给r代表的x赋值，即x = x2，现在r，x2都是5
int& r2 = r;//r2代表r，相当于代表x，所以r2是5
```
- 引用智能绑定一次
- `sizeof(r) == sizeof(x)`
- `&x == &r` 
- `object` 和其`reference`的大小地址都相同(编译器帮助实现，其实是假象)

## reference 的常见用途
- reference 通常不用于声明变量，而用于参数类型(parameter type)和返回类型(return type)
```cpp {.line-numbers}
void func1(Cls* pobj){ pobj->xxx(); }
void func2(Cls  obj ){  obj->xxx(); }
void func3(Cls& obj ){  obj->xxx(); }

··· 
Cls obj;
func1(&obj);
func2(obj);
func3(obj);
```
- 在函数内部传值和传引用调用函数的写法相同
- 在外部传入参数，传值和传引用的接口也相同


```cpp
double imag(const double& im) {···}
double imag(const double  im) {···}
```
- 这两个函数不能共存，因为它们的函数签名相同
- 函数括号后的const 也是函数签名的一部分，函数签名不包括前面的返回类型

# P16. 复合 +  继承关系下的构造和析构
复习之前课程

# P17.关于vptr和vtbl
- 带着虚函数的类会比类内生命数据加起来还要大（因为虚函数要占用一个指针）
- 类的继承：继承了成员和函数的调用权
- 虚函数的虚指针vptr指向虚表vtbl，虚表里放着要调用的虚函数的地址
- 当改写继承的虚函数就是修改了虚标里的函数指针，
- 多态中 动态绑定：1.通过指针指向某类（某个子类）；2.指针向上转型；3.调用虚函数
- 静态绑定: 调用函数直接跳转到某个地址 
- 动态绑定的C描述`(*p->vptr[n])(p)`

# 关于this 
- 使用动态绑定重新讲解`Template Method`
```cpp {.line-numbers}
CDocument::OnFileOpen()
{
  ···
  Serialize();//它的定义是个虚函数，由子类实现
  ···
}
virtual Serialize();

class CMyDoc:public CDocument
{
  virtual Serialize(){···}
}

main()
{
  CMyDoc myDoc;
  ···
  myDoc.OnFileOpen();//
}
```
- 虽然子类调用的是从父类继承的函数(`18`行)，但是其中的虚函数已经在子类改写过(`11`行),就会以动态绑定的方式执行子类重新定义过的虚函数

# P18.关于Dynamic Binging(1)
## 谈谈 const
- const 只能加在成员函数参数括号之后，函数体花括号之前，意为没有修改类成员
- const obj 不能调用 non-const member function
```cpp
const String str("hello world");
str.print()
```
- 若设计`string::print()` 没有指明`const` 那么`str.print()`无法通过编译。
- 当成员函数的`const`和`non-const`版本同时存在时，`const obj`,只能调用`const`版本，`non-const obj`,只能调用`non-const`版本

# P19.关于Dynamic Binging
- 继承关系:`A<-B<-C`
  ```cpp {.line-numbers}
  B b;
  A a =(A)b;
  a.vfunc1();//静态绑定
  ```
  - 子类强制类型转换,用对象调用虚函数,不是指针。因此调用的还是`A::vfunc1()`
  ---
  ```cpp
  A* pa = new B;
  pa -> vfunc1();

  pa = &b;
  pa -> vfunc1();
  ```
- 符合动态绑定的三个特征：指针指向，向上转型，虚函数
# P20,21,22. 关于new,delete及其重载
- 可以对new 和delete 进行全局重载  `::operator new`,`::operator delete`,`::operator new[]`,`::operator delete[]` 注意影响所有函数
- 也可以仅重载member operator new/delete new\[]/delete[]
- 接管 `new`和`delete`的目的是为了实现内存池
- 以下写法强制采取全局的new和delete ,忽略内部的重载。
```cpp
Foo* pf = ::new Foo;
::delete pf;
```

# P23,24.重载new(),delete()
- 可以重载`class member operator new()` 写出多个版本，但前提是每个版本都要声明独特的参数，其中第一个参数必须是`size_t`类型,其余参数以new所指定的所指定的`placement arguments`为初值，出现于`new(···)`, 例如`Foo* pf= new(300,'c')Foo;`
- 同样的也可以重载`delete` 但是`delete`只有在对应的 `operator new`的构造函数失败时才调用。
```cpp
Rep* p = new(extra)Rep;
```
标准库字符串重载了placement new，申请大于字符串长度的空间
|        |< -  extra   ->|     
---------|---------------|
 Rep     |   string 内容 |
前面的Rep用于引用计数