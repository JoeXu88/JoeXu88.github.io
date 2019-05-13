---
title: windows build ffmpeg
date: 2019-05-10 15:00:26
tags: [ffmpeg]
categories: multimedia
---

为了方便调试，故在windows上搭建FFmpeg 的调试环境。

## 系统编译环境
**OS**: <u>windows7 64bit</u>  
**ffmpeg dll build env**: <u>msys2</u>[windows环境下的Linux编译环境，使用mingw]; <u>yasm</u>[asm汇编使用]  
**develop env**: <u>visual studio 2017</u>
<!--more -->

Notes: msys2 是用来编译FFmpeg的库的，这些库用于visual studio 工程的链接使用，而visual studio并不能直接编译对应的库，因为没有对应的vs工程。  

* msys2的准备  
可以从[msys官网](https://www.msys2.org/)下载。  
另外这里需要注意编译过程中可能会遇到中文乱码的问题，解决方法是设置中文的encode 类型，位置是窗口的左上角点击图标下拉到options->
![设置中文编码](/img/articles/msys_set_chinese.png)

* 汇编yasm 的准备
从[yasm官网](http://yasm.tortall.net/Download.html)下载，用于asm 编译使用。  


## 编译FFmpeg库  
- **1.准备msys2 的编译环境**  
到安装目录下拷贝一份“msys2_shell.cmd” ，命名为“msys2_shell_vs2017.cmd”。同时编辑文件，在第一句“echo off” 之后添加visual studio的编译环境：
```
@echo off
call "c:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\VC\Auxiliary\Build\vcvars32.bat"
setlocal
......
```
*这个脚本的作用是在msys 启动后自动设置相应的环境变量，其中添加了visual studio的环境变量，包括库的位置等信息。* 但是需要注意的是PATH并不完整，需要自己在"~/.bashrc" 中添加如下环境变量：
```
export PATH=/c/Program\ Files\ \(x86\)/Microsoft\ Visual\ Studio/2017/Professional/VC/Tools/MSVC/14.15.26726/bin/Hostx86/x86/:$PATH
```

检测是否设置成功的方法：
```
$ echo $PATH
/c/Program Files (x86)/Microsoft Visual Studio/2017/Professional/VC/Tools/MSVC/14.15.26726/bin/Hostx86/x86/:/usr/local/bin:/usr/bin:/bin:/opt/bin:/c/Windows/System32:/c/Windows:/c/Windows/System32/Wbem:/c/Windows/System32/WindowsPowerShell/v1.0/:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl
$ which cl
/c/Program Files (x86)/Microsoft Visual Studio/2017/Professional/VC/Tools/MSVC/14.15.26726/bin/Hostx86/x86/cl
$ echo $LIB
c:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\VC\Tools\MSVC\14.15.26726\ATLMFC\lib\x86;c:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\VC\Tools\MSVC\14.15.26726\lib\x86;C:\Program Files (x86)\Windows Kits\NETFXSDK\4.6.1\lib\um\x86;C:\Program Files (x86)\Windows Kits\10\lib\10.0.17134.0\ucrt\x86;C:\Program Files (x86)\Windows Kits\10\lib\10.0.17134.0\um\x86;
```
如果有类似上面的结果就说明设置环境变量成功了，可以继续编译工作了。但是在此之前还需要进行一些系统库环境的补齐工作，可以命令行执行如下命令安装相应的库：    
```
$ pacman -S base-devel --needed
```

另外需要拷贝下载的yasm exe文件到/usr/bin/下，以用于汇编。  

- **2.编译库**  
双击"msys2_shell_vs2017.cmd" 打开命令行窗口。
FFmpeg的编译配置如下：  
```
./configure --enable-shared --disable-static --prefix=./vs2017_build_dll --extra-ldflags="/NODEFAULTLIB:libcmt" --extra-cflags="-MDd" --enable-debug --toolchain=msvc
```
然后执行：  
```
make
make install
```
然后相应的库安装在vs2017_build_dll目录下：  
```
--bin
  *.lib
  *.dll
  *.exe
--lib
  *.def
--include
--share
```


## visual studio设置  
新建一个空的工程 -> 添加实现的cpp 文件，或者直接从FFmpeg的example里面拷贝文件。
设置工程：
![include头文件设置](/img/articles/vs_ffmpeg_include_setting.png)
里面设置sdl 是为了减少一些warning的编译结果转换为error。  

![lib库依赖设置](/img/articles/vs_ffmpeg_lib_setting.png)
  
然后就可以进行调试了，但是还需要最后一步就是将编译好的库拷贝到visual studio工程的对象生成目录下，一般是Debug目录，因为运行和debug都需要用到。  
