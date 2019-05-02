---
layout: post
title: linux快速删除文件
---
使用rsync命令。
这也是我被网上的一些资料坑的地方。

坑的过程如下：

第一次使用的命令
```Bash
$ rsync --delete-before -a -H -v /root/test ./bak
```
其中，bak为要删除的目录，结果执行后，发现./bak目录下的文件没有删除，反而还将/root/test的空目录复制了过来。
后来，多次验证下，才发现应该这么写：
```Bash
$ rsync --delete-before -a -H -v /root/test/ ./bak 
```
