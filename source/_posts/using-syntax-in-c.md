---
title: using syntax in c++
date: 2019-02-19 17:31:55
tags: c++
categories: program
---


看到using ， 联想到最多的在c++ 中的应用就是声明域名，比如using namespace std。事实上using 还有其他多种用法，这里总结下。

<!--more-->

#### 声明域名  
这个最常用，也最简单。用法比如：using namespace std，或者某个类名。


#### 解锁某些不可见  
解锁未必描述的最准确，但是大概能说明其中的情况。
且看以下几种场景：
1.子类以public 方式继承基类，但是基类有些成员函数被隐藏了
```c
class base
{
    public:
    base() {};
    virtual ~base(){}

    void pfunc() {printf("base void func\n");}
    void pfunc(int i) {printf("base int func\n");}

    private:
    int val;
};

class derive: public base
{
    public:
    derive() {}
    virtual ~derive(){}

    using base::pfunc;
    void pfunc(double i) {printf("derive double func\n");}   //base 中的pfunc都会被隐藏，这里用using 声明后就可以解锁隐藏，从而使得derive对象可以访问base的pfunc实现
};

int main()
{
    derive *d = new derive();
    d->pfunc(3.2);
    d->pfunc(); //如果没有using， 则编译会出错，base函数不能访问

    return 0;
}
```

2.子类以private 的方式继承基类，那么子类对象将无法访问基类的public 成员函数
```c
class base
{
    public:
    base() {};
    virtual ~base(){}

    void pfunc() {printf("base void func\n");}
    void pfunc(int i) {printf("base int func\n");}

    private:
    int val;
};

class derive: private base
{
    public:
    derive() {}
    virtual ~derive(){}

    using base::pfunc;
    void pfunc(double i) {printf("derive double func\n");}   //base 中的pfunc为私有不可见，这里用using 声明后就可以解锁保护，从而使得derive对象可以访问base的pfunc实现
};
```

#### 别名的替代  
在c++ 11 中引入了新的用法，就是替代别名，并且这种做法被鼓励用来替代过去的 typedef（具体可以参见Morden C++ Design）。这里大概看下他们的区别：  
1.类型的别名  
```c
typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;
using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;
```
typedef 的写法被Morden C++ Design作者Meyers描述为太过C++98，而using 则显得更加友好。或许这个例子并不能让我们深切感受到其区别，那么继续。

2.函数别名  
Meyers 举了个例子：
```c
typedef void (*FP) (int, const std::string&);
```
这个定义对于不是很熟悉函数指针和typedef的同学来说就会显得不那么友好，而using 则显得更加显示一些：
```c
using FP = void (*) (int, const std::string&);
```
例子的补充：
```c
typedef std::string (Foo::* fooMemFnPtr) (const std::string&);

using fooMemFnPtr = std::string (Foo::*) (const std::string&);
```
从这些例子可以看出using的用法其实从代码的可读性上来说，是要优于typedef 的，但是仅仅从上述的例子并不足以说服我们转向using，而弃用typedef。  

3.模板的别名  
这个功能才是using 的杀手锏，而且typedef 做不到。
```c
template <typename T>
using Vec = MyVector<T, MyAlloc<T>>;

// usage
Vec<int> vec;
```
这个看起来很自然，但是typedef 却是做不到的：
```c
template <typename T>
typedef MyVector<T, MyAlloc<T>> Vec;

// usage
Vec<int> vec;
```
这个用法当编译的时候会遭遇 "error: a typedef cannot be a template的错误信息。"的错误。  
> 那么，为什么typedef不可以呢？在 n1449 中提到过这样的话："we specifically avoid the term “typedef template” and introduce the new syntax involving the pair “using” and “=” to help avoid confusion: we are not defining any types here, we are introducing a synonym (i.e. alias) for an abstraction of a type-id (i.e. type expression) involving template parameters."  

那么如果要用typedef 来做，改如何做呢，Meyers给出了一些示例：
```c
template <typename T>
struct Vec
{
  typedef MyVector<T, MyAlloc<T>> type;
};

// usage
Vec<int>::type vec;


template <typename T>
class Widget
{
  typename Vec<T>::type vec;
};
```
这种实现看起来很不友好，更糟糕的是在传递别名类型的时候还要用typename 来声明为类型而不是其他的情况。但是用using 就不会有这种情况发生：  
```c
// if we use using syntax.
template <typename T>
class Widget
{
  Vec<T> vec;
};
```
因此在c++11中建议直接用using来代替typedef。同时给出Meyers给出的Things to Remember:  
* typedefs don’t support templatization, but alias declarations do. 
* Alias templates avoid the “::type” suffix and, in templates, the “typename” prefix often required to refer to typedefs.
* C++14 offers alias templates for all the C++11 type traits transformations.

[参考知乎上的一篇专栏](https://zhuanlan.zhihu.com/p/21264013)

