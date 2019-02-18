---
title: C++中函数返回value与reference的区别
date: 2019-02-18 10:46:48
category: C++
tags:
- 函数返回值
- 值与引用
---

通常情况下，函数的返回值可以分为两种情况，一种是返回***value***，另一种是返回***reference***。只有函数的返回值为引用类型时，该函数的返回值才属于***左值***。

```c++
class A{ ... };
//以下函数返回类型A，返回值为右值
A myVal(){
    A a;
    return a;
}
//以下函数返回类型A的引用，返回值为左值
A& myRef(A &a){
    ...
    return a;
}
```

除了左值与右值的差别以外，返回value与返回reference在某些情况下还会带来程序效率的差异。为了方便说明，下面用函数X表示调用者，函数Y表示***返回value***的函数，函数Z表示***返回reference***的函数。

在X中调用Y：首先在X的栈中为***临时对象***分配内存区域，等到Y执行结束时，用Y的返回值初始化***临时对象***（调用拷贝构造函数）。

在X中调用Z：Z执行结束时并不存在复制操作，而是直接返回对象本身，~~此处姑且简单理解为返回了对象的指针~~。***因此，一定不要返回局部变量的引用，否则会导致无法预计的后果。***

下面用具体例子进行说明。由于C++编译器通常会使用copy elision技术来避免不必要的拷贝操作，因此，为了不受编译器优化的影响，在gcc中加入-fno-elide-constructors命令。

```c++
#include <iostream>
#include <string>
using namespace std;

class A{
public:
	//此处为了实验方便直接将成员变量声明为public
	int v;
	A(){
		cout << " Empty-constructor: " << this << endl;
	}
	//拷贝构造函数的参数必须为引用类型，否则会在传参的过程中造成无限递归
	A(const A &a){
		cout << " Copy-constructor: " << this << endl;
		this->v = a.v;
	}
	~A(){
		cout << " ~A: " << this << endl;
	}
	A operator=(const A &a){
		cout << " Assignment: " << this << endl;
		this->v = a.v;
		return *this;
	}
};

A func(){
	A a;
	return a;
}

int main(){
	cout << " 1" << endl;
	func();
	cout << " 2" << endl;
	A b = func();
	cout << " 3" << endl;
	A c;
	c = b;
	cout << " 4" << endl;
	return 0;
}
```

***注：***C++的构造函数是个回调函数，没有返回值。对象的构造过程是先分配内存，当内存分配成功后再调用构造函数。

上述代码的执行结果以及相应的分析如下：

```
 1
 //构造func函数中的对象a（fecc）
 Empty-constructor: 0x28fecc
 //调用拷贝构造函数初始化临时对象（ff04）
 Copy-constructor: 0x28ff04
 //func函数执行结束之时，析构func函数栈中的对象a（fecc）
 ~A: 0x28fecc
 //语句结束，析构因调用func函数所生成的临时对象（ff04）
 ~A: 0x28ff04
 2
 //构造func函数中的对象a（fecc）
 Empty-constructor: 0x28fecc
 //调用拷贝构造函数初始化临时对象（ff08）
 Copy-constructor: 0x28ff08
 //析构func函数栈中的对象a（fecc）
 ~A: 0x28fecc
 //调用拷贝构造函数用临时对象初始化对象b（ff00）
 Copy-constructor: 0x28ff00
 //语句结束，析构临时对象（ff08）
 ~A: 0x28ff08
 3
 //调用构造函数构造对象c（fefc）
 Empty-constructor: 0x28fefc
 //对象c（fefc）调用重载后的赋值运算符
 Assignment: 0x28fefc
 //重载的赋值运算符执行结束之时，调用拷贝构造函数初始化临时对象（ff0c）
 Copy-constructor: 0x28ff0c
 //语句结束，析构临时对象（ff0c）
 ~A: 0x28ff0c
 4
 //析构对象c（fefc）
 ~A: 0x28fefc
 //析构对象b（ff00）
 ~A: 0x28ff00

--------------------------------
Process exited with return value 0
Press any key to continue . . .
```

若将代码中赋值运算符的重载函数进行修改，让其返回类型A的***引用***，如下：

```c++
	A& operator=(const A &a){
		cout << " Assignment: " << this << endl;
		this->v = a.v;
		return *this;
	}
```

则程序的运行结果与相应分析为：

```
 1
 Empty-constructor: 0x28fedc
 Copy-constructor: 0x28ff08
 ~A: 0x28fedc
 ~A: 0x28ff08
 2
 Empty-constructor: 0x28fedc
 Copy-constructor: 0x28ff0c
 ~A: 0x28fedc
 Copy-constructor: 0x28ff04
 ~A: 0x28ff0c
 3
 //调用构造函数构造对象c（ff00）
 Empty-constructor: 0x28ff00
 //对象c（ff00）调用重载后的赋值运算符，由于函数返回引用类型，故不存在临时对象
 Assignment: 0x28ff00
 4
 ~A: 0x28ff00
 ~A: 0x28ff04

--------------------------------
Process exited with return value 0
Press any key to continue . . .
```

让重载的赋值运算符重新返回类型A，并将main函数修改如下：

```c++
int main(){
	A a, b, c;
	a.v = 1;
	b.v = 2;
	c.v = 3;
	(a = b) = c;
	cout << " " << a.v << " " << b.v << " " << c.v << endl;
	return 0;
}
```

程序的运行结果为：

```
 Empty-constructor: 0x28fef4
 Empty-constructor: 0x28fef0
 Empty-constructor: 0x28feec
 Assignment: 0x28fef4
 Copy-constructor: 0x28fefc
 Assignment: 0x28fefc
 Copy-constructor: 0x28fef8
 ~A: 0x28fef8
 ~A: 0x28fefc
 2 2 3
 ~A: 0x28feec
 ~A: 0x28fef0
 ~A: 0x28fef4

-------------------------------
Process exited with return value 0
Press any key to continue . . .
```

程序运行的预期结果应该为 3  2  3 ，但实际输出却是 2  2  3，分析发现，问题出在了重载的赋值运算符的返回值上。由于函数返回的是类型A而非类型A的引用，导致(a=b)=c中第二个赋值运算符的实际调用者是临时对象（0x28fefc），该临时对象由对象a（0x28fef4）调用赋值运算符所产生。

让重载的赋值运算符返回类型A的***引用***，重新执行程序，结果为：

```
 Empty-constructor: 0x28fefc
 Empty-constructor: 0x28fef8
 Empty-constructor: 0x28fef4
 Assignment: 0x28fefc
 Assignment: 0x28fefc
 3 2 3
 ~A: 0x28fef4
 ~A: 0x28fef8
 ~A: 0x28fefc

--------------------------------
Process exited with return value 0
Press any key to continue . . .
```

观察程序输出可知，两个赋值运算符的调用者都是对象a（0x28fefc），且没有多余的拷贝操作，程序输出符合预期。