---
title: emplace option for STL containers in c++11
date: 2019-02-19 14:10:27
tags: [c++]
categories: program
---

在c++ 11 之前，容器的插入类对象操作一般都是insert 或者 push_back 之类的，但是都不可避免的会有对象内存的拷贝。究其原因是这些操作都需要预先创建一个类，再调用类的拷贝构造函数来复制拷贝相应的内存。为了提高效率，c++ 11 中引入了emplace 操作，可以直接在容器中分配的内存地址上创建对象，而不需要预先创建类，这就省去了内存拷贝的工作，也省去了可能的临时类的创建和销毁。同时，emplace操作依然支持插入已经创建的对象，也就是调用了拷贝构造函数或者move 构造函数（move语意操作也是c++ 新引入的，主要针对右值引用"&&"，其实就是有效利用了将亡变量的内存，省去内存的重新再分配，这也是提高效率的一个操作）。

<!--more-->

##### 示例  
```c
class A
{
public:
    A(const char* s):name_(s) { printf("A %s constructor\n", name_.c_str()); }
    ~A() { printf("A %s deconstructor\n", name_.c_str()); }
    A(const A& a):name_(a.name_) { printf("A %s copy constructor\n", name_.c_str()); }
    A(A&& a):name_(move(a.name_)) {printf("A %s move constructor\n", name_.c_str()); }

private:
    string name_;
};
```
```c
测试1：
A a("local var"); //local var
vector<A> va;
va.reserve(100);
va.push_back(a);
va.push_back(A("tmp var")); //rval, will be moved

输出：
A local var constructor    //local a ctor
A local var copy constructor  //insert to va, call copy ctor
A tmp var constructor        //tmp var A
A tmp var move constructor   //insert to va, call move ctor
```

```c
测试2：
A a("local var"); //local var
vector<A> va;
va.reserve(100);
va.emplace_back("emplace var");
va.emplace_back(a);
va.emplace_back(A("tmp var"));


输出：
A local var constructor  //local var ctor
A emplace var constructor  //emplace to insert var, only call ctor, do not create any other objects
A local var copy constructor  //emplace local var, will call copy ctor to create object and insert to va
A tmp var constructor         //emplace a tmp rval, create a tmp var first
A tmp var move constructor    //call move ctor and create object to insert to va
```

从上述的示例中可以看到，emplace 的操作完全可以代替原先的push_back 操作，而且emplace还支持直接创建对象，而不需要再利用额外的对象来辅助创建容器内存中的对象。因此可以大大提高效率，尤其在类比较大的时候。
总结一句：emplace 操作是在不同的情况下调用不同的构造函数，这一点和push_back 操作一致，唯一区分效率的是支持直接调用构造函数，而不需要事先分配对象，然后拷贝之前的对象的内存。  