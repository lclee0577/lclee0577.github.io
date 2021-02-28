---
title: C++设计模式
date: 2021-02-22 10:25:20
tags:
categories:
---

# P1. 设计模式简介

- 设计模式：“每一个模式描述了一个在我们周围不断重复发生的问题，以及该问题的解决方案的核心。这样，你就能一次又一次地使用该方案而不必做重复劳动。"——Christopher Alexander

- 软件设计的金科玉律 ——复用

- 各种因素的变化造成软件设计十分复杂,常见做法有分解和抽象。

  - 分解: 将将大问题分解为多个小问题，将复杂问题分解为多个简单问题

  - 抽象: 忽视非本质细节，处理泛化和理想化的对象模型

  - 举例：假设需要实现一个画布，需要划出直线和矩形
    - 按照分解的思路，需要写出`Line`和`Rect`两个类，再在画布上进行绘制
    - 按照抽象的思路，`Line`和`Rect`两个类继承自抽象类`Shape`，通过override虚函数实现`draw()`自身的绘制，画布上`Shape`类的多态调用`draw()`即可。
  
  - 就现在的情况看，两种写法并没有什么差别，但当需要增加`Circle`类时，
    - 分解法不仅要新增`Circle`类，还要在画布上新增绘制函数
    - 抽象法只需新增`Circle`类即可
    - 分解法改动地方多，因此需要编译多个文件，抽象法只需编译新增的文件即可

# P2. 面向对象设计原则

## 重新认识面向对象

- 理解隔离变化
  - 从宏观层面来看，面向对象的构建方式更能适应软件的变化，能将变化所带来的影响减为最小
- 各司其职
  - 从微观层面来看，面向对象的方式更强调各个类的“责任”（多态中接口一致，实现不同）
  - 由于需求变化导致的新增类型不应该影响原来类型的实现——是所谓各负其责
- 对象是什么？
  - 从语言实现层面来看，对象封装了代码和数据。
  - 从规格层面讲，对象是一系列可被使用的公共接口。
  - 从概念层面讲，对象是某种拥有责任的抽象。

## 面向对象八大原则

- 依赖倒置原则（DIP）
  - 高层模块(稳定)不应该依赖于低层模块(变化)，二者都应该依赖于抽象(稳定) 。
  - 抽象(稳定)不应该依赖于实现细节(变化) ，实现细节应该依赖于抽象(稳定)。（实现变化的隔离）

- 开放封闭原则（OCP）
  - 对扩展开放，对更改封闭。
  - 类模块应该是可扩展的，但是不可修改（应该增加一些东西而不是对原来的东西进行修改）

- 单一职责原则（SRP）
  - 一个类应该仅有一个引起它变化的原因。
  - 变化的方向隐含着类的责任。(类中方法太多时候需要分割责任)

- Liskov 替换原则（LSP）
  - 子类必须能够替换它们的基类(IS-A)。（子类不应该否定父类的方法，否则不应该继承）
  - 继承表达类型抽象。

- 接口隔离原则（ISP）
  - 不应该强迫客户程序依赖它们不用的方法。
  - 接口应该小而完备。（开放给子类的用protected，仅自己用的protected，给别人的的才public，给别人用的接口要保持稳定）

- 优先使用对象组合，而不是类继承
  - 类继承通常为“白箱复用”，对象组合通常为“黑箱复用” 。
  - 继承在某种程度上破坏了封装性，子类父类耦合度高。
  - 而对象组合则只要求被组合的对象具有良好定义的接口，耦合度低。

- 封装变化点
  - 使用封装来创建对象之间的分界层，让设计者可以在分界层的一侧进行修改，而不会对另一侧产生不良的影响，从而实现层次间的松耦合。

- 针对接口编程，而不是针对实现编程
  - 不将变量类型声明为某个特定的具体类，而是声明为某个接口。
  - 客户程序无需获知对象的具体类型，只需要知道对象所具有的接口。（抽象类和抽象类的调用函数）
  - 减少系统中各部分的依赖关系，从而实现“高内聚、松耦合”的类型设计方案。

# P3. 模板方法

## 重构获得模式 Refactoring to Patterns

- 现代软件设计的特征是“需求的频繁变化”。设计模式的要点是“寻找变化点，然后在变化点处应用设计模式，从而来更好地应对需求的变化”
- “什么时候、什么地点应用设计模式”比“理解设计模式结构本身”更为重要。

- 没有一步到位的设计模式。敏捷软件开发实践提倡的“Refactoring to Patterns”从重构到模式是目前普遍公认的最好的使用设计模式的方法。（分析现有程序违背那些设计原则，一步一步分析得到某种模式）

## 重构关键技法

- 静态 → 动态
- 早绑定  →  晚绑定
- 继承  →  组合
- 编译时依赖 →  运行时依赖
- 紧耦合  → 松耦合

## 模板方法模式

- 对于某一项任务，它常常有稳定的整体操作结构，但各个子步骤却有很多改变的需求，或者由于固有的原因（比如框架与应用之间的关系）而无法和任务的整体结构同时实现。

- 举例：需要顺序执行5个步骤，库开发人员完成了1,3,5，剩下2,4需要有具体业务而定
  - 结构化过程则为：app开发人员完成2,4，和软件主流程1-5的调用
  - 面向对象过程：库开发人员在类内`run()`中实现调用1-5步骤，app继承基类并override步骤2，4，调用`run()`即可（这里步骤2,4是纯虚函数）
  - 注意基类的结构函数应该是虚函数（）
- 晚绑定：父类调用子类的方法

- 定义一个操作中的算法的骨架 (**稳定**)（就是上面的run（）），而将一些步骤延迟(**变化**)到子类中。Template Method使得子类可以不改变(复用)一个算法的结构即可**重定义**(override 重写)该算法的某些特定步骤。

- 注意应用设计模式程序中必须有稳定的部分，若全稳定或全不稳定就不能应用设计模式，设计模式是找到稳定与不稳定之间的分离点

## 模板方法要点总结

- 最简洁的机制（虚函数的多态性）为很多应用程序框架提供了灵活的扩展点（继承和override），是代码复用方面的基本实现结构。

- 除了可以灵活应对子步骤的变化外，“不要调用我，让我来调用你”的反向控制结构是Template Method的典型应用（父类指针调用子类实现多态）。

- 在具体实现方面，被Template Method调用的虚方法可以具有实现，也可以没有任何实现（抽象方法、纯虚方法），但一般推荐将它们设置为protected方法（这些虚函数单独调用没有意义，客户只需调用调用这个流程`run()`）

# P4. 策略模式

- 与 模板方法模式一样属于组件协作模式

- 代码中出现大量 if-else 多情况分类处理（并且有可能出现更多情况）时，极大概率可以应用策略模式

- 定义：一系列算法，把它们一个个封装起来，并且使它们可互相替换（变化）。该模式使得算法可独立于使用它的客户程序(稳定)而变化（扩展，子类化）

- 举例：计算三个国家的税率
  - 分解法：客户类中先判断是哪个国家（多个if-else），再根据各个国家进行计算
  - 策略模式法：定义税率计算策略基类，其中定义虚函数计算税率，各个国家税率继承自这个基类，客户类中只需建立基类指针调用虚函数即可

- 可以看出分解法每新增一个国家税率都需要修改客户类，不符合我们的设计原则，二策略模式仅仅新增一个新的国家税率即可

## 策略模式要点总结

- 为组件提供了一系列可重用的算法，从而可以使得类型在运行时方便地根据需要在各个算法之间进行切换

- 提供了用条件判断语句以外的另一种选择，消除条件判断语句，就是在解耦合。含有许多条件判断语句的代码通常都需要Strategy模式

# P5. 观察者模式

- 属于组件协作模式
- 当需要建立一种通知依赖关系，当一个对象状态发生改变时，所有的依赖（观察者对象）都将得到通知

- 设想如下场景： 在界面上显示任务的运行进度
  - 一般的做法是在任务内调用界面的绘图
  - 但是这样做会使任务依赖于界面，根据设计原则两者都应该依赖于抽象
  - 因此需要一个抽象通知基类，类内只含有接口。界面继承自该基类，任务类中含有该基类指针，使用指针调用接口函数进行绘图
- C++ 支持多继承，但是并不推荐继承多个类，这样会造成紧耦合，推荐的是继承一个类和多个接口，这里的通知类就是接口

- 当有多个观察者时，应在任务类内使用容器存储多个通知类指针，同时要在类内实现添加、删除观察者的成员函数。对容器进行迭代实现通知所有观察者。在界面类中添加多个观察者

## 观察者模式要点总结

- 使用面向对象的抽象，Observer模式使得我们可以独立地改变目标与观察者，从而使二者之间的依赖关系达致松耦合。（可以任意增加或删去容器内观察者数量，改变任务类程序）

- 目标发送通知时，无需指定观察者，通知（可以携带通知信息作为参数）会自动传播。

- 观察者自己决定是否需要订阅通知，目标对象对此一无所知。（界面类中是否增加观察者）

- Observer模式是基于事件的UI框架中非常常用的设计模式，也是MVC模式的一个重要组成部分。

# P6. 装饰模式

- 属于单一责任模式（并不是其他模式没有责任问题，而是这个模式在责任问题上特别明显）

- 过度的使用继承来拓展对象的功能会导致子类的膨胀，根据需求来动态实现子类拓展避免子类膨胀

- 模式定义：动态（组合）地给一个对象增加一些额外的职责。就增加功能而言，Decorator模式比生成子类（继承）更为灵活（消除重复代码 & 减少子类个数）

- 举例：有一系列流如文件流、网络流、内存流，有一系列操作如加密、缓存。若使用继承来实现这些操作则需要定义很多次继承，如加密缓存文件流→缓存文件流→文件流→流。一旦操作的种类增加，将需要写出许多子类，而且包含大量重复代码（加密和缓存的手段相同）
  
  - 流的种类和操作并不是继承关系，应该是组合关系

---

```cpp
//业务操作
class Stream{

public:
    virtual char Read(int number)=0;
    virtual void Seek(int position)=0;
    virtual void Write(char data)=0;
    
    virtual ~Stream(){}
};

//主体类
class FileStream: public Stream{
public:
    virtual char Read(int number){
        //读文件流
    }
    virtual void Seek(int position){
        //定位文件流
    }
    virtual void Write(char data){
        //写文件流
    }

};
```

- 版本一：可以看到加密流不仅继承父类，还包含一个指向父类的指针（这个可以认为是装饰模式的重要标志）

- 这里的继承是为了保持接口一致，父类指针时为了实现加密操作

```cpp

class CryptoStream: public Stream {

Stream* stream;//...

public:
CryptoStream(Stream* stm):stream(stm){

}


virtual char Read(int number){
    
    //额外的加密操作...
    stream->Read(number);//读文件流
}
//...
}
/////使用方法
FileStream* s1=new FileStream();//FileStream 继承自Stream
CryptoStream* s2=new CryptoStream(s1);
```

---

- 版本二：虽然版本一已经实现了大部分功能，但是当操作种类增加时每类内包含的一个父类指针时固定的。因此可以将指针和装饰操作抽象为一个装饰类

```cpp
class DecoratorStream: public Stream{
protected:
    Stream* stream;//...
    
    DecoratorStream(Stream * stm):stream(stm){
    
    }
    
};

class CryptoStream: public DecoratorStream {
public:
    CryptoStream(Stream* stm):DecoratorStream(stm){
    
    }
    
    virtual char Read(int number){
       
        //额外的加密操作...
        stream->Read(number);//读文件流
    }
    }
```

## 装饰模式要点总结

- 通过采用组合而非继承的手法， Decorator模式实现了在运行时 动态扩展对象功能的能力而且可以根据需要扩展多个功能。避免 了使用继承带来的“灵活性差”和“多子类衍生问题”

- Decorator类在接口上表现为is-a Component的继承关系，即 Decorator类继承了Component类所具有的接口。但在实现上又 表现为has-a Component的组合关系，即Decorator类又使用了另外一个Component类。（装饰类继承和组合了同一个类，这是十分少见的。继承是为了保持接口的一致，组合是为了对具体类新型包装增加新的功能）

- Decorator模式的目的并非解决“多子类衍生的多继承”问题， Decorator模式应用的要点在于解决“主体类在多个方向上的扩展 功能”——是为“装饰”的含义。

# P7. 桥模式

- 属于单一职责模式

- 模式定义：将抽象部分(业务功能)与实现部分(平台实现)分离，使它们都可以独立地变化。

- 例如：信息发送业务有移动和pc两个平台，精简版和完整版2个版本

```cpp {.line-numbers}
class Messager{
public:
    virtual void Login(string username, string password)=0;
    virtual void SendMessage(string message)=0;
    virtual void SendPicture(Image image)=0;

    virtual void PlaySound()=0;
    virtual void DrawShape()=0;
    virtual void WriteText()=0;
    virtual void Connect()=0;
    
    virtual ~Messager(){}
};
```

- 若使用继承方法，就会有移动精简发送→移动发送→发送基类 等等这四种类，然而精简和完整版的流程与平台实现无关，因此需要将其剥离

- 与平台实现相关的成员函数为7-10行，因此需将其与实现相关部分变成一个子类,在运行时装配

```cpp
class MessagerImp{
public:
    virtual void PlaySound()=0;
    virtual void DrawShape()=0;
    virtual void WriteText()=0;
    virtual void Connect()=0;
    
    virtual MessagerImp(){}
};

class Messager{
protected:
     MessagerImp* messagerImp;//...
public:
    virtual void Login(string username, string password)=0;
    virtual void SendMessage(string message)=0;
    virtual void SendPicture(Image image)=0;
    
    virtual ~Messager(){}
};

class PCMessagerImp : public MessagerImp{
public:
    
    virtual void PlaySound(){
        //**********
    }
    virtual void DrawShape(){
        //**********
    }
    virtual void WriteText(){
        //**********
    }
    virtual void Connect(){
        //**********
    }
};

```

---

```cpp

class MessagerLite :public Messager {

public:
    
    virtual void Login(string username, string password){
        
        messagerImp->Connect();
        //........
    }
    virtual void SendMessage(string message){
        
        messagerImp->WriteText();
        //........
    }
    virtual void SendPicture(Image image){
        
        messagerImp->DrawShape();
        //........
    }
};

void Process(){
    //运行时装配
    MessagerImp* mImp=new PCMessagerImp();
    Messager *m =new Messager(mImp);
}

```

## brige模式要点总结

- Bridge模式使用“对象间的组合关系”解耦了抽象和实现之间固 有的绑定关系，使得抽象和实现可以沿着各自的维度来变化。所谓抽象和实现沿着各自纬度的变化，即“子类化”它们。

- Bridge模式有时候类似于多继承方案，但是多继承方案往往违背 单一职责原则（即一个类只有一个变化的原因），复用性比较差。Bridge模式是比多继承方案更好的解决方法。

- Bridge模式的应用一般在“两个非常强的变化维度”，有时一个 类也有多于两个的变化维度，这时可以使用Bridge的扩展模式。

# P8. 工厂方法

- 属于对象创建模式，通过这个模式来避免`new`过程中所在造成的紧耦合（依赖于具体类）。它是接口抽象后的第一步工作

- 在软件系统中，经常面临着创建对象的工作；由于需求的变化，需要创建的对象的具体类型经常变化

- 定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使得一个类的实例化延迟（目的：解耦，手段：虚函数）到子类

- 举例：创建多种不同类型的分割器

```cpp
class MainForm : public Form
{
    SplitterFactory*  factory;//工厂

public:
    MainForm(SplitterFactory*  factory){
        this->factory=factory;
    }
    
    void Button1_Click(){ 
        ISplitter * splitter=
            factory->CreateSplitter(); //多态new
        splitter->split();
    }
};
```

```cpp
//抽象类
class ISplitter{
public:
    virtual void split()=0;
    virtual ~ISplitter(){}
};


//工厂基类
class SplitterFactory{
public:
    virtual ISplitter* CreateSplitter()=0;
    virtual ~SplitterFactory(){}
};

class BinarySplitter : public ISplitter{
    
};

class TxtSplitter: public ISplitter{
    
};

class BinarySplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new BinarySplitter();
    }
};

class TxtSplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new TxtSplitter();
    }
};

```

## 工厂方法要点总结

- Factory Method模式用于隔离类对象的使用者和具体类型之间的 耦合关系。面对一个经常变化的具体类型，紧耦合关系(new)会导 致软件的脆弱

- Factory Method模式通过面向对象的手法（多态），将所要创建的具体对象工作**延迟**到子类，从而实现一种扩展（而非更改）的策略，较好地解决了这种紧耦合关系（需要不同对象时只需新建不同类型工厂，并传递该工厂指针即可，对原有运行代码`MainForm`无需更改）

- Factory Method模式解决“单个对象”的需求变化。缺点在于要求创建方法/参数相同。

# P9. 抽象工厂

- 与工厂模式只有细微变化：一系列相互依赖的对象的创建工作

- 模式定义：提供一个接口，让该接口负责创建一系列“相关或者相互依赖的对象”，无需指定它们具体的类。
- 举例：有一系列数据库相关操作，连接、命令、设置连接、读取。同时需要支持不同的数据库

  - 按照之前工厂方法，需要操作、连接、读取的基类、工厂、和具体工厂 （这里会写出很多类）

```cpp
class EmployeeDAO{

IDBConnectionFactory* dbConnectionFactory;
IDBCommandFactory* dbCommandFactory;
IDataReaderFactory* dataReaderFactory;

public:
vector<EmployeeDO> GetEmployees(){
    IDBConnection* connection =
        dbConnectionFactory->CreateDBConnection();
    connection->ConnectionString("...");

    IDBCommand* command =
        dbCommandFactory->CreateDBCommand();
    command->CommandText("...");
    command->SetConnection(connection); //关联性

    IDBDataReader* reader = command->ExecuteReader(); //关联性
    while (reader->Read()){

    }
}
};
```

- 由于读取、命令、连接有很强的关联性，必须要使用同一种类型的数据库，因此这三个操作应该由同一个工厂产生（高内聚）

```cpp
class IDBFactory{
public:
    virtual IDBConnection* CreateDBConnection()=0;
    virtual IDBCommand* CreateDBCommand()=0;
    virtual IDataReader* CreateDataReader()=0;
    
};

class SqlDBFactory:public IDBFactory{
public:
    virtual IDBConnection* CreateDBConnection(){/*...*/};
    virtual IDBCommand* CreateDBCommand(){/*...*/};
    virtual IDataReader* CreateDataReader(){/*...*/};
};

class EmployeeDAO{
    IDBFactory* dbFactory;
    
public:
    vector<EmployeeDO> GetEmployees(){
        IDBConnection* connection =
            dbFactory->CreateDBConnection();
        connection->ConnectionString("...");

        IDBCommand* command =
            dbFactory->CreateDBCommand();
        command->CommandText("...");
        command->SetConnection(connection); //关联性

        IDBDataReader* reader = command->ExecuteReader(); //关联性
        while (reader->Read()){

        }

    }
};
```

## 抽象工厂要点总结

- 抽象工厂更形象的解释是家族工厂，生产一系列相关的类，工厂方法可以看成抽象工厂中的一个特例

- 如果没有应对“多系列对象构建”的需求变化，则没有必要使用Abstract Factory模式，这时候使用简单的工厂完全可以。

- “系列对象”指的是在某一特定系列下的对象之间有相互依赖、或作用的关系。不同系列的对象之间不能相互依赖。

- Abstract Factory模式主要在于应对“新系列”的需求变动。其缺点在于难以应对“新对象”的需求变动。

# P10. 原型模式

- 属于对象创建模式，相较于工厂模式简单一些，是工厂模式的一种变体（实际中用的不是特别多）

- 对象有很复杂的中间状态，从工厂生产出来的步骤过于复杂，使用原型模式直接进行拷贝。这些对象经常面临剧烈的变化，但是却拥有稳定一致的接口

- 模式定义：使用原型实例指定创建对象的种类，然后通过拷贝这些原型来创建新的对象

- 举例：具体实现上就是将抽象类与与工程合并，工厂创建变为克隆

```cpp
class ISplitter{//抽象类
public:
    virtual void split()=0;
    virtual ISplitter* clone()=0; //通过克隆自己来创建对象
    
    virtual ~ISplitter(){}

};

//具体类
class BinarySplitter : public ISplitter{
public:
    virtual ISplitter* clone(){
        return new BinarySplitter(*this);//调用拷贝构造函数克隆当前状态
    }
};

class MainForm : public Form
{
    ISplitter*  prototype;//原型对象

public:
    
    MainForm(ISplitter*  prototype){
        this->prototype=prototype;
    }
    
    void Button1_Click(){

        ISplitter * splitter=
            prototype->clone(); //克隆原型
            //prototype->split;    不能直接使用，原型对象是专门供你克隆的，真正使用的时候，我们要使用新的对象
        
        splitter->split();
    }
};
```

## 原型模式要点总结

- Prototype模式同样用于隔离类对象的使用者和具体类型（易变类）之间的耦合关系。它同样要求这些“易变类”拥有“稳定的接口”

- Prototype模式对于“如何创建易变类的实体对象”采用“原型克隆”的方法来做，它使得我们可以非常灵活地动态创建“拥有某些稳定接口”的新对象——所需工作仅仅是注册一个新类的对象（即原型），然后在任何需要的地方克隆。

- Prototype模式中的clone方法可以利用某些框架中的序列化来实现深拷贝

# P11. 构建器

- 属于对象创建模式，（不常用）

- “一个复杂对象”的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着**剧烈的变化**，但是将它们组合在一起的算法却**相对稳定** (与模板方法很像，但是这里是用来创建对象)

---

- 举例：`House`的需要按照一定流程创建5个部分，这里创建流程和5个部分的创建都很复杂。

  - 首先将`House`类与他的创建过程分离
  - 将5个部分的具体实现（`HouseBuilder` 第5行）与创建流程(`HouseDirector`第46行)分离（若创建流程和步骤不复杂放在一起也行）

```cpp {.line-numbers}

class House{
    //....
};

class HouseBuilder {
public:
    House* GetResult(){
        return pHouse;
    }
    virtual ~HouseBuilder(){}
protected:
    
    House* pHouse;
    virtual void BuildPart1()=0;
    virtual void BuildPart2()=0;
    virtual void BuildPart3()=0;
    virtual void BuildPart4()=0;
    virtual void BuildPart5()=0;
};

class StoneHouse: public House{

};

class StoneHouseBuilder: public HouseBuilder{
protected:
    
    virtual void BuildPart1(){
        //pHouse->Part1 = ...;
    }
    virtual void BuildPart2(){
        
    }
    virtual void BuildPart3(){
        
    }
    virtual void BuildPart4(){
        
    }
    virtual void BuildPart5(){
        
    }
    
};

class HouseDirector{
    
public:
    HouseBuilder* pHouseBuilder;
    
    HouseDirector(HouseBuilder* pHouseBuilder){
        this->pHouseBuilder=pHouseBuilder;
    }
    
    House* Construct(){
        
        pHouseBuilder->BuildPart1();
        
        for (int i = 0; i < 4; i++){
            pHouseBuilder->BuildPart2();
        }
        
        bool flag=pHouseBuilder->BuildPart3();
        
        if(flag){
            pHouseBuilder->BuildPart4();
        }
        
        pHouseBuilder->BuildPart5();
        
        return pHouseBuilder->GetResult();
    }
};

```

## Builder要点总结

- Builder 模式主要用于“分步骤构建一个复杂的对象”。在这其中“分步骤”是一个稳定的算法，而复杂对象的各个部分则经常变化。

- 变化点在哪里，封装哪里—— Builder模式主要在于应对“复杂对象各个部分”的频繁需求变动。其缺点在于难以应对“分步骤构建算法”的需求变动。

- 在Builder模式中，要注意不同语言中构造器内调用虚函数的差别（ C++ vs. C# )

# P12. 单件模式

- 属于性能对象模式

- 面向对象很好地解决了"抽象"问题，但是不可避免的要付出一定代价。（通常继承和虚函数带来的影响可以忽略不计，但是大规模的应用可能会出现性能问题）

- 有些特殊的类，必须保证他们在系统效率中只存在一个实例，（如保证逻辑正确、良好的效率）。因此需要绕过常规的构造器，提供一种机制来保证一个类只有一个实例。这种功能应该由类的设计者来实现，而不是使用者的责任

- 模式定义：保证一个类仅有一个实例，并提供一个该实例的全局访问点

- 双检查锁由于内存读写reorder不安全。编译器出于优化的目的会先返回地址再进行构造，导致读到的内存里还没有完成构造，需要使用原子操作

```cpp
class Singleton{
private:
    Singleton();
    Singleton(const Singleton& other);
public:
    static Singleton* getInstance();
    static Singleton* m_instance;
};

Singleton* Singleton::m_instance=nullptr;

//线程非安全版本 单线程可用
Singleton* Singleton::getInstance() {
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}


//线程安全版本，但锁的代价过高
Singleton* Singleton::getInstance() {
    Lock lock;
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}


//双检查锁，但由于内存读写reorder不安全
Singleton* Singleton::getInstance() {
    
    if(m_instance==nullptr){
        Lock lock;
        if (m_instance == nullptr) {
            m_instance = new Singleton();
        }
    }
    return m_instance;
}

//C++ 11版本之后的跨平台实现 (volatile)
std::atomic<Singleton*> Singleton::m_instance;
std::mutex Singleton::m_mutex;

Singleton* Singleton::getInstance() {
    Singleton* tmp = m_instance.load(std::memory_order_relaxed);
    std::atomic_thread_fence(std::memory_order_acquire);//获取内存fence
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock(m_mutex);
        tmp = m_instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton;
            std::atomic_thread_fence(std::memory_order_release);//释放内存fence
            m_instance.store(tmp, std::memory_order_relaxed);
        }
    }
    return tmp;
}
```

## Singleton要点总结

- Singleton 模式中的实例构造器可以设置为protected以允许子类派生

- Singleton 模式一般不要支持拷贝构造函数和Clone接口，因为这有可能导致多个对象实例，与Singleton初衷违背

- 多线程环境下需要借助C++11 的原子模式实现

# P13. 享元模式

- 纯粹对象方案的问题在于大量细粒度的对象会很快充斥在系统中

- 模式定义：运用共享技术有效的支持大量细粒度的对象

```cpp
class Font {//字体
private:
    string key;//字体唯一标识
public:
    Font(const string& key){
        //...
    }
};
```

- 若为文章中的每个字符创建一个字体，导致大量的重复字体类存在，我们可以定义一个字体的池，当该字体的类对象已经存在于该池内就不用再创建该类对象，这类似于word中的操作

```cpp
class FontFactory{
private:
    map<string,Font* > fontPool;//字体池   
public:
    Font* GetFont(const string& key){
        map<string,Font*>::iterator item=fontPool.find(key);
          if(item!=footPool.end()){
            return fontPool[key];
        }
        else{
            Font* font = new Font(key);
            fontPool[key]= font;
            return font;
        }
    }
    void clear(){
        //...
    }
};
```

## Flyweight要点总结

- 面向对象很好的解决了抽象的问题，但是作为一个运行在 机器中的程序实体，我们需要考虑对象的代价问题。Flyweight主要解决面向对象的代价问题，一般不触及面向对象的抽象性问题。
- Flyewight采用对象共享的做法来降低系统中对象的个数，从而降低细粒度对象给系统带来的内存压力。在具体实现方面，要注意对象状态的处理。（一般这些对象时只读的，否则可能出现共享出错）
- 对象的数量多少才算大？这需要我们仔细地根据具体应用情况进行评估，而不能凭空臆断（sizeof）

# P14. 门面模式

- 属于接口隔离模式

- 在组件构建过程中，某些接口之间直接的依赖常常会带来很多问题、甚至无法实现。采用添加一层间接（稳定）接口，来隔离本来互相紧密关联的接口是一种常见的方案

- 模型定义：为子系统中的一组接口提供一个一致（稳定）的界面，Facade模式定了一个高层接口，这个接口使得这一子系统更加容易使用（复用）。

- Facade模式并没有一种具体的代码结构，在现实中的表现差异可能非常大，它表述的是子系统与外界解耦合的思想

## Facade要点总结

- 从客户程序的角度来看，Facade模式简化了整个组件系统的接口，对于接口内部和外部客户程序来说，达到了一种”解耦“的效果——内部子系统的任何变化不会影响到Facade接口的变化。
- Facade设计模式更注重从**框架**的层次去看整个系统，而不是单个类的层次。Facade很多时候更是一种框架设计模式。
- Facade设计模式并非一个集装箱，可以任意地放进任何多个对象。Facade模型中的组件的内部应该是“相互耦合关系比较大的一系列组件”，而不是一个简单的功能集合。

# P15. 代理模式

- 属于接口隔离模式

- 在面向对象系统中，有些对象由于某种原因(比如创建对象的开销很大，或者某些操作需要安全控制，需要进程外的访问)，直接访问会给使用者、或者系统结构带来很多麻烦。
- 如何在不失去透明操作对象的同时来管理/控制这些对象特有的复杂性？增加一层间接层是软件开发中常见的解决方式

- 模式定义: 为其他对象提供一种代理以控制（隔离，使用接口）对这个对象的访问

## Proxy要点总结

- “增加一层间接层”是软件系统中对许多复杂问题的一种常见解决方法。在面向对象系统中，直接使用某些对象会带来很多问题，作间接层的proxy对象便是解决这一问题的常用手段。

- 具体proxy设计模式的实现方法、实现粒度都相差很大，有些可能对单个对象做细粒度的控制，如copy-on-write技术（string 若是直接复制就是浅拷贝，当需要修改时就拷贝一份修改，这就是一种代理操作），有些可能对组建模块提供抽象代理层，在架构层次对对象做proxy。（很多时候代理模式的代码是由程序生成的）

- Proxy并不一定要求保持接口完整的一致性，只要能够实现间接控制，有时候损及一些透明性是可以接受的。

# P16. 适配器

- 属于接口隔离模式

- 在软件系统中，由于应用环境的变化，常常需要将“一些现存的对象”放在新的环境中应用，但是新环境要求的接口是这些现存对象所不满足的。

- 如何应对这种“迁移的变化”？如何既能利用现有对象的良好实现，同时又能满足新的应用环境所要求的接口？

- 模式定义:将一个类的接口转换成客户希望的另一个接口。Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

```cpp
//目标接口（新接口）
class ITarget{
public:
    virtual void process()=0;
};

//遗留接口（老接口）
class IAdaptee{
public:
    virtual void foo(int data)=0;
    virtual int bar()=0;
};

//遗留类型
class OldClass: public IAdaptee{
    //....
};

//对象适配器
class Adapter: public ITarget{ //继承
protected:
    IAdaptee* pAdaptee;//组合
    
public:
    Adapter(IAdaptee* pAdaptee){
        this->pAdaptee=pAdaptee;
    }
    virtual void process(){
        int data=pAdaptee->bar();
        pAdaptee->foo(data);
    }
};

//类适配器
class Adapter: public ITarget,
               protected OldClass{ //多继承,不推荐这样就绑定到oldclass 失去灵活性
}

int main(){
    IAdaptee* pAdaptee=new OldClass();
    ITarget* pTarget=new Adapter(pAdaptee);
    pTarget->process();

}

```

## Adapter要点总结

- Adapter模式主要应用于“希望复用一些现存的类，但是接口又与复用环境要求不一致的情况”，在遗留代码复用、类库迁移等方面非常有用。（如stack 和 queue 都是 内含了一个deque）

- GoF 23 定义了两种Adapter模式的实现结构：对象适配器和类适配器。但类适配器采用“多继承”的实现方式，一般不推荐使用。对象适配器采用“对象组合”的方式，更符合松耦合精神。

- Adapter模式可以实现的非常灵活，不必拘泥于GoF23中定义的两种结构。例如，完全可以将Adapter模式中的“现存对象”作为新的接口方法参数，来达到适配的目的。

# P17. 中介者模式

- 在软件构建过程中，经常会出现多个对象互相关联交互的情况，对象之间常常会维持一种复杂的引用关系，如果遇到一些需求的更改，这种直接的引用关系将面临不断的变化。

- 在这种情况下，我们可以使用一个“中介对象”来管理对象间的关联关系，避免相互交互的对象之间的紧耦合引用关系，从而更好的抵御变化。

- 模式变化:用一个中介对象来封装（封装变化）一系列的对象交互。中介者使各对象不需要显式的相互引用（编译时依赖→运行时依赖），从而使其耦合松散（管理变化），而且可以独立地改变它们之间的交互。

## Mediator要点总结

- 将多个对象间复杂的关联关系解耦，Mediator模式将多个对象间的控制逻辑进行集中管理，变“多个对象互相关联”为“多个对象和一个中介者关联”，简化了系统的维护，抵御了可能的变化
。
- 随着控制逻辑的复杂化，Mediator具体对象的实现可能相当复杂。这时候可以对Mediator对象进行分界处理。

- Facade模式是解耦系统间（单向）的对象关联关系; Mediator模式是解耦系统内各个对象之间（双向）的关联关系。

# P19. 备忘录

- 属于状态变化模式

- 在软件构建过程中，某些对象的状态在转换过程中，可能由于某种需要，要求程序能够回溯到对象之前处于的某个点时的状态。如果使用一些共有接口来让其他对象得到对象的装填，便会暴露对象的细节实现

- 模式定义: 在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可以将该对象恢复到原先保存的状态。

## Memento 要点总结

- 备忘录（Memento）存储原发器（Originator）对象的内部状态，在需要时恢复原发器状态。
- Memento模式的核心是信息隐藏，即Originator需要向外接隐藏信息，保持其封装性。但同时又需要将状态保持到外界（Memento）。（类似于对对象的深拷贝）
- 由于现代语言运行时（C#、Java等）都具有相当的对象序列化支持，因此往往采用效率较高、又较容易正确实现的序列化方案来实现Memento模式。（对于有层层指针的类型一般使用序列化的方式，当年受制于变成语言的发展，现代编程语言实现备忘录模式已经有多种方法）

# P20. 组合模式

- 属于数据结构模式

- 软件在某些情况下，客户代码过多依赖于对象容器复杂的内部实现结构，对象容器内部实现结构（而非抽象接口）的变化将引起客户代码的频繁变化，带来了代码的维护性、扩展性等弊端

- 如何和将“客户代码与复杂的对象容器结构”解耦？让对象容器自己来实现自身的复杂结构，从而使得客户代码就像处理简单对象一样来处理复杂的对象容器？

- ​ 将对象组合成**树形结构**以表示“部分-整体”的层次结构。Composite使得用户对单个兑现和组合对象的使用具有一致性（稳定）

```cpp

class Component
{
public:
    virtual void process() = 0;
    virtual ~Component(){}
};

//树节点
class Composite : public Component{
    
    string name;
    list<Component*> elements;
public:
    Composite(const string & s) : name(s) {}
    
    void add(Component* element) {
        elements.push_back(element);
    }
    void remove(Component* element){
        elements.remove(element);
    }
    
    void process(){
        //1. process current node
        
        //2. process leaf nodes
        for (auto &e : elements)
            e->process(); //多态调用
    }
};

//叶子节点
class Leaf : public Component{
    string name;
public:
    Leaf(string s) : name(s) {}
            
    void process(){
        //process current node
    }
};


void Invoke(Component & c){
    //...
    c.process();
    //...
}


int main()
{

    Composite root("root");
    Composite treeNode1("treeNode1");
    Composite treeNode2("treeNode2");
    Composite treeNode3("treeNode3");
    Composite treeNode4("treeNode4");
    Leaf leat1("left1");
    Leaf leat2("left2");
    
    root.add(&treeNode1);
    treeNode1.add(&treeNode2);
    treeNode2.add(&leaf1);
    
    root.add(&treeNode3);
    treeNode3.add(&treeNode4);
    treeNode4.add(&leaf2);
    
    process(root);//处理根、树、叶都是同样的接口，使用多态的递归调用
    process(leaf2);
    process(treeNode3);
  
}
```

## Composite要点总结

- Composite模式采用属性结构来实现普遍存在的对象容器，从而将“一对多”的关系转换为“一对一”的关系，使得客户代码可以一直地（复用）处理对象和对象容器，无需关心处理的是单个还是组合的对象容器。

- 将“客户代码与复杂的对象容器结构”解耦是Composite的核心思想，解耦之后，客户代码将与纯粹的抽象接口——而非对象容器的内部实现结构——发生依赖，从而更能“应对变化”。

- Composite模式在具体视线中，可以让父对象中的子对象反向追溯；如果父对象有频繁的遍历需求，可使用缓存技巧来改善效率。

# P21. 迭代器

- 属于数据结构模式

- 在软件构建过程中，集合对象内部结构常常变化各异。但对于这些集合对象，我们希望在不暴露其内部结构的同时，可以让外部客户代码透明访问其中包含的元素；同时这种“透明遍历”也为“同一种算法在多种集合对象上进行操作”提供了可能。

- 使用面向对象技术奖这种遍历机制抽象为“迭代器对象”为“应对变化中的集合对象”提供了一种优雅的方式。

- 模式定义: 提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露（稳定）该对象的内部表示。

```cpp
template<typename T>
class Iterator
{
public:
    virtual void first() = 0;
    virtual void next() = 0;
    virtual bool isDone() const = 0;
    virtual T& current() = 0;
};

template<typename T>
class MyCollection{
public:
    Iterator<T> GetIterator(){
        //...
    }
    
};

template<typename T>
class CollectionIterator : public Iterator<T>{
    MyCollection<T> mc;
public:
    
    CollectionIterator(const MyCollection<T> & c): mc(c){ }
    
    void first() override {
        
    }
    void next() override {
        
    }
    bool isDone() const override{
        
    }
    T& current() override{
        
    }
};

void MyAlgorithm()
{
    MyCollection<int> mc;
    
    Iterator<int> iter= mc.GetIterator();
    
    for (iter.first(); !iter.isDone(); iter.next()){
        cout << iter.current() << endl;
    }
    
}
```

## Iterator 模式总结

- 迭代抽象：访问一个聚合对象的内容而无需暴露它的内部表示。

- 迭代多态：为遍历不同的集合结构提供一个统一的接口，从而支持同样的算法在不同的集合结构上进行操作。

- 迭代器的健壮性考虑：遍历的同时更改迭代器所在的集合结构，会导致问题。（许多迭代器要求是只读的）

- 上述为面向对象的迭代器，现在推荐使用泛型编程的迭代器（编译时确定，而不是运行时的调用），没有虚函数的调用的开销

# P22. 职责链

- 属于数据结构模式

- 在软件构建过程中，一个请求可能被多个对象处理，但是每个请求运行时只能有一个接受者，如果显式指定，将必不可少地带来请求发送者与接受者的紧耦合。

- 如何使请求的发送者不需要指定具体的接受者？让请求的接受者自己在运行时决定来处理请求，从而使二者解耦。

- 模式定义：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止

## chain of Responsibility 要点总结

- COR模式的应用场合在于“一个请求可能有多个接受者，但是最后真正的接受者只有一个”，这时候请求发送者与接受者的耦合有可能出现“变化脆弱”的症状，职责链的目的就是将二者解耦，从而更好地应对变化。

- 应用了COR模式后，对象的职责分派将更具灵活性。我们可以在运行时动态添加/修改请求的处理职责。

- 如果请求传递到职责链的末尾仍得不到处理，应该有一个合理的缺省机制。这也是每个接受对象的责任，而不是发出请求的对象的责任。

- 随着数据结构的发展，现在这个模式更像是一种数据结构（不怎么流行）