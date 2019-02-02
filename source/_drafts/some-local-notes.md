---
title: some local notes
tags: notes
---

shell notes:  

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


<!--more -->

#### some english words:  
concurrency: 并发性  
clause: 条款，法则；子句
consistent: 一致性  
cache coherence: 高速缓存一致性  
coherence： 一致性  
arbitrary : 任意的  
compromise : 妥协，折中  
agnostic: 不可知  
Obstacles: 障碍  
impose: 利用，欺骗  
push around: 任由摆布  
entity: 实体  
distinction: 区别  
eliminatio: 消除; 排除  
idiom: 习语；成语  
polymorphically : 动态地  
elaborate : 精心制作的，精巧的  
Fallacy: 谬论  
concrete : 混凝土的，实际的，具体的  
yield: 屈从，让步，出产  
nevertheless : 不过  
arithmetic: 算法，算术  



#### 虚拟内存：  
虚拟内存是在大内存程序的背景下产生的，在现代操作系统上，每个应用程序都拥有一个完整的内存地址。假设是32bit 操作系统，那么每个应用程序都拥有4G 的内存地址空间。如果是大于实际物理内存空间，可以借用swap 机制，将磁盘拿来当内存，当然效率并不高，这个另当别论，至少操作系统支持这样的操作。 从这个角度看，所有应用程序内存加起来肯定是要远大于实际物理内存地址的，不过实际情况下会通过调度方式换出某些应用程序以给与让路。  
地址空间：
虚拟地址其实是逻辑地址，32bit 系统是从0 ~ 4G的分配，但并不是直接与物理地址挂钩，需要MMAP 的方式映射到实际物理地址，而这个映射实际上是页表一对一映射的过程（准确的说是页和页帧映射，页帧是对物理地址的分割）。以堆（malloc）的方式来说，首先malloc 一个堆内存，那么是分配到一个逻辑地址，这个地址由页号和偏移量组成。所谓的页号是内存分配中的基本操作单位，也就是将4G 内存分隔成多个页来操作分配以提高效率，而偏移量是实际数据内存大小，也就是能存多少数据。当分配一个堆内存后并不会立即挂钩物理地址，而是第一次使用这块内存时发现这个页没有挂钩到实际物理地址的时候，以缺页的方式陷入到内核中重新分配挂钩物理内存页帧。  
物理地址则好说，就是实际物理内存的地址空间。  
