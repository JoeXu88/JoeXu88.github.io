---
title: 记录一道c++面试题
date: 2019-01-27 10:28:14
tags: [c++, 面试]
categories: 面试
description:
---

# 目的
考察虚函数表以及基本的类函数的调用。

<!--more -->

## 题目
有2个类的定义如下:
```c
class A
{
public:
    void f1() { printf("A f1\n"); }
    static void f2() { printf("A f2\n"); }
    virtual void f3() { printf("A f3\n"); }
    virtual void f4() { printf("A f4\n"); }
};

class B
{
public:
    virtual void f1() { printf("B f1\n"); }
    static void f2() { printf("B f2\n"); }
    void f3() { printf("B f3\n"); }
    virtual void f4() { printf("B f4\n"); }
};
```

那么如下调用会输出什么样的结果：
```c
A obj;
B *b = (B*)&obj;
b->f1();
b->f2();
b->f3();
b->f4();
```

首先这个当然是可以编译通过的，调用完全正常，并不非法。这里涉及的是基本的函数调用和虚函数表的理解。

结果如下：
```
A f3
B f2
B f3
A f4
```

对结果的理解：
1. 首先B 对象的指针指向了A 类型，那么虚函数表就是A 类的虚函数表。
2. 对于非虚函数的调用，如f2, f3都是正常调用的B 类的函数，因为不基于虚函数表的函数调用都将直接调用该类的函数，不管是不是static的。
3. 关于虚函数表的函数，其实就是从对应的类的虚函数表中找对应的函数的地址。而这个对应关系其实并不是按照函数名字，而是按照函数地址的偏移。

对于虚函数表A的函数地址：
addr1: f3
addr2: f4

对于虚函数表B的函数地址：
addr1: f1
addr2: f4