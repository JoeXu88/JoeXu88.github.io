---
title: code ref indx
date: 2019-05-28 17:06:55
tags: [c,c++]
categories: program
---

### time
* [部分函数使用参考](https://www.runoob.com/cplusplus/cpp-date-time.html)  
* [online utc time covert](https://www.epochconverter.com/)
* utc 转日期时间等  
<!--more -->
```c
#include <chrono>
#include <time.h>

inline std::string utc2date(const time_t &rawtime /*unit:s*/)
{
    struct tm *tinfo = std::localtime(&rawtime);
    char buffer[30];
    strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", tinfo);
    return std::string(buffer);
}

inline int getUTC_Seconds()
{
    return std::chrono::duration_cast<std::chrono::seconds>(std::chrono::system_clock::now().time_since_epoch()).count();
}

inline time_t convertStrToUTC(const char* str)
{
    int yy, mm, dd, hour, min, sec;
    sscanf(str, "%d-%d-%d %d:%d:%d", &yy, &mm, &dd, &hour, &min, &sec);

    struct tm timeinfo;
    timeinfo.tm_year = yy - 1900;
    timeinfo.tm_mon  = mm - 1;
    timeinfo.tm_mday = dd; 
    timeinfo.tm_hour = hour;
    timeinfo.tm_min  = min;
    timeinfo.tm_sec  = sec;

    return mktime(&timeinfo);
}

void test_utc_equal2time()
{
	const char* start_time = "1970-01-01 1:0:0"; //ignore yy:mm:day, use hour:min:second
	const int HOURS24 = 24 * 60 * 60;
	time_t start = convertStrToUTC(start_time);
	int dist_now = (getUTC_Seconds() - timezone)%HOURS24; 
	int dist_start = (start - timezone)%HOURS24; //seconds since 00:00:00 in one day

	while(dist_now != dist_start)
	{
		printf("\r dist now:%d, start:%d", dist_now, dist_start);
		dist_now = (getUTC_Seconds() - timezone)%HOURS24;
		dist_start = (start - timezone)%HOURS24;
	}
	printf("\nnow: %d\n", getUTC_Seconds());
}

```
* [计算时间消耗](https://joexu88.github.io/2019/04/08/time-cost-template/)

### 优秀开源项目  
[open source projects](https://www.ezlippi.com/blog/2014/12/c-open-project.html)
[nxweb http server](http://nxweb.org/)

### 算法学习
[labuladong algo](https://labuladong.gitbook.io/algo/)

### 代码优化
* [并行计算优化](https://blog.csdn.net/xubuwei/article/details/103478227)

### string
* [字符串数字互转参考](https://blog.csdn.net/jiang111_111shan/article/details/80430281)  
* [除去字节序中0的字符操作](https://joexu88.github.io/2019/05/21/record-method-bytes-to-string-remove-zero/)
* gb2312 中文url 编码 [可以参考文章](https://www.cnblogs.com/xiaoka/articles/2585189.html)
```c
string urlEncGB2312(const char * str)
{
    string dd;
    size_t len = strlen(str);
    for (size_t i=0;i<len;i++)
    {
        if(isalnum((unsigned char)str[i]))
        {
            char tempbuff[2];
            sprintf(tempbuff,"%c",str[i]);
            dd.append(tempbuff);
        }
        else if (isspace((unsigned char)str[i]))
        {
            dd.append("+");
        }
        else
        {
            char tempbuff[4];
            sprintf(tempbuff,"%%%X%X",((unsigned char*)str)[i] >>4,((unsigned char*)str)[i] %16);
            dd.append(tempbuff);
        }

    }
    return dd;
}
```

### 打印
* c++使用PRID64 （[参考文章](http://www.voidcn.com/article/p-cxeykvnw-rc.html)）
1. 包含头文件：<inttypes.h>
2. 定义宏：STDC_FORMAT_MACROS，可以通过编译时加-DSTDC_FORMAT_MACROS，或者在包含文件之前定义这个宏。


### 文件IO
* [使用ifstream和getline读取文件内容](https://blog.csdn.net/xubuwei/article/details/88978325)
* [fstream文件读写操作](https://blog.csdn.net/kingstar158/article/details/6859379)
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

### 资源管理  
* [退出作用域自动消除资源](https://joexu88.github.io/2019/04/08/auto-src-release-template/)

### 多线程  
* [C和C++中的volatile、内存屏障和CPU缓存一致性协议MESI](http://blog.chinaunix.net/uid-20682147-id-5817710.html#_Toc27148_WPSOffice_Level1)


### 网络  
* [libcurl multi thread problems](https://curl.haxx.se/libcurl/c/threadsafe.html)
* [libcurl threaded-ssl](https://curl.haxx.se/libcurl/c/threaded-ssl.html)
* [libcurl parallel](https://izualzhy.cn/use-curl-with-high-performance)
* [libcurl basic infos](https://ec.haxx.se/how.html)

### 编译（及汇编）  
* [cpu 指令编译选项](https://www.cnblogs.com/aios/p/9955339.html)
* [IA-32处理器基本功能](https://blog.csdn.net/qq_36982160/article/details/84033068)


### shell
* ubuntu 配置多个gcc 版本  
```js
#!/bin/bash

#number after version is priority, higher is prefered
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
* set bash prompt
```js
edit ~/.bashrc or /etc/profile to add following:
PS1='\[`[ $? = 0 ] && X=2 || X=1; tput setaf $X`\][\d \t] \u@\h\[`tput sgr0`\]:$PWD\n\$ '
PS1="\[\e[m\]\[\e[32m\][\[\e[m\]\[\e[32m\]\d\[\e[m\]\[\e[32m\] \[\e[m\]\[\e[32m\]\A\[\e[m\]\[\e[32m\]]\[\e[m\]\[\e[32m\] \[\e[m\]\[\e[32m\]\u\[\e[m\]\[\e[32m\]@\[\e[m\]\[\e[32m\]\h\[\e[m\]\[\e[32m\]:\[\e[m\]\w\[\e[m\]\n\$ "
"\[\e[m\]" 可以禁止颜色被改，有时候终端出现错误可能会有红色error出现，禁止被篡改为最后的颜色

##will show like this:
ulu@localhost:/home/ulu/devlp/face_control_test_bin
```
[params explain site](https://wiki.archlinux.org/index.php/Bash/Prompt_customization_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
custom define refer site: http://ezprompt.net/  
