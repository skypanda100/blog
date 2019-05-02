---
layout: post
title: 关闭selinux
---
+ 永久关闭selinux  
将`/etc/sysconfig/selinux`和`/etc/selinux/config`中的`SELINUX=enforcing`改为`SELINUX=disabled`  

+ 临时关闭selinux  
`$ setenforce 0`