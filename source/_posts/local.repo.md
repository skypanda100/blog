---
layout: post
title: 创建本地yum源
---
在没有网络，但是拥有系统光盘或者系统u盘的情况下，可以创建本地yum源来安装一些软件。

以centos为例： 

+ 挂载光盘或u盘  
```Bash
$ mkdir -p /yum/system
$ mount  /dev/hdc  /yum/system
```
**上面的`/dev/hdc`是插入光盘或者u盘后的路径，通过`disk -l`查看**  

+ 创建local.repo，并添加内容，相关内容如下  
```Bash
$ cat /etc/yum.repos.d/local.repo
[base] 
name=local
baseurl=file:///yum/system
gpgcheck=0
enabled=1
```