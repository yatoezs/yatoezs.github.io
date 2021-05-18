---
title:		C++信号处理 Signal() & raise()小记
date:		2021-05-18
author:	y1
header-img: img/header_of_index.jpg
catalog:	true
tags:
    - C++
    - Linux
    - Function
---

# C++信号处理 Signal() & raise()小记

​	Linux下开启Taskmanager选定一个task会有三个选项：Stop、Terminate、Kill。

> ​	It sends different stop signals to a process. Here's some info:
>
> - Stop: *SIGSTOP* - This signal makes the operating system **pause** a process's execution. The process **cannot ignore** the signal.
> - Kill: *SIGKILL* - The SIGKILL signal forces the process to **stop executing immediately**. The program **cannot ignore** this signal. This process does not get to clean-up either.
> - Terminate: *SIGTERM* - *This signal requests a process to stop running.* **This signal can be ignored.** The process is given time to gracefully shutdown. When a program  gracefully shuts down, that means it is given time to save its progress  and release resources. In other words, it is not forced to stop. SIGINT  is very similar to SIGTERM.

​	Terminate信号是能捕获并处理的，可以捕获后什么也不做，比如这样

```c++
#include <signal.h> 

#include <stdio.h> 

#include <unistd.h> 

static void signalHandle(int sig) {
  printf("caught signal %d, but do nothing.\n", sig);
}
int main(int argc, char const *argv[]) {
  signal(SIGTERM, signalHandle);
  int a = 0;
  printf("Any key to exit.\n");
  scanf("%d", &a);
  return 0;
}
```

终端下执行

```bash
➜   ./signal
Any key to exit.
caught signal 15, but do nothing.
1
```

用raise函数生成信号

```c++
#include <signal.h>

#include <stdio.h>

static bool stop = false;
static void signalHandle(int sig) {
  stop = true;
  printf("caught signal %d\n", sig);
}

int main(int argc, char const *argv[]) {
  signal(SIGINT, signalHandle);
  printf("Going to raise a signal\n");
  raise(SIGINT);
  return 0;
}
```

