---
title:		C/C++调用Shell system()与popen()
date:		2021-06-17
author:	y1
header-img: img/header_of_index.jpg
catalog: true
tags:
    - C++
    - C
---



# C/C++调用Shell system()与popen()

​	最近在看一本英文书，有些单词需要查。在Arch上安装了个Terminal内的翻译软件感觉挺好用的，然后我就想着能少输入就少输入，把前面的几个parameter也给省略掉。

## 方法一：使用system()

> ## Description
>
> The C library function **int system(const char \*command)** passes the command name or program name specified by **command** to the host environment to be executed by the command processor and returns after the command has been completed.
>
> ## Declaration
>
> Following is the declaration for system() function.
>
> ```c
> int system(const char *command)
> ```
>
> ## Parameters
>
> - **command** − This is the C string containing the name of the requested variable.
>
> ## Return Value
>
> The value returned is -1 on error, and the return status of the command otherwise.

​	该函数无法获取shell返回值，通常仅用来执行一条命令语句。

## 方法二：使用popen()

> **popen** is used to read and write to a unix pipe.
>
>  This function is NOT included in 'C Programming Language' (ANSI) but can be found in 'The Standard C Library' book.
>
> ```c
> Library:   stdio.h
> 
> Prototype: FILE *popen(const char *command, const char *type);
> 
> Syntax:    FILE *fp;
> 	   fp = popen( "ls -l", "r");
> 
> Notes:
> 	command - is the command to be issued.
> 	type    - r - read O/P from command.
>                 - w - Write data as I/P to command.
> ```

​	`popen()`调用`fork()`创建子进程，由子进程调用shell命令，根据type值建立管道连接至子进程的标准I/O流，`popen()`返回一个FILE指针，然后就可以进行读写操作了。返回值是个标准I/Ol流，关闭FILE要用`pclose()`进行。

## 实作

```c++
#include <cstring>
#include <iostream>
#include <string>
using namespace std;

int main(int argc, char const *argv[]) {
  while (1) {
    string tmp;
    cin >> tmp;
    string word = "trans en:zh " + tmp;
    const char *cstr = word.c_str();
    FILE *fp;
    char buffer[80];
    fp = popen(cstr, "r");
    while (!feof(fp) && !ferror(fp)) {
      strcpy(buffer, "\n");
      fgets(buffer, sizeof(buffer), fp);
      printf("%s", buffer);
    }
    pclose(fp);
  }
  return 0;
}
```

