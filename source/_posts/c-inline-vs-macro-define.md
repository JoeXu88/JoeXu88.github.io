---
title: c++ inline vs macro define
date: 2019-02-15 09:48:00
tags: [c++, program]
categories: program
---

c++ inline 函数和宏定义目的都是为了替换，在函数调用的场景下省去函数调用的开销。不过他们的区别还是挺大的。
<!--more -->

#### inline 函数  
inline 只修饰函数，一般都是加在函数定义的前面。比如：
```c
inline int max(int a, int b)
{
    return a>b?a:b;
}
```
inline 函数调用的地方其实是函数实现的复制，省去了函数调用的开销。这么好的作用为什么不大部分函数都声明为inline呢，因为inline函数带来的副作用是程序体积会增大。如果基本都声明为inline函数，那么应用程序的体积会大的无法接受。所以inline函数一般用于使用频率高，而且功能简短的情景，这样不至于体积肆意膨胀。


#### 宏定义  
宏定义其实就没有修饰的限制，因为宏定义其实就是简单的字符串替换。这样会带来的一些不便的后果：
```C
这里定义一个max的情景=>
#define MAX(a,b) a>b?a:b  
```

* 没有任何的变量类型检查，在实际替换语境中药时刻注意变量的类型隐藏变化  
```c
int a=3; double b=3.2;
int c=MAX(a,b);
这里就需要注意下变量之间的定义，虽然结果可能是正确的
```
* 表达式复制后在实际语境中带来的歧义，因为是简单的复制替换，因此在实际中可能会导致意外的结果，如：
```c
int a=1, b=2;
int c=3;
int d=MAX(a+b,c);
替换后的表达式：a+b>c?a+b:c，这个结果就乱了
```