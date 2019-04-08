---
title: auto src release template
date: 2019-04-08 16:15:08
tags: [c++]
categories: program
---

平时代码中经常会遇到资源管理的情况，既然申请了资源就必须在合适时候释放。但是这种配对会经常被处理的不完整，也就是会忘记释放，而且从代码整洁角度也会带来不适。但是我们可以利用c++ 的析构特性来适当设计资源的自动释放管理。同时利用c++11 引入的lamda 表达式，可以为我们提供更加优雅的设计。
<!--more -->

reference: [刘未鹏的现代c++语言设计](http://mindhacks.cn/2012/08/27/modern-cpp-practices/)


```c
#include <iostream>
#include <functional>

class ScopeGuard final
{
public:
    ScopeGuard(std::function<void()> f):_exit(f) {}
    ~ScopeGuard() { _exit(); }
    
    ScopeGuard(const ScopeGuard&) = delete;
    ScopeGuard& operator=(const ScopeGuard&) = delete;
private:
    std::function<void()> _exit;
};

#define EXIT_GUARD_NAMING(name, line) name##line
#define EXIT_GUARD(name, line) EXIT_GUARD_NAMING(name, line)
#define ON_EXIT(callback) ScopeGuard EXIT_GUARD(EXIT,__LINE__)(callback)



//usage
{
    std::ifstream ifs(filename);    //open one file for reading
    ON_EXIT([&] { ifs.close(); });  //auto close file when exit
}
```