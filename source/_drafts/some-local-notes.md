---
title: some local notes
tags: notes
---

shell notes:  
* /etc/profile.d 目录下的环境变量是 /etc/profile 启动时一起被读取，那么想要在当前shell终端临时生效可以使用 source /etc/profile，要全局生效则需要注销重登录或者直接重启系统，和 /etc/profile 原理一样;   /etc/profile.d 目录下的环境变量和 /etc/profile 的环境变量优先级，根据环境变量在 /etc/profile 的for循环语句调用 /etc/profile.d 的前面还是后面，在前则被 /etc/profile.d 目录下的环境变量覆盖，在后则被 /etc/profile 的环境变量覆盖  

* chrome secure shell: host chang => open secure shell app; then open console and type "term_.command.removeKnownHostByIndex(4)" if know indx or type "term_.command.removeAllKnownHosts()" if do not know indx  

<!--more -->
* centos 6 update gcc:  
$ sudo yum install centos-release-scl
$ sudo wget https://copr.fedorainfracloud.org/coprs/rhscl/devtoolset-3/repo/epel-6/rhscl-devtoolset-3-epel-6.repo -O /etc/yum.repos.d/rhscl-devtoolset-3-epel-6.repo
$ sudo yum install devtoolset-3-gcc devtoolset-3-gcc-c++  devtoolset-3-gdb
$ scl enable devtoolset-3 bash

refer: https://www.hi-linux.com/posts/25767.html


curl related:
[libcurl parallel](https://izualzhy.cn/use-curl-with-high-performance)
[libcurl basic infos](https://ec.haxx.se/how.html)


#### memcheck
*valgrind: valgrind --tool=memcheck --leak-check=full --log-file=./mem.log ./exec


boost related:
[boost source download](https://dl.bintray.com/boostorg/release/1.69.0/source/)  
[boost official site](http://www.boost.org/)  

#### 线程并发相关资料:  
[Sequencing and the concurrency memory model](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2052.htm)  
[singleton obj in multithread env](https://www.aristeia.com/Papers/DDJ_Jul_Aug_2004_revised.pdf)  
[The "Double-Checked Locking is Broken" Declaration](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)
[java memory model](http://tutorials.jenkov.com/java-concurrency/java-memory-model.html)  

templates:  
[Variadic Templates](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2087.pdf)

参数转发：  
[The Forwarding Problem: Arguments](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2002/n1385.htm)  

#### program notes:
* 打印不换行，当前行刷新进度：利用'\r' 回到行首，但是不换行
```c
printf("\r current percentage:%d%", per);

//output:
current percentage:10% //will update at current line
```
* "corrupted size vs. prev_size" glibc error: 一般是内存越界，可能写出了内存分配区域。
* 'PRIu64' not work in c++: <1> need add #include <inttypes.h>  <2> need add __STDC_FORMAT_MACROS to cxx_flag in makefile

#### c++ notes:
* 关于基类，如果基类只能被继承使用，而不能单独创建对象，可以将基类的构造函数声明为protected，这样基类的对象就不能够被显示的单独创建，而只能被继承后来创建。  
* 对象的声明:  
假设构造函数没有入参，那么申明的时候需要注意产生歧义：
```c
比如 
A a; //ok   
A a(); //error, will not produce obj A, but a function   
A a{}; //ok, can produce obj A  
A *a = new A(); //ok  
```

```c
假设如下：Derive: public Base  
Derive d{};  
Base b = d; //虚函数表不会改变，依然是base自己的虚函数表，也就是不会调用到可能子类的实现，因为编译器会在初始化和赋值assign之间做出折中，假设一个object 里面含有一个或者以上的虚函数表指针，那么原先子类的虚函数表不会被基类的初始化所改变
```
* c++ 通过指针或者引用来实现多态, 这就是面向对象的设计风格    
* 对于c++对象的理解
一个c++对象的内存只包含成员变量，以及虚函数表指针（如果有虚函数的话），另外还可能会有为了补齐字节对齐所带来的额外空间。而static, nonstatic 的非虚函数在编译期就已经指定完成，并且不占对象内存，所以在没有虚函数的情况下（也就是没有多态的情况下）是效率比较高的，但是在多态的情况下效率就被牺牲了。因为虚函数实现的调用是间接性的，也就是要通过指针访问虚函数表来间接访问访问函数，这就牺牲了效率。  
* virtual 函数其实可以用另外一个词语来解释，就是共享函数，可以给所有的子类来共享。而不同子类实现又打包了对应的一个共享的表，为节省存储空间这个表是要存储到一个公共的地方的，那就只能是全局数据区了。  
* 构造函数的未知行为  
类定义如果没有显示声明构造函数，那么编译器会**在需要的时候**生成一个默认的，这个默认的构造函数又分2种情况：trivial 和 nontrivial。trivial的版本其实没有什么具体行为，就是不需要做什么额外的添加动作，最终也不会调用；但是nontrivial的版本就是有所作为，因为编译器觉得无所作为是不正确的。nontrivial 的版本又称为被合成的版本，就是编译器添加了额外的工作量来完成默认构造函数的行为。而nontrivial的版本又分4种情况：1-包含类成员，且这个成员有默认的构造函数；2-继承了基类，且基类有默认的构造函数；3-包含虚函数，那么编译器会添加虚函数表，同时在构造函数里面分配一个指针并指向这个虚函数表；4-虚继承，这个情况会处理比较多的事情，主要是将基类共享给所有继承的子类，而不是在每个子类中添加基类的数据。*而且需要注意的是构造函数都不会初始化成员变量的值，这个需要程序员自己来完成。*  
* 拷贝构造函数  
拷贝构造函数的应用事实上给了编译器很大的操作空间，因为涉及到优化问题。拷贝构造函数的理解包括2个方面，一个是默认函数的产生，另一个是拷贝构造函数的调用。  
首先是函数的产生，如果程序没有显示的定义拷贝构造函数，那么编译器会根据需要产生一个默认的。这里需要与否有一个参考标准，就是bitwise 还是 memberwise。如果是bitwise就不用添加特殊操作代码，默认的bitcopy 就可以，而且高效；但是如果是memberwise就需要添加额外的操作代码来合成。那么什么是bitwise呢？俗称位域操作，可以直接memcpy 对象的内存，也就是可以直接bit copy，那么就是bitwise，这种情况编译器没有额外的member操作。而memberwise是编译器会额外添加member 操作来完成每一个对象的复制。和构造函数类似，以下四种情况就是memberwise的情形：1-含有虚函数； 2-含有成员类，且该类对象有default 拷贝构造函数需要调用；3- 虚继承基类；4- 继承了一个基类，且该基类含有显示的拷贝构造函数。上述这些情况可以归结为2点，一是有虚函数表需要编译器额外导入操作的情况，如果直接拷贝对象内存可能会出现问题（比如基类等于一个继承类，虚函数表地址不能直接拷贝，但是如果是同一个对象类型就可以直接拷贝虚函数表指针，统一对象可以bitcopy），二是有默认的显示定义拷贝构造函数，那么编译器需要合成操作，将继承的基类或者成员类的拷贝构造函数的调用添加到对应类的构造函数中（比如类定义中包含std::string类）。  
然后是拷贝构造函数的调用，有2种情形会发生拷贝构造函数的调用：1是显示赋值操作，如A a = b; A a(b); A a = A(x); 子类的；2是函数入参或者返回的时候，如果是传值，那么会发生拷贝构造函数的调用。重点关注下函数参数形式发生的拷贝构造。由于函数入参如果是传值，会根据需求产生临时对象，那么这个临时对象就会调用拷贝构造来完成对象的生成，然后操作这个临时对象；同时函数返回的时候如果直接返回函数体内的对象值，那么会调用拷贝构造函数重新在函数外生成对象，以避免堆栈数据的销毁造成的灾难。但是重点来了，这个是理论上的操作，现代的编译器会根据情况来进行优化，以抑制拷贝构造函数的调用，尽量提高效率。这里重点关注函数返回值的情形，因为入参由于堆栈的内存属性总是会产生临时对象，所以解决方法是最好传指针或者引用来避免产生临时对象，从而避免调用拷贝构造产生新对象。而函数返回的值对象就留给了编译器很大的发挥空间，有些编译器会直接将函数改造，本来返回对象值的，改成传入对象引用而返回void（比如void foo(X &__result)，同时调用这个未初始化对象引用的构造函数产生实际对象内存；有些编译器则另外调用拷贝构造函数来进行函数外的对象产生（c++11会调用move语意的拷贝构造函数来优化堆内存的使用）。  
*为了帮助编译器做优化，程序使用返回值方面的一个优化技巧：*
```c
X foo(int x, int y)
{
    /*method 1*/
    X x;
    //process x,y
    reuturn x;  //not good here, will contruct x, and call copy construct x to new obj returned

    /*method 2*/
    return X(x,y);  //return ctor directly, will be more efficient, no more need to construct and then call copy contruct.
}
```
一般而言，假如函数多有返回对象值的情况，那么定义一个拷贝构造函数会比较合理和必要，因为会触发NRV优化。  

* 构造函数初始化列表  
初始化列表主要是为了提高效率来设计的，如果不在初始化列表里面进行初始化成员，尤其是类成员，那么将会先产生临时对象然后在构造函数实现里面加入赋值拷贝构造函数调用，效率大打折扣。但是要注意的是初始化列表的顺序其实是类定义中的顺序，而不是列表里面声明的顺序，因此要注意他们初始化的可能性错误。并且初始化列表的操作是会在显示的用户定义代码之前的，也就是构造函数内容是在初始化列表操作之后的。    
* 关于对象成员变量  
成员变量分static 和 nonstatic，其中static是只有一份实体，存储在static数据区，但是nonstatic的存储在具体对象的内存中的。需要注意的是static的存储区域的特殊性，即使没有具体对象也可以访问，他和具体对象内存无关。另外需要注意的一个点就是成员取地址的方法：&A::x(返回的是x在A中的内存offset，因为变量存储在具体对象内存中，没有具体object也就没有实际地址可言，只能是offset); &a.x(返回的是对象a中x的具体内存地址). 由于取地址需要一些间接转换，所以需要通过编译器优化来加快访问效率。  
* 成员函数的理解  
成员函数包括static，nonstatic, virtual三个大的类型。无论是哪种函数类型，其实现都是放在代码段的，也就是函数实现都是有具体函数地址的。因此这点扩展下就是取非虚函数地址都是取的实际内存地址，而不是offset，但是取虚函数地址则是取的虚函数表中offset，然后根据offset拿到表中真正对应的函数实现地址。不同类的函数是怎么区分的呢，如果名字一样怎么办？其实就是通过mangling，加一些区分的字符来重新命名某一个类的某一个成员函数。那么调用不同的对象怎么区分结果呢？答案就是this指针，通过this指针传参来获取具体对象的成员，从而取得不同的结果。那么进一步，什么是this指针？this指针其实就是具体对象的内存首地址。比如定义了一个对象，那么取他的地址就是this指针地址。  
```c
A a;
A *b = new A();
a.func(); //will call function like mangle_A_func(&a)
b->func(); //will call function like mangle_A_func(b)
```
static函数，它不属于具体对象，可以直接通过类来call，也可以通过具体对象来call。但是static函数和其余2类函数最大的区别是它没有this指针参数，因为不属于具体对象，进而也只能处理static成员变量。同时static不能用来修饰虚函数，因为虚函数的目的是为了实现多态，必然需要用到this指针，也就是虚函数只能是成员函数，因此不能用static修饰。  
虚函数是用virtual修饰的nonstatic成员函数，其目的是为了实现多态。只要一个类定义中声名了虚函数，编译器就会自动帮我们生成一个虚函数表。而这个表是跟随这个类定义的，因此一旦类定义确定，虚函数表也就确定，并且会放到全局数据区用来后续索引。那么虚函数表咋实现多态呢？实际机制就是子类如果实现了基类的某个虚函数，那么虚函数表就会将基类的对应函数地址替换成子类的函数实现。而这个表一般是放在对象内存的最初位置，那么无论是基类还是子类都将访问这个虚函数表来访问对应的函数实现，也就实现了多态。
```c
base *b = new derive; //this 指针就是b，指向的其实是derive对象内存，虚函数表也是对应的derive的虚函数表
/*那么当基类指针指向了继承子类对象的时候发生了什么，首先访问的将是子类对象的内存地址，而子类虚函数表对应的实现可能包含了基类的实现（如果没有被覆盖）和子类的实现。
*那么当访问基类的函数实现的时候this指针又是子类对象，又怎么访问到基类的成员变量呢？其实在c++的一个标准中定义了这么个规则，就是子类的对象内存中必须保证基类对象的完整性。
*也就是说子类的对象内存中完整包含了基类的所有内存，包括成员变量。实际上子类的对象内存中紧接着虚函数表之后就会首先跟随基类的成员变量。
*因此可以通过子类的this指针来访问到对应offset上的基类成员变量值。如果子类成员变量名字和基类一样，如何来确定访问哪一个？一样，mangling字符处理会帮我们解决唯一性。
*/
```
* inline 函数的重新理解  
inline修饰的函数，编译器将会重新改造。其实就是将函数代码重新整理贴到调用地方，这样就省去了函数调用的消耗。但是由于展开后会产生很多扩展码，同时很多时候会产生大量的临时变量，这样就会大大增加代码体积。  
```c
inline int MAX(int i, int j);
int a,b;
MAX(a, b); //这样不会产生临时变量
MAX(MAX(12,3), 3); //这样会产生临时变量，会先将MAX(12,3)的结果放到一个临时变量中，然后再次调用外层MAX
```
* 虚函数表的初始化  
虚函数表的初始化是有时机考虑的。出发点是为了在构造函数中调用虚函数时能够取消虚函数机制。那么虚函数想要不在构造函数中起效，就要在适当时候设置虚函数表。因此虚函数表事实上需要在基类构造函数调用完成后才设置虚函数表，这样在基类的构造函数中调用虚函数就只会调用自身的实现，而不会指向其他的实现。同时虚函数表需要在初始化列表行为开始前设置完成，这样才能让成员类调用到正确的虚函数实现（如果有虚函数调用的话）。  
* 析构函数  
一个类的析构函数不一定都需要显示定义，可以根据需求来定义。在继承和类成员环境中，析构函数的调用顺序都是和构造反着来。注意的是基类的析构函数最好定义为虚函数，这样才会正确的从继承子类开始调用，不然可能会直接调用基类的析构，而不会调用到子类的析构。析构函数不要声明为纯虚函数，因为在继承链中，每一个继承顺序中的类的析构函数都会被调用到，那么纯虚函数就不能被call。  
* 全局和局部静态对象  
全局对象的初始化会被安插在main函数中user代码部分之前，并且销毁会被安插在main函数退出之前。也就是说类似_main 和 _exit 类似的函数扩展代码会被编译器安插到main函数的最开始和最后。  
另外局部静态对象并不能在最开始的时候就进行初始化，要到用的时候才进行初始化。为了实现静态变量只初始化一次，里面加入了临时的变量来判断是否初始化过，在销毁的时候也要先判断是否初始化过然后进行销毁操作。  
* 临时对象  
c++ 编译器会无形中会容易产生一些临时对象，比如参数，或者比如赋值操作。因此要时刻注意下尽量减少临时变量的产生，多用引用入参。  
* exception 处理  
支持exception处理的构造函数会变得更加复杂。但是有一点准则，只有完整构造的对象才会在exception时被析构。如果构造函数中发生exception，destruction不会被调用。  
* dynamic_cast  
动态类型检查所付出的代价要大于static_cast，因为其要在多态类的虚函数表中产生一个typeinfo，并拿出这个typeinfo来判断是否可以进行类型转换。不过也因为如此而安全很多，如果不符合转换的条件将不会返回对应的指针。同时动态监测也适用于reference，但是reference不像指针，如果失败会返回空指针，reference失败时候的做法是抛出一个exception。    
* 引用可以在对象中声名，在构造的时候初始化引用对象。  


#### effective c++  
* 拷贝构造函数和拷贝赋值函数的调用区别：如果对象还没有被构造，那么将会调用拷贝构造，比如X x = y; 相反如果对象已经构造，那么赋值函数将会被调用，比如X x; x = y;  
* 尽量用const，enum，inline 替代#define。
```c
不好的例子
#define MAX((a),(b)) (a)>(b)?(a):(b)
int a=5,b=0;
MAX(a++, b); //a 自增了2次
MAX(a++, b+10); //a 自增了1次

enum的好处
class A
{
private:
    enum {NUM = 5}; //类似于#define
    int x[NUM];
};
```
* 尽量多使用const，在类定义中使用const 修饰函数可以有很多好处，首先是告知其他程序员这个函数不允许修改类的non-static成员变量，其次可以处理const 对象调用该函数。如果不是const 修饰，那么将不允许const对象调用非const 函数。同时mutable 是和const 配合使用的关键词，由于某些变量可能会被频繁修改，即使是在const函数中，但是const函数又不允许修改，因此mutable修饰变量就可以允许const函数来修改该变量。  
const 函数调用non-const函数是不安全的，因为non-const函数可能会修改类变量。  
const 函数返回值可以避免一些意外的或者无意义的赋值，比如一些operator 重载的返回结果去做了赋值，相当于临时变量去赋值，没有实际意义。  
* 尽量使用类的初始化列表来对变量进行初始化，既提高效率也保证变量有初始值。但是需要注意的是这种情况下如果static类相互引用则可能导致一些由于初始化顺序未知的意外，这里就引出了singleton的设计。也即singleton中返回local static对象的引用，然后外部调用这个函数来获取static对象，这样就安全了。  
* 析构函数不能抛出异常，否则要么程序过早退出，要么会导致不确定的行为。所以如果有一个异常需要抛出和检测，那么最好放到另外一个member function中提供给外部调用，而不要放在析构函数中。  
* 最好不在构造和析构函数里面调用虚函数，因为那时候虚函数还未必完全定义完整，引发的行为也也可能不会是自己希望的。//但是如今的编译器已经可以正确择机完成虚函数表的制定，因此行为也是正确的  
* 赋值拷贝的时候要注意避开自我拷贝，否则可能引入一些意外导致的破坏，原则就是尽量不损害原先的对象状态。
```c
A& A::operator=(A &rhs)
{
    //做法1
    if(this == &rhs) return *this;
    else 
        ...
    
    //做法2
    //save origin val first
    val_orig = this->val;
    new_val = new val;
    this-val = new_val;
    return *this;
}
//此外如果要保证异常安全，可以考虑使用copy and swap，也就是现将this 对象拷贝到临时副本，再创建一个新的对象，然后swap 这2个对象
```
* 自己实现拷贝构造函数时候要注意拷贝所有内容，包括继承下来的基类的拷贝构造函数也要逐一调用。  
* 尽量使用智能指针来管理资源以避免资源泄漏，但是智能指针不能用来指向数组对象，因为智能指针底层是只调用的delete，而没有定义delete[]。另外需要注意的是智能指针的对象初始化最好是独立语句完成，不能为了简单而直接在类似函数参数这样的地方来初始化，否则由于执行语句的顺序的不确定性可能会导致内存泄漏。并且智能指针不能利用局部变量的地址，否则会产生二次销毁的情况。
```c
shared_ptr<A> a(new A);
func(a, funcp()); //ok, good way
func(shared_ptr(new A), funcp()); // not good, maybe call new A first, and then call funcp, and set object a to shared_ptr, but if funcp meet exception then a will not be released
```
* 设计接口要明确参数定义，易于使用。  
* 内置类型比如int，传参最好用传值，但是自定义的类传参最好用引用。这都是最好的效率的方法。但是对于函数返回如何选择返回对象值，还是引用或者其他呢？这个需要多考虑实际场景。但是有几个方式需要严禁的，比如返回非类本身或者类成员时候就不能返回其引用，因为如果是local变量出了函数作用域就会销毁，引用也就消失；如果是new 出来的对象，返回指针解引用后的对象会让调用者忽略delete从而造成内存泄漏；如果是static 的local对象，那么将不会产生多份，这个适用于单例模式。上述几种情况要么返回指针，要么返回值（宁愿耗费构造函数和析构函数代价）。  
* 尽量延后变量的定义，这样可以减少更多不必要的开销，比如预先定义一个类，调用了无意义的default 构造函数，如果中间还没使用遭遇了exception，那么还会耗费析构。在循环中，尽量类定义到循环体内，除非一些情况下明确知道构造和析构的代价要比赋值构造大。所以总之尽量延后变量的定义可以提高一些效率。  
* 尽量不要用转型，会遭遇多余的消耗，编译器可能会添加额外的操作，甚至临时对象。比如static_cast 会产生临时对象，转型后会调用临时对象的接口，需要注意下。  
```c
class base
{
   public:
   virtual display() {...}
};
class derive: public base
{
   public:
   virtual display() {static_cast<base>(*this).display(); ...} //将会通过临时对象副本调用base的display，而不是无限调用derive 本身的display，那样会无限递归
};
```
* inline 函数不是一定会被编译器接受改编，这个关键字只是一个申请，向编译器申请改造，但是编译器有理由拒绝。比如太过复杂的inline 函数或者虚函数，太过复杂，编译器会放弃改造，虚函数由于需要在执行期动态决定调用哪一个实现，所以虚函数没法申明为inline。类似，指针指向的函数通常不会被inline 化。  


##### more effective c++
* 不要对数组对象使用多态。因为数组元素的遍历是根据sizeof(element) * i + addressOfArray[0]，但是以下例子就行不通了：
```c
class base
{
   public:
   virtual void func();
   private:
   int a;
};
class derive: public base
{
   public:
   virtual void func();
   private:
   int a;
   int b;
};

sizeof(derive) is not equal to sizeof(base)

void testFunc(base c[]); //if pass base[] it will be ok, but if pass derive[] it will be not ok
```
* 带参数构造的对象数组定义：
```c
class EquipmentPiece {
public:
EquipmentPiece(int IDNumber);
...
};
EquipmentPiece bestPieces[10]; // 错误！没有正确调用 EquipmentPiece 构造函数
EquipmentPiece *bestPieces = new EquipmentPiece[10]; // 错误！与上面的问题一样

EquipmentPiece bestPieces[] = { // 正确, 提供了构造
EquipmentPiece(ID1), // 函数的参数
EquipmentPiece(ID2),
EquipmentPiece(ID3),
...,
EquipmentPiece(ID10)
};

//heap array
EquipmentPiece **ep = new EquipmentPiece* [10];
//或者使用using
using EquipmentPiecePtr = EquipmentPiece*;
EquipmentPiecePtr *epp = new EquipmentPiecePtr [10];
```
* 要谨慎设计隐士类型转换，不然可能会导致不可预知的结果。当构造只是简单的内置类型的参数的时候，最好能声明为explicit，禁止隐士转换。  
* new, operator new, placement new  
new 就是常用的创建对象操作符，会分配内存并且调用构造函数。  
operator new 其实相当于C 中的malloc，只是分配了内存，但是没有调用构造函数，因此要注意这个分配的对象地址，要自行调用析构函数，然后用operator delete 回收内存。其包含在<new> 头文件中。  
placement new 是允许在已有的内存上创建对象，比如已有一个对象内存buffer，然后可以用 new (buffer) A(param); 重新在buffer上创建对象A。  
* c++ 11 中用unique_ptr 替换了 auto_ptr [参考](https://www.geeksforgeeks.org/auto_ptr-unique_ptr-shared_ptr-weak_ptr-2/)，主要是为了防止拷贝，加强安全性。  
* 在构造函数中防止exception 而导致内存泄漏的好方法是将指针对象声明为智能指针类型。智能指针目的其实就是通过对象来管理资源，利用析构释放资源，防止内存泄漏。  
* 在析构函数中尽量不要将exception 传导到析构函数外。因为析构函数被调用是由于主动delete 或者 exception 引发的stack-unwind。如果是exception引发的析构调用，然而析构又导致exception将会自私系统调用terminate 终止程序。 所以需要在析构函数中去捕获exception 以终止向外传导，同时可以尽量完成能完成的事。  
* 捕获异常和函数调用的区别：异常的时候传入的参数会被强制复制，因为此时对象基本上都是要被销毁的，因此要拷贝到副本来操作，并且作为传值捕获exception的时候对象将会被拷贝2次，一次是抛出的时候，另外一次是传到catch执行的时候；异常一般不会对参数进行类型隐士转换，因为要精确匹配类型，但是函数参数可能会进行转换；异常会匹配最先适合的类型，因此如果一个处理派生类异常的 catch 子句位于处理基类异常的 catch 子句后面，编译器会发出警告；异常的效率会比函数调用慢，因为要进行参数拷贝。  
* 最好用引用捕获异常，因为传值会被拷贝2次并且可能会被slice掉(即派生类的异常对象被做为基类异常对象捕获时)，而传指针将会面临到底要不要销毁指针所指向的对象内存问题，如果是local变量的地址显然是不能销毁的，所以要尽量避免指针异常。  
* 为提高效率，可以采用懒惰和过度激情的方法。懒惰法是指尽量拖延推迟对象等资源的创建，等到实际用的时候才创建；过度激情法是指可以预见一些频繁使用或者大的内存，可以提前做好准备，比如存储一些buffer cache，或者提前计算或者分配内存资源。  
* 传参的时候可能会因为隐士的转换需求而产生临时对象，比如字符串数组转成std::string。然而临时对象是不允许修改的，所以c++中就禁止了非const 参数的隐士转换，以避免产生临时对象，也就禁止了这种情形的传参转换。然而如果是const 参数，产生临时对象又是允许的，因为const 本身不允许修改，所以只是引用临时对象内容并不影响什么，因此就允许了这种传参的隐士转换。  
```c
void func(std::string &str)
{
   ...
}
char str[] = "adc";
func(str); //编译不会通过，因为禁止这种情况的隐士转换，不管func函数里面是否有对str 临时对象的操作与否，如果函数改为const 参数，那么编译可以通过
```
* 智能指针 unique_ptr 不允许拷贝，因为没法保证指针的唯一性，同时没法考量拷贝的那个指针是否已经被删除，可能会引发多次删除指针，陷入未知的危险境地。  
* 标准库中的string 类是没有虚析构的，为了提高效率而这样设计。设计者甚至不希望使用string *。  
* 拷贝构造可以申明为虚函数，看实际情况而定。比如需要基类指针对象赋值，然而实际对象是子类的情况：
```c
base *b1 = new derive1();
base *b2 = new derive2();
*b1 = *b2; //not same derive class object, we need to use virtual copy ctor
```
* 纯虚类是不可以被实例化的，但是指针可以指向具体的实例化继承类。  


##### c++ 11  
* __func__ 的使用更加灵活，其原理是编译器会隐士的在函数定义内最开始的地方初始化__func__ 变量。可以用来初始化成员列表，但是不能用来作为函数参数的默认值，因为这个时候函数还没完全定义完。  
* __cplusplus 其实就是阻止编译器对函数名进行mangling，所以对于export C 接口的时候需要用到这个宏定义，并配合extern "C"。不同版本的c++ 对于__cplusplus 的定义值不同，比如c++11中就定义为201103L的整型值。  
* #error 可以预处理编译问题，告诉编译器在错误的条件下停止编译并给出错误提示，如：
```c
#if __cplusplus < 201103L
#error "please use c++11"
#endif
```
* {} 的使用：初始化变量，或者构造函数参数的传入。可以更多的使用{} 来代替()，因为使用场景更加丰富，也可以更好的区分() 下一些可能的歧义。比如在c++11中非static的成员变量就地初始化就不能使用()，但是可以使用{}，而一般情况下类初始化构造也可以使用{}来传参，所以{} 更加通用，可以尽量使用{}。而且如果是类定义传参可能会引起歧义，比如A a(); 这个表达式就可能被编译器翻译为返回A的函数a() ，而不是类A的定义，但是换成{}，A a{}; 就不会引起歧义。     
* override 和 final 关键字：这2个关键字都是用来修饰虚函数的（也可以修饰变量），目的是为了更好的管理虚函数的继承与实现，也更加易读和管理。override修饰虚函数就说明该函数必须覆盖实现，否则编译器会报错，在之前的c++标准中，如果没有覆盖实现编译器也不会报错，会自动连接到上一级的实现；final 修饰虚函数可以中途中断继承，也就是表示到目前的设计为止，如果继续继承实现，那么编译器将会报错。  
* using 继承构造函数。using base::base 在继承类定义中表示继承基类的构造函数，这样的话就可以省掉一些基类的构造调用，但是如果选择继承基类构造也意味着放弃继承类的默认构造函数：
```c
class base
{
   public:
   base(int a) {}
};
class derive :public base
{
   using base::base;
   public:
   void printf() {};
};

derive d; //compile error, default derive() ctor will be deleted
```
* 右值的引入：左值是具名的能取地址的变量，剩下的就是右值。lamda 表达式属于右值。另外右值中的纯右值和将亡值，纯右值就是不和名字挂钩的一些常量，比如数字，字符，或者基本类型；将亡值主要是函数返回值。  
std::move 的作用其实就是强行将左值转换为右值，类似于static_cast<T &&> x。但是对x 本身并没有什么操作，只是将左值变化为右值后调用右值拷贝构造的时候会偷走x 中的堆内存地址，并将x 的堆内存地址设置为null，因此用到move 语意的时候也最好是放弃对象使用周期的时候。  
```c
class A
{
public:
	A(int n) :x(n) { printf("A %d ctor\n", x); }
	~A() { printf("A %d dcotor\n",x); }
	A(const A& a):x(a.x) { printf("A %d cpy ctor\n", x); }
	A(A &&a) :x(std::move(a.x)) { a.x = -999; printf("A %d move cotr\n", x); }
	A& operator=(A &a) { printf("A equal ctor\n"); }
private:
	int x;
};

//test1
A test_a()
{
	A tmp(-1);
	return tmp;
}
A a = test_a();
//output of VS2017, gcc will optimize due to NRV
A -1 ctor
A -1 move cotr
A -999 dcotor
main return
A -1 dcotor

//test2
A test_a()
{
	return A(-1);
}
A a = test_a();
//output of VS2017
A -1 ctor
main return
A -1 dcotor

//test3
A&& test_a()
{
	return std::move(A(-1));
}
A a = test_a();
//output of VS2017
A -1 ctor
A -1 dcotor
A 2130567168 move cotr
main return
A 2130567168 dcotor

//test4
A&& test_a()
{
	return std::move(A(-1));
}
A &&a = test_a();
//output of VS2017
A -1 ctor
A -1 dcotor
main return

//test5
A&& test_a()
{
	A tmp(-1);
	return std::move(tmp);
}
A &&a = test_a();
//output of VS2017
A -1 ctor
A -1 dcotor
main return

//test6
A&& test_a()
{
	A tmp(-1);
	return std::move(tmp);
}
A a = test_a();
//output of VS2017
will crash, due to move ctor can not access x of tmp, because it is in function stack memory and it can not be accessed again out of function
```
* 默认的移动构造函数实现是简单的位拷贝复制，和拷贝构造一样。所以默认的是不够的，如果需要移动版本构造函数，需要自行实现。在c++ 11中必须同时提供赋值，拷贝，移动构造函数，否则只能实现一种。  
* 完美转发：std::forward，目的是将参数无损的转发到各阶段函数的参数上。主要是为了更好的利用move语意，以及右值引用。  
* initializer_list 可以使用到函数参数中，包括构造和普通函数。目的是更好的扩展使用初始化列表，同时可以防止类型收窄（就是类型向下转换，float转为int等）。同时注意{} 的使用的广泛性。  
* using 的语法更加灵活化，还可以精简模板类的书写。比如：
```c
template <typename T> using mapstr = std::map<T, std::string>
```
* auto 自动推导类型是重要的特性之一，可以多使用。其还可以和volatile const 配合使用。  
另外一个搭档delctype，接受表达式作为参数来判断类型。使用例子：
```c
int a,b;
decltype(a+b) c; //c is int

float func();
decltype(func()) d; //d is float
```
上述哥俩还可以和函数返回值追踪结合，更加适合泛化编程：
```c
template <typename T1, typename T2>
auto func(T1 a, T2 b) -> decltype(a+b)
{
   return a+b;
}
```
* for循环的加强:
```c
int action(int &a) {a *= 2;}
int a[5] = {1,2,3,4,5};
for_each(a, a+sizeof(a)/sizeof(a0), action);

std::vector<int> v;
for(auto x: v) { /*process*/ }
```
* 强类型枚举，只需在enum后面加上class：
```c
//加强了域名的作用，更加强类型，而且还可以指定类型
enum class Type:char { Male, Female};
```
* x86平台对于原子操作都是强顺序的，也就是都是按照顺序执行机器指令；但是对于PowerPC其实是弱顺序的，也就是不一定按照定义的原子操作顺序执行。在弱顺序的硬件平台上如果要保证执行的顺序性，必须在对应的汇编段加上内存栅栏（memory barrier）。 




rocksdb notes:
writebach 支持多个操作的原子性，但是不进行冲突的检查  
transaction 支持多个操作并发进行，并且进行冲突的检查，只有在没有冲突的情况下才会commit  

#### linux system
* fwrite 不是原子操作，是ansic c封装的函数接口；但是write是Linux系统操作，是原子的，写文件的时候还注意加上apend打开属性可以保证不同进程写文件的原子操作，多线程可能还需要加锁来保证一些日子的产生。  


#### hls 测试link  
http://playertest.longtailvideo.com/adaptive/bipbop/gear4/prog_index.m3u8  
https://devstreaming-cdn.apple.com/videos/streaming/examples/img_bipbop_adv_example_ts/master.m3u8
http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8  
http://devimages.apple.com/iphone/samples/bipbop/gear1/prog_index.m3u8  
LIVE：
http://live.3gv.ifeng.com/zixun.m3u8  


#### hls learning
https://www.afterdawn.com/glossary/term.cfm/mpeg2_transport_stream

#### tools
[mp4 parser online](http://mp4parser.com/)  


#### ffmpeg commands
* ffmpeg -i video.mp4 -c:v rawvideo -pix_fmt yuv420p out.yuv


#### some english words:  
adept: 熟练的 adj; 内行 n;  
adequately: 充分的 adj;  
succinctly: 简洁地  adv;  
agnostic: 不可知  
arithmetic: 算法，算术  
arbitrary : 任意的  
asymptotic: 渐进的  
axiom: 明理， 道理 n;   
bandwagon: 流行，时尚 n;  
burden: 负担，责任 n;  
concurrency: 并发性  
clause: 条款，法则；子句
consistent: 一致性  
cache coherence: 高速缓存一致性  
coherence： 一致性  
compromise : 妥协，折中  
concrete : 混凝土的，实际的，具体的  
contradictions: 矛盾  
critical thinking: 批判性思维  
degenerate: 退化的，堕落的 adj;  使退化 adv;  
distinction: 区别  
eliminatio: 消除; 排除  
elaborate : 精心制作的，精巧的  
entity: 实体  
Fallacy: 谬论  
flame: 火焰 n;  
flaw: 缺陷 n;  
hood: 头巾，覆盖 n;  
idiom: 习语；成语  
impose: 利用，欺骗  
impede: 妨碍，阻碍  
inherently: 内在地  
knack: 本领，窍门 n;  
literally: 字面地，逐字地，不夸张地 adv;  
lexicographically: 字典地，按字典顺序地  adv;  
mundane: 世俗的， 平凡的 adj;  
mortal: 凡人，人类 n; 凡人的，致死的 adj;  
nevertheless : 不过  
Obstacles: 障碍  
push around: 任由摆布  
polymorphically : 动态地  
prejudice: 偏见  n;  
preceding: 前述的，之前的 adj;  
proponents: 反对者 n;  
Purging: 清除 v;  清除，换气 n;  
suffice: 使满足，足够用  
trait: 特性，特质 n;  
trivial: 不重要的，琐碎的 adj;  
yield: 屈从，让步，出产  vt,vi; 产量 n;  
vendor: 供应商 n;  



#### 虚拟内存：  
虚拟内存是在大内存程序的背景下产生的，在现代操作系统上，每个应用程序都拥有一个完整的内存地址。假设是32bit 操作系统，那么每个应用程序都拥有4G 的内存地址空间。如果是大于实际物理内存空间，可以借用swap 机制，将磁盘拿来当内存，当然效率并不高，这个另当别论，至少操作系统支持这样的操作。 从这个角度看，所有应用程序内存加起来肯定是要远大于实际物理内存地址的，不过实际情况下会通过调度方式换出某些应用程序以给与让路。  
地址空间：
虚拟地址其实是逻辑地址，32bit 系统是从0 ~ 4G的分配，但并不是直接与物理地址挂钩，需要MMAP 的方式映射到实际物理地址，而这个映射实际上是页表一对一映射的过程（准确的说是页和页帧映射，页帧是对物理地址的分割）。以堆（malloc）的方式来说，首先malloc 一个堆内存，那么是分配到一个逻辑地址，这个地址由页号和偏移量组成。所谓的页号是内存分配中的基本操作单位，也就是将4G 内存分隔成多个页来操作分配以提高效率，而偏移量是实际数据内存大小，也就是能存多少数据。当分配一个堆内存后并不会立即挂钩物理地址，而是第一次使用这块内存时发现这个页没有挂钩到实际物理地址的时候，以缺页的方式陷入到内核中重新分配挂钩物理内存页帧。  
物理地址则好说，就是实际物理内存的地址空间。  


#### webrtc  
signal server example: https://www.tutorialspoint.com/webrtc/webrtc_signaling.htm
```js
/* node js example server code */
//require our websocket library 
var WebSocketServer = require('ws').Server;
 
//creating a websocket server at port 9090 
var wss = new WebSocketServer({port: 9090}); 

//all connected to the server users 
var users = {};
  
//when a user connects to our sever 
wss.on('connection', function(connection) {
  
   console.log("User connected");
	
   //when server gets a message from a connected user
   connection.on('message', function(message) { 
	
      var data; 
      //accepting only JSON messages 
      try {
         data = JSON.parse(message); 
      } catch (e) { 
         console.log("Invalid JSON"); 
         data = {}; 
      } 
		
      //switching type of the user message 
      switch (data.type) { 
         //when a user tries to login 
			
         case "login": 
            console.log("User logged", data.name); 
				
            //if anyone is logged in with this username then refuse 
            if(users[data.name]) { 
               sendTo(connection, { 
                  type: "login", 
                  success: false 
               }); 
            } else { 
               //save user connection on the server 
               users[data.name] = connection; 
               connection.name = data.name; 
					
               sendTo(connection, { 
                  type: "login", 
                  success: true 
               }); 
            } 
				
            break; 
				
         case "offer": 
            //for ex. UserA wants to call UserB 
            console.log("Sending offer to: ", data.name); 
				
            //if UserB exists then send him offer details 
            var conn = users[data.name];
				
            if(conn != null) { 
               //setting that UserA connected with UserB 
               connection.otherName = data.name; 
					
               sendTo(conn, { 
                  type: "offer", 
                  offer: data.offer, 
                  name: connection.name 
               }); 
            } 
				
            break;  
				
         case "answer": 
            console.log("Sending answer to: ", data.name); 
            //for ex. UserB answers UserA 
            var conn = users[data.name]; 
				
            if(conn != null) { 
               connection.otherName = data.name; 
               sendTo(conn, { 
                  type: "answer", 
                  answer: data.answer 
               }); 
            } 
				
            break;  
				
         case "candidate": 
            console.log("Sending candidate to:",data.name); 
            var conn = users[data.name];  
				
            if(conn != null) { 
               sendTo(conn, { 
                  type: "candidate", 
                  candidate: data.candidate 
               });
            } 
				
            break;  
				
         case "leave": 
            console.log("Disconnecting from", data.name); 
            var conn = users[data.name]; 
            conn.otherName = null; 
				
            //notify the other user so he can disconnect his peer connection 
            if(conn != null) { 
               sendTo(conn, { 
                  type: "leave" 
               }); 
            }  
				
            break;  
				
         default: 
            sendTo(connection, { 
               type: "error", 
               message: "Command not found: " + data.type 
            }); 
				
            break; 
      }  
   });  
	
   //when user exits, for example closes a browser window 
   //this may help if we are still in "offer","answer" or "candidate" state 
   connection.on("close", function() { 
	
      if(connection.name) { 
      delete users[connection.name]; 
		
         if(connection.otherName) { 
            console.log("Disconnecting from ", connection.otherName);
            var conn = users[connection.otherName]; 
            conn.otherName = null;  
				
            if(conn != null) { 
               sendTo(conn, { 
                  type: "leave" 
               });
            }  
         } 
      } 
   });  
	
   connection.send("Hello world"); 
	
});  

function sendTo(connection, message) { 
   connection.send(JSON.stringify(message)); 
}
```