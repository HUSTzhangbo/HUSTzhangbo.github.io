---
title: C++中的字符串流使用总结
date: 2019-02-18 10:48:37
category: C++
tags:
- 类型转换
- stringstream
---

在接触stringstream之前，每当需要进行类型转换时，想到的便是<stdio.h>中的sscanf()函数和sprintf()函数。那么，既然已经有了sscanf()和sprintf()，为什么还要学习和使用stringstream来实现类型转换呢？***因为用stringstream来实现类型转换更加自动和安全。***

***自动：***由于变量的类型在编译期就确定了，所以stringstream可以根据变量类型自动选择要执行的转换，不需要人为给定格式化字符串。

***安全：***stringstream对象有按需分配的内部缓冲区，不会出现缓冲区溢出的问题。

<sstream\>库定义了三种类：istringstream、ostringstream和stringstream，分别用来进行流的输入、输出和输入输出操作。在进行类型转换时，我们常用的是stringstream类及其clear()和str()方法。

```c++
void clear (iostate state = goodbit);
```

> Set error state flags
>
> Sets a new value for the stream's internal *error state flags*. The value existing before the call has no effect.
>
> The current value of the flags is overwritten: All bits are replaced by those in state; If state is goodbit (which is zero) all error flags are cleared.
>
> The current state can be obtained with member function rdstate.

```c++
string str() const;
void str (const string& s);
```

> Get/set content
>
> The first form *(1)* returns a string object with a copy of the current contents of the stream.
>
> The second form *(2)* sets s as the contents of the stream, discarding any previous contents. **Notice that setting a new string does not clear the error state flags currently set in the stream object unless the member function clear is explicitly called.**

**通过以上描述可知，调用clear()方法仅清空流对象内部的错误标志位，不影响流对象中的内容；调用str("")仅清空流对象中的内容，不影响流对象的错误标志位。**

```c++
#include <iostream>
#include <sstream>
#include <string>
using namespace std;

int main(){
    stringstream ss;
    string s = "100";
    int a;
    
    ss << s;
    ss >> a;
    ss.clear();
    cout << "size of stream: " << ss.str().length() << ", result: " << a << endl;
    
    a = 66;
    ss << a;
    ss >> s;
    ss.clear();
    cout << "size of stream: " << ss.str().length() << ", result: " << s << endl;
    
    return 0;
}
```

上述代码的运行结果为：

```
size of stream: 3, result: 100
size of stream: 5, result: 66

--------------------------------
Process exited with return value 0
Press any key to continue . . .
```

若在每个clear()方法后调用对象的str("")方法清空内部的缓冲区，则运行结果如下：

```
size of stream: 0, result: 100
size of stream: 0, result: 66

--------------------------------
Process exited with return value 0
Press any key to continue . . .
```

**因此，如果程序中重复使用同一个stringstream对象，最好在每次使用结束后调用该对象的clear()方法和str("")方法。**

此外，还可以利用stringstream定义一个模板函数，用于任意类型之间的转换。

```c++
#include <iostream>
#include <sstream>
using namespace std;

template<typename out_type, typename in_type>
out_type convert(const in_type &src){
	stringstream ss;
	out_type result;
	ss << src;
	ss >> result;
	return result;
}

int main(){
    double d;
    string s = "10086.5";
    d = convert<double>(s);
    cout << d << endl;
    return 0;
}
```

以上代码的输出结果为10086.5，结果符合预期。