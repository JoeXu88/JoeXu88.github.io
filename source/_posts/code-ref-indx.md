---
title: code ref indx
date: 2019-05-28 17:06:55
tags: [c,c++]
categories: program
---

### time
* [部分函数使用参考](https://www.runoob.com/cplusplus/cpp-date-time.html)  
* utc 转日期时间等  
<!--more -->
```c
inline std::string utc2date(const time_t &rawtime /*unit:s*/)
{
    struct tm *tinfo = std::localtime(&rawtime);
    char buffer[30];
    strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", tinfo);
    return std::string(buffer);
}
```
* [计算时间消耗](https://joexu88.github.io/2019/04/08/time-cost-template/)


### string
* [字符串数字互转参考](https://blog.csdn.net/jiang111_111shan/article/details/80430281)  
* [除去字节序中0的字符操作](https://joexu88.github.io/2019/05/21/record-method-bytes-to-string-remove-zero/)



### 文件IO
* [使用ifstream和getline读取文件内容](https://blog.csdn.net/xubuwei/article/details/88978325)
* 判断文件夹存在  
```c
static bool checkExist(const std::string &path)
{   
    DIR* dir = opendir(path.c_str());
    if(dir) { closedir(dir); return true; }
    else if(ENOENT == errno) return false;
}
```
* 删除文件及文件夹  
```c
#include <stdio.h> //delete file
#include <unistd.h> //delete dir
#include <errno.h>

int ret = remove(file_name);
int ret = rmdir("/home/cnd/mod1");
printf("remove err:%s\n", strerror(errno)); //check err
```

### shell
* 配置多个gcc 版本  
```js
#!/bin/bash

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 20
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 30

sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 10
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 20
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.9 30

sudo update-alternatives --set cc /usr/bin/gcc
sudo update-alternatives --set c++ /usr/bin/g++

sudo update-alternatives --config gcc
sudo update-alternatives --config g++
```