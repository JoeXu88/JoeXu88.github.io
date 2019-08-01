---
title: code ref indx
date: 2019-05-28 17:06:55
tags: [c,c++]
categories: program
---

### time
* [部分函数使用参考](https://www.runoob.com/cplusplus/cpp-date-time.html)  
* [utc time covert](https://www.epochconverter.com/)
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


### string
* [字符串数字互转参考](https://blog.csdn.net/jiang111_111shan/article/details/80430281)  
* [除去字节序中0的字符操作](https://joexu88.github.io/2019/05/21/record-method-bytes-to-string-remove-zero/)



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
PS1='\[`[ $? = 0 ] && X=2 || X=1; tput setaf $X`\]\u@\h\[`tput sgr0`\]:$PWD\n\$ '

##will show like this:
ulu@localhost:/home/ulu/devlp/face_control_test_bin
```
[params explain site](https://wiki.archlinux.org/index.php/Bash/Prompt_customization_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
custom define refer site: http://ezprompt.net/  
