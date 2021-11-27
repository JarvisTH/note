---
title:   GDB基本使用
comments: true
category: [linux, coredump]
keywords: [c++,gdb]
date: 2021-10-13 21:30:40
update: 2021-10-13 21:30:40
description:   GDB基本使用
tags: [c++,gdb]
---

# GDB基本使用

GDB调试的三种方式：

1. 目标板直接使用GDB进行调试。

2. 目标板使用gdbserver，主机使用xxx-linux-gdb作为客户端。

3. 目标板使用ulimit -c unlimited，生成core文件；然后主机使用xxx-linux-gdb ./test ./core。



调试程序：

```c
main.c:

#include <stdio.h>
#include <stdlib.h>
 
extern int sum(int value);
 
struct inout {
    int value;
    int result;
};

int main(int argc, char * argv[])
{
    struct inout * io = (struct inout * ) malloc(sizeof(struct inout));
    if (NULL == io) {
        printf("Malloc failed.\n");
        return -1;
    }

    if (argc != 2) {
        printf("Wrong para!\n");
        return -1;
    }

    io -> value = *argv[1] - '0';
    io -> result = sum(io -> value);
    printf("Your enter: %d, result:%d\n", io -> value, io -> result);
    return 0;
}

sum.c:

int sum(int value) {
    int result = 0;
    int i = 0;
    for (i = 0; i < value; i++)
        result += (i + 1);
    return result;
}
```



# 一、gdb调试

## 1.设置断点

- 设置断点可以通过b或者break设置断点，断点的设置可以通过函数名、行号、文件名+函数名、文件名+行号以及偏移量、地址等进行设置。格式为：

```
break 函数名

break 行号

break 文件名:函数名

break 文件名:行号

break +偏移量

break -偏移量

break *地址
```

- 查看断点，通过info break查看断点列表。

![img](https://img2018.cnblogs.com/blog/1083701/201809/1083701-20180920085853505-389571840.png)

- 删除断点通过命令包括：

```
delete <断点id>：删除指定断点

delete：删除所有断点

clear

clear 函数名

clear 行号

clear 文件名：行号

clear 文件名：函数名
```

- 断点还可以条件断住：

```
break 断点 if 条件；比如break sum if value==9，当输入的value为9的时候才会断住。

condition 断点编号：给指定断点删除触发条件

condition 断点编号 条件：给指定断点添加触发条件
```

![img](https://img2018.cnblogs.com/blog/1083701/201809/1083701-20180920090216497-1116462691.png)

可以看出，当入参为9的时候被断住，而入参为8的时候运行到结束。

- 断点还可以通过disable/enable临时停用启用。

  ```
  disable
  
  disable 断点编号
  
  disable display 显示编号
  
  disable mem 内存区域
  
   
  
  enable
  
  enable 断点编号
  
  enable once 断点编号：该断点只启用一次，程序运行到该断点并暂停后，该断点即被禁用。
  
  enable delete 断点编号
  
  enable display 显示编号
  
  enable mem 内存区域
  ```

  ## 2.断点commands高级功能

  大多数时候需要在断点处执行一系列动作，gdb提供了在断点处执行命令的高级功能commands。

  



参考：https://www.cnblogs.com/arnoldlu/p/9633254.html

