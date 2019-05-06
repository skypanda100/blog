---
layout: post
title: short与long是限定符，不是基本数据类型
date: 2017-09-14 11:00:05
---
c语言只提供了几种基本数据类型
* char
* int
* float
* double

而short，long为限定符，用来限定整型,
```c
short int a;
long int b;
```
我们平常只是出于习惯把int给省略了。