---
title: C++笔记
date: 2020-11-24 08:50:30
tags:
toc: true 
---
<!-- toc -->

# 资源

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

# 资源

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
      //...
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
	   //... plus
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
    //...
};

class Foo
{
  //...
  void method(void){}
}

shared_ptr<Foo> sp(new Foo);

Foo f(*sp);// * 解引用

sp->method();//  转换为 px->method(),  sp-> 重载后 ->会继续作用 即 (sp.operator->())->method 也就是px->method
```

* \* 称之为 解引用
* `operator->()`重载后 会继续作用 `->`

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


