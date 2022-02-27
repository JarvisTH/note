---
title:   likely与unlikely
comments: true
category: [c++, like]
keywords: [c++, likely, unlikely]
date: 2021-10-16 21:30:40
update: 2021-10-16 21:30:40
description:   likely与unlikely
tags: [c++, likely,unlikely]
---



# likely与unlikely

在c++程序中，可能会遇见likely、unlikely语句，这个也是第一次接触，所以去查了下资料。

这两个宏在内核中定义如下：

```c++
#define likely(x)       __builtin_expect((x),1)
#define unlikely(x)     __builtin_expect((x),0)
```

__builtin_expect() 是 GCC (version >= 2.96）提供给程序员使用的，目的是将“分支转移”的信息提供给编译器，这样编译器可以对代码进行优化， **以减少指令跳转带来的性能下降**。

```c++
__builtin_expect((x),1) 表示 x 的值为真的可能性更大；
__builtin_expect((x),0) 表示 x 的值为假的可能性更大。
```

也就是说，**使用 likely() ，执行 if 后面的语句 的机会更大，使用unlikely()，执行else 后面的语句的机会更大**。

从上面可以看出：

-  if(likely(value)) 等价于 if(value)

- if(unlikely(value)) 也等价于 if(value)

  **likely() 和 unlikely() 从阅读和理解代码的角度来看，是一样的**

通过这种方式，**编译器在编译过程中，会将可能性更大的代码紧跟着起面的代码，从而减少指令跳转带来的性能上的下降**。

