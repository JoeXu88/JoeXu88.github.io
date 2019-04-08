---
title: time cost template
date: 2019-04-08 15:56:41
tags: [c++, time]
categories: program
---

c++11 标准库中引入了std::chrono 类，可以方便管理时间相关的计算。这里记录一个计算时间消耗的模板类。
<!--more -->

```c
#include <chrono>
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <functional>

using namespace std::chrono;
using std::string;

template <typename T>
class TimeCost final
{
public:
    TimeCost(string name, string unit):_name(name), _unit(unit), _start(system_clock::now()) {}
    void start() { _start = system_clock::now(); _showAtDctor = false; }
    void end() { show_cost(); }

    ~TimeCost() { if(_showAtDctor) show_cost(); }

    TimeCost(const TimeCost&) = delete;
    TimeCost& operator=(const TimeCost&) = delete;

private:
    void show_cost()
    {   
        _cost = duration_cast<T>(system_clock::now() - _start);
        std::cout << _name << " TimeCost "<< _cost.count() << _unit << std::endl;
    }   

    string _name;
    string _unit;
    system_clock::time_point _start;
    T _cost;
    bool _showAtDctor = true;
};


//-------------------------------------------------
//usage 1
{
    TimeCost<std::chrono::milliseconds> cost("do something", "ms");
    //do some job
}
//usage 2
{
    TimeCost<std::chrono::milliseconds> cost("do something", "ms");
    cost.start();
    //do some job
    cost.end();
}
//-------------------------------------------------

//时间刻度参考
std::chrono::nanoseconds	duration</*至少 64 位的有符号整数类型*/, std::nano>
std::chrono::microseconds	duration</*至少 55 位的有符号整数类型*/, std::micro>
std::chrono::milliseconds	duration</*至少 45 位的有符号整数类型*/, std::milli>
std::chrono::seconds	duration</*至少 35 位的有符号整数类型*/>
std::chrono::minutes	duration</*至少 29 位的有符号整数类型*/, std::ratio<60>>
std::chrono::hours	duration</*至少 23 位的有符号整数类型*/, std::ratio<3600>>
std::chrono::days (C++20 起)	duration</*至少 25 位的有符号整数类型*/, std::ratio<86400>>
std::chrono::weeks (C++20 起)	duration</*至少 22 位的有符号整数类型*/, std::ratio<604800>>
std::chrono::months (C++20 起)	duration</*至少 20 位的有符号整数类型*/, std::ratio<2629746>>
std::chrono::years (C++20 起)	duration</*至少 17 位的有符号整数类型*/, std::ratio<31556952>>
```
