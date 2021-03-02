---
layout: post
title: 字节序
date: 2018-04-15 14:03:05
---
计算机硬件有两种字节序：
1. 大端字节序（big endian）
   高位字节在前，低位字节在后。0x12345678表示为12 34 56 78。
2. 小端字节序（little endian）
   低位字节在前，高位字节在后。0x12345678表示为78 56 34 12。

字节序的应用：
1. 主机字节序（HBO，Host Byte Order）
   不同的机器HBO不相同，与CPU设计有关，数据的顺序是由cpu决定的,而与操作系统无关。
2. 网络字节序（NBO，Network Byte Order）
   为了兼容性统一采取大端字节序，关于为什么会采用大端字节序而不是小端字节序，有这种说法，早期的网络非常不稳定，太容易丢失数据，为了尽量少地丢数据而采用大端字节序，因为即使丢了一部分数据那也只是丢失的低字节的数据^_^

网络字节顺序与本地字节顺序之间的转换函数：
1. htonl
   Host to Network Long
2. ntohl
   Network to Host Long
3. htons
   Host to Network Short
4. ntohs
   Network to Host Short

简单的查看本机字节序的代码：

第一段是看本机字节序，第二段是看网络字节序也就是小端字节序。

```c
#include <stdio.h>
#include <netinet/in.h>
int main()
{
    int i_num = 0x12345678;
    printf("[0]:0x%x\n", *((char *)&i_num + 0));
    printf("[1]:0x%x\n", *((char *)&i_num + 1));
    printf("[2]:0x%x\n", *((char *)&i_num + 2));
    printf("[3]:0x%x\n", *((char *)&i_num + 3));

    i_num = htonl(i_num);
    printf("[0]:0x%x\n", *((char *)&i_num + 0));
    printf("[1]:0x%x\n", *((char *)&i_num + 1));
    printf("[2]:0x%x\n", *((char *)&i_num + 2));
    printf("[3]:0x%x\n", *((char *)&i_num + 3));
 
    return 0;
} 
```

