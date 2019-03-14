---
title: some local notes
tags: notes
---

shell notes:  

boost related:
[boost source download](https://dl.bintray.com/boostorg/release/1.69.0/source/)  
[boost official site](http://www.boost.org/)  

<!--more -->
#### 线程并发相关资料:  
[Sequencing and the concurrency memory model](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2052.htm)  
[singleton obj in multithread env](https://www.aristeia.com/Papers/DDJ_Jul_Aug_2004_revised.pdf)  
[The "Double-Checked Locking is Broken" Declaration](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)
[java memory model](http://tutorials.jenkov.com/java-concurrency/java-memory-model.html)  

templates:  
[Variadic Templates](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2087.pdf)

参数转发：  
[The Forwarding Problem: Arguments](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2002/n1385.htm)  


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

* 多态的疑惑：
基类指针或者引用指向子类，那么将会可能访问子类函数实现（如果函数被覆盖的话）。而如果子类函数实现用了子类的私有变量也一样没有问题，因此多态的虚函数覆盖和子类私有变量无关，只是函数有关。  
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
类定义如果没有显示声明构造函数，那么编译器会**在需要的时候**生成一个默认的，这个默认的构造函数又分2种情况：trivial 和 nontrivial。trivial的版本其实没有什么具体行为，就是不需要做什么额外的添加动作；但是nontrivial的版本就是有所作为，因为编译器觉得无所作为是不正确的。nontrivial 的版本又称为被合成的版本，就是编译器添加了额外的工作量来完成默认构造函数的行为。而nontrivial的版本又分4种情况：1-包含类成员，且这个成员有默认的构造函数；2-继承了基类，且基类有默认的构造函数；3-包含虚函数，那么编译器会添加虚函数表，同时在构造函数里面分配一个指针并指向这个虚函数表；4-虚继承，这个情况会处理比较多的事情，主要是将基类共享给所有继承的子类，而不是在每个子类中添加基类的数据。*而且需要注意的是构造函数都不会初始化成员变量的值，这个需要程序员自己来完成。*  
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

* 构造函数初始化列表  
初始化列表主要是为了提高效率来设计的，如果不在初始化列表里面进行初始化成员，尤其是类成员，那么将会先产生临时对象然后在构造函数实现里面加入赋值拷贝构造函数调用，效率大打折扣。但是要注意的是初始化列表的顺序其实是类定义中的顺序，而不是列表里面声明的顺序，因此要注意他们初始化的可能性错误。并且初始化列表的操作是会在显示的用户定义代码之前的，也就是构造函数内容是在初始化列表操作之后的。    
* 关于对象成员变量  
成员变量分static 和 nonstatic，其中static是只有一份实体，存储在static数据区，但是nonstatic的存储在具体对象的内存中的。需要注意的是static的存储区域的特殊性，即使没有具体对象也可以访问，他和具体对象内存无关。另外需要注意的一个点就是成员取地址的方法：&A::x(返回的是x在A中的内存offset); &a.x(返回的是对象a中x的具体内存地址). 由于取地址需要一些间接转换，所以需要通过编译器优化来加快访问效率。  



rocksdb notes:
writebach 支持多个操作的原子性，但是不进行冲突的检查  
transaction 支持多个操作并发进行，并且进行冲突的检查，只有在没有冲突的情况下才会commit  


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



#### 虚拟内存：  
虚拟内存是在大内存程序的背景下产生的，在现代操作系统上，每个应用程序都拥有一个完整的内存地址。假设是32bit 操作系统，那么每个应用程序都拥有4G 的内存地址空间。如果是大于实际物理内存空间，可以借用swap 机制，将磁盘拿来当内存，当然效率并不高，这个另当别论，至少操作系统支持这样的操作。 从这个角度看，所有应用程序内存加起来肯定是要远大于实际物理内存地址的，不过实际情况下会通过调度方式换出某些应用程序以给与让路。  
地址空间：
虚拟地址其实是逻辑地址，32bit 系统是从0 ~ 4G的分配，但并不是直接与物理地址挂钩，需要MMAP 的方式映射到实际物理地址，而这个映射实际上是页表一对一映射的过程（准确的说是页和页帧映射，页帧是对物理地址的分割）。以堆（malloc）的方式来说，首先malloc 一个堆内存，那么是分配到一个逻辑地址，这个地址由页号和偏移量组成。所谓的页号是内存分配中的基本操作单位，也就是将4G 内存分隔成多个页来操作分配以提高效率，而偏移量是实际数据内存大小，也就是能存多少数据。当分配一个堆内存后并不会立即挂钩物理地址，而是第一次使用这块内存时发现这个页没有挂钩到实际物理地址的时候，以缺页的方式陷入到内核中重新分配挂钩物理内存页帧。  
物理地址则好说，就是实际物理内存的地址空间。  
