---
title: c++ 类对象内存布布局
date: 2019-01-25 17:13:45
tags: c++ program
---

# 前言
为了更清晰的说明类的内存布局，首先说明下一个执行程序的内存格局，通常其包含：全局数据区，代码区，栈区，堆区。全局数据区存放全局变量，静态数据和常量；代码段存放函数实现；栈区存放为函数运行而分配的局部变量、函数参数、返回数据、返回地址等；剩余的内存就是堆，可以用来分配动态内存。

<!-- more -->

# 正文
接下来是正文内容。首先c++类成员包含了函数和变量，那么当分配一个对应的对象的时候，这个对象需要包含函数，以及变量等数据（编译系统额外产生的内存数据）。所以一个对象内存可以分为2部分，一部分是函数实现地址，另外一部分就是数据。

## 函数部分
首先是函数部分，对于同一个类分配的不同对象而言，其函数实现是一样的，所以如果每个不同对象都存放一份同样的代码，那么显然浪费内存。因此编译系统会将函数实现统一放到代码区，那么所有对象都会调用一个函数实现的地址，这样就可以大大节约内存。因此函数的成员不会增加对象的内存。
但是函数又分静态成员函数和非静态成员函数，他们的存储是否有区别呢？ 事实上从存储的角度是没有区别的，都是存放在代码区。只不过对于c++而言静态函数和非静态函数实现接口上是有区别的。我们试想不同对象调用同一个接口（函数会使用到成员变量的值）怎么得出不同的结果呢？ 事实就是非静态成员函数默认会有一个指向类对象指针的参数（即this指针），这样就可以从this指针中拿到具体对象的成员变量值。然而静态成员函数是不需要这个参数的，因为静态函数是共享给所有同一个类的对象的，因此也可以解释为什么静态函数内只能用静态成员变量，不然没有this指针无法得到具体对象的成员变量值。

那么最后c++的精华部分虚函数怎么存放和实现？首先虚函数也是普通的成员函数加上一个virtual 修饰而已，所以从存放的角度，依然是存放在代码段。只是为了实现多态，目前大部分的编译系统都会通过添加虚函数表的方法来完成，只是不同编译系统可能有些许不一样，但是大体上都是差不多的实现方法。因此这个虚函数表是编译系统徒增出来的，这里把它放到数据部分。

## 数据部分
数据部分这里分2大块，一块是成员变量，另外一块是虚函数表。

成员变量其实相对简单，并且它是跟随不同对象的，因为不同具体的对象需要存储不同的变量值。但是有一个情况例外，就是静态型成员变量。因为静态的成员变量是存储在静态数据区的，也就是全局数据区，它并不属于具体的对象，而是所有对象共享的。也因此，静态的成员变量并不会增加对象的内存，也就是不会体现在对象的内存中，这一点和函数实现一样。

另一块虚函数表其实就是函数地址的索引表，里面存放了基类或者子类的实现函数地址。这里引申出2个问题：虚函数表本身在内存中存放在哪里，是否是跟随不同对象改变； 虚函数表如何存放函数地址，如何实现多态。
首先第一个问题，虚函数表存放在哪里。其实当类，以及类继承定义完成，针对不同的对象的虚函数表就可以确定了。那么虚函数表是跟随类的，即不同类继承以及实现对应不同的虚函数表。但是虚函数表跟随具体对象吗？也就是没生成一个具体对象的时候，虚函数表是否都会生成一份？事实上是不会的，因为虚函数表在类定义阶段就可以被确定，所以所有对象都可以共享一份。所以虚函数表其实是放在全局数据区的。[参考：https://blog.csdn.net/houdy/article/details/1496161]
接着第二个问题，虚函数表如何实现多态。这个问题相对会有些复杂，因为需要根据不同的继承方式来定义这个虚函数表。对于继承方式，层层递进可以分为：单继承，多继承，重复继承，虚继承。
最简单的单继承的情况，定义2个类：
```C++
class parent
{
public:
	virtual void f() { printf("parent f\n"); }
	virtual void g() { printf("parent g\n"); }

private:
	int a;
};

class parent1: public parent
{
public:
	virtual void f() { printf("parent1 f\n"); }
	virtual void g1() { printf("parent1 g1\n"); }

private:
	int a;
};
```

*我们看看visual studio给出的虚函数表情况：*
```
class parent1	size(12):
	+---
 0	| +--- (base class parent)
 0	| | {vfptr}
 4	| | a
	| +---
 8	| a
	+---

parent1::$vftable@:
	| &parent1_meta
	|  0
 0	| &parent1::f
 1	| &parent::g
 2	| &parent1::g1
```

上述结果中，首先对象的内存顺序依次是虚函数表->基类的成员变量->子类的成员变量，因此可以看到的是首先会存储基类的相关数据，然后是子类的相关数据，而且虚函数表会在最开始的位置。再看虚函数表中的实现函数地址，子类的实现f() 是覆盖了基类的实现f() 的，因此虚函数表中会直接存放子类的实现f() 的地址，这就是多态的实现原理。当基类指针指向了子类对象的时候，虚函数表会提供具体对象的实现函数地址。

进一步，多继承，类定义：
```C++
class parent
{
public:
	virtual void f() { printf("parent f\n"); }
	virtual void g() { printf("parent g\n"); }

private:
	int a;
};

class parent1
{
public:
	virtual void f() { printf("parent1 f\n"); }
	virtual void g1() { printf("parent1 g1\n"); }

private:
	int a;
};

class parent2
{
public:
	virtual void f() { printf("parent2 f\n"); }
	virtual void g2() { printf("parent2 g2\n"); }

private:
	int a;
};

class son : public parent1, public parent2
{
public:
	virtual void f() { printf("son f\n"); }
	virtual void gs() {  printf("son g\n"); }

private:
	int a;
};
```

*我们看看visual studio给出的虚函数表情况：*
```
class son	size(20):
	+---
 0	| +--- (base class parent1)
 0	| | {vfptr}
 4	| | a
	| +---
 8	| +--- (base class parent2)
 8	| | {vfptr}
12	| | a
	| +---
16	| a
	+---

son::$vftable@parent1@:
	| &son_meta
	|  0
 0	| &son::f
 1	| &parent1::g1
 2	| &son::gs

son::$vftable@parent2@:
	| -8
 0	| &thunk: this-=8; goto son::f
 1	| &parent2::g2
```

依旧看看内存顺序：虚函数表1->基类1成员变量->虚函数表2->基类2成员变量->子类的成员变量，依旧是先存储基类的数据，这里多个基类的顺序是程序中声名的顺序而已，并没有其余的规则（至少vs和gcc编译器是这样的）。值得注意的是2个虚函数表中f() 的实现都是子类的实现，因为子类实现覆盖了基类的实现。

再进一步，重复继承，类定义：
```C++
class parent
{
public:
	virtual void f() { printf("parent f\n"); }
	virtual void g() { printf("parent g\n"); }

private:
	int a;
};

class parent1: public parent
{
public:
	virtual void f() { printf("parent1 f\n"); }
	virtual void g1() { printf("parent1 g1\n"); }

private:
	int a;
};

class parent2: public parent
{
public:
	virtual void f() { printf("parent2 f\n"); }
	virtual void g2() { printf("parent2 g2\n"); }

private:
	int a;
};

class son :  public parent1, public parent2
{
public:
	virtual void f() { printf("son f\n"); }
	virtual void gs() { printf("son g\n"); }

private:
	int a;
};
```

*我们看看visual studio给出的虚函数表情况：*
```
class son	size(28):
	+---
 0	| +--- (base class parent1)
 0	| | +--- (base class parent)
 0	| | | {vfptr}
 4	| | | a
	| | +---
 8	| | a
	| +---
12	| +--- (base class parent2)
12	| | +--- (base class parent)
12	| | | {vfptr}
16	| | | a
	| | +---
20	| | a
	| +---
24	| a
	+---

son::$vftable@parent1@:
	| &son_meta
	|  0
 0	| &son::f
 1	| &parent::g
 2	| &parent1::g1
 3	| &son::gs

son::$vftable@parent2@:
	| -12
 0	| &thunk: this-=12; goto son::f
 1	| &parent::g
 2	| &parent2::g2
```

依旧看看内存顺序：虚函数表->基类的成员变量->基类1的成员变量->虚函数表->基类的成员变量->基类2的成员变量->子类的成员变量。从内存存储中可以看出重复继承的一个不好的地方，同样的数据会存放多份。因此c++引出了虚继承。

再进一步来看看虚继承的情况，类定义：
```C++
class parent
{
public:
	virtual void f() { printf("parent f\n"); }
	virtual void g() { printf("parent g\n"); }

private:
	int a;
};

class parent1: virtual public parent
{
public:
	virtual void f() { printf("parent1 f\n"); }
	virtual void g1() { printf("parent1 g1\n"); }

private:
	int a;
};

class parent2: virtual public parent
{
public:
	virtual void f() { printf("parent2 f\n"); }
	virtual void g2() { printf("parent2 g2\n"); }

private:
	int a;
};

class son : virtual public parent1, virtual public parent2
{
public:
	virtual void f() { printf("son f\n"); }
	virtual void gs() { printf("son g\n"); }

private:
	int a;
};
```

*继续看看visual studio的虚函数表情况：*
```
class son	size(44):
	+---
 0	| {vfptr}
 4	| {vbptr}
 8	| a
	+---
	+--- (virtual base parent)
12	| {vfptr}
16	| a
	+---
	+--- (virtual base parent1)
20	| {vfptr}
24	| {vbptr}
28	| a
	+---
	+--- (virtual base parent2)
32	| {vfptr}
36	| {vbptr}
40	| a
	+---

son::$vftable@:
	| &son_meta
	|  0
 0	| &son::gs

son::$vbtable@son@:
 0	| -4
 1	| 8 (sond(son+4)parent)
 2	| 16 (sond(son+4)parent1)
 3	| 28 (sond(son+4)parent2)

son::$vftable@parent@:
	| -12
 0	| &son::f
 1	| &parent::g

son::$vftable@parent1@:
	| -20
 0	| &parent1::g1

son::$vbtable@parent1@:
 0	| -4
 1	| -12 (sond(parent1+4)parent)

son::$vftable@parent2@:
	| -32
 0	| &parent2::g2

son::$vbtable@parent2@:
 0	| -4
 1	| -24 (sond(parent2+4)parent)

son::f this adjustor: 12
son::gs this adjustor: 0
```

从结果中可以看到虚继承的内存布局会复杂一些，而且成员变量存储也一改之前的情况，改为子类存储在前。虽然避免了同样的数据存放多份，但是也为此付出了更多的内存存储来存放多个类自己的虚函数表及数据。也因此会添加调用开销，因为地址偏移更加复杂。

另外提一下析构函数最好声名为virtual，这样才能正确的按顺序调用对象的析构函数。因为如果不声明为virtual，那么当一个指向子类的基类指针被delete 释放的时候，就会只调用基类的析构函数，而不会按顺序调用子类的析构函数。


#### <code>**关于虚函数表的参考:**</code>
https://blog.csdn.net/haoel/article/details/3081328 （来自耗子叔）
https://blog.csdn.net/haoel/article/details/3081385 （来自耗子叔）
https://blog.csdn.net/fuzhongmin05/article/details/59112081
