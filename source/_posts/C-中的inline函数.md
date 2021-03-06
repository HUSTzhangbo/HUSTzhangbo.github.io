---
title: C++中的inline函数
date: 2019-03-04 14:53:46
category: C++
tags: 
- 内联函数
---

inline关键字是用来定义内联函数的，引入内联函数主要是为了替代C语言中的宏函数。

## 内联函数的声明与定义

关键字inline必须与函数定义放在一起才能使函数成为内联，仅将inline放在函数声明前面不起任何作用。因此，可以认为，inline是“用于实现的关键字”，而不是“用于声明的关键字”。 此外，在类声明中进行定义的成员函数将自动地成为内联函数。

## 编译器对内联函数的处理

由于内联函数需要进行类型检查，因此，编译器是在**编译阶段**对内联函数进行处理的。编译器通过复制函数体来消除函数调用的开销。但是，inline关键字对编译器只是一种**建议**，编译器可以根据自己的实现策略选择是否内联。

## 内联函数的优点

1、内联函数类似于宏函数，会在被调用的地方进行代码展开，省去了函数调用的开销，从而提高程序运行速度。

2、内联函数相比于宏函数，在进行代码展开时，会进行类型检查等操作，更加安全。

3、内联函数可以访问类的成员变量，而宏函数则不行。

## 内联函数的缺点

1、**代码膨胀：**内联是以代码复制为代价来消除函数调用开销的，使用内联函数，会使程序的代码增多。

2、**更新麻烦：**函数库中的内联函数，会在调用它的各个文件中先展开，然后编译。一旦该内联函数发生变化，所有用到该函数的文件都必须重新编译。而如果是非内联函数，只需与其他文件重新链接即可。

## 注意事项

1、使用函数指针调用内联函数将会导致内联失败：使用指针调用函数，就必须获取函数地址。而要取得一个内联函数的地址，编译器就必须为该函数生成一个函数实体。

2、如果函数体的代码过长，或者函数的调用开销相比于函数体的执行开销很小时，则不宜内联。

3、**类的构造函数、析构函数和虚函数不宜内联：**类的构造函数可能需要调用父类的构造函数，析构函数同理。二者背后可能隐藏着许多代码，因此不适合作为内联函数。虚函数是运行时才确定的，而内联操作需要在编译时执行，所以内联与虚函数相矛盾。

4、**内联函数应该定义在头文件中：**由于编译器必须在内联函数被调用的时候将其展开，而不同的源文件只有在链接的时候才会发生关联。因此，必须将其定义在头文件中。

5、如果内联失败，通常编译器会让该函数失去外部链接性，所以一般不会出现重定义的错误。