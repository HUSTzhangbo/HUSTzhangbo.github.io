---
title: C++中的虚函数
date: 2019-02-26 15:07:24
category: C++
tags: 
- 动态绑定
- 虚函数
---

**virtual**是C++中用来定义**虚函数**的关键字。由于C++中成员函数的调用默认不执行动态绑定，若要触发动态绑定，必须满足两个条件：1、将该成员函数指定为虚函数；2、通过基类类型的引用或指针进行调用。

为什么必须是引用或指针呢？为了能够“在单一对象中展现多种类型”，多态需要一层间接性。倘若我们直接声明一个对象，则系统会分配一块用来容纳该实际对象的内存空间。如果后续传入的是派生类对象，便没有足够的内存放置派生类中的各个数据成员。

## 在派生类中定义虚函数

当我们要在派生类中对基类所提供的虚函数进行覆盖时，那么派生类所提供的函数定义的原型必须完全符合基类所声明的函数原型，包括：**参数列表**、**返回类型**、**常量性**。但这条规则存在一个***例外***：当基类的虚函数返回某个基类形式（通常是引用或指针）时，派生类中的同名函数可以返回该基类所派生出来的类型。

## 虚函数的静态解析

如果在**基类**的构造函数或析构函数中调用某个虚函数，则虚函数机制不会出现预期行为。这是因为，当我们构造派生类对象时，基类的构造函数会先被调用，但派生类中的数据成员还未初始化，如果此时基类的构造函数中调用了某个虚函数，便有可能访问未经初始化的数据成员。同理，若在基类的析构函数中调用某个虚函数，便可能访问已经析构掉的数据成员。基于以上原因，在基类的构造函数和析构函数中，派生类的虚函数绝对不会被调用。

## 以下函数不能用virtual修饰

1、内联函数：内联函数在编译时期展开，而虚函数是在运行时才动态绑定的。两者相互矛盾，所以不能将内联函数定义为虚函数。

2、构造函数：在调用构造函数时，虚函数表Vtable还没有建立，因此无法调用虚函数。且构造函数不需要也不应该是虚函数，因为我们在创建一个对象时总是要指明对象所属的类。**但基类的析构函数通常都是虚函数！**

3、静态成员函数：静态成员函数属于整个类而非某个对象，没有this指针，无法访问指向虚函数表的指针vptr（虚函数的调用关系是：this->vptr->Vtable->virtual function）。

4、友元函数：C++不支持友元函数的继承，对于没有继承特性的函数也就不存在虚函数的说法了。