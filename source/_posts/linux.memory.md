---
layout: post
title: linux内存分析
---
+ 分析top命令以及smaps  
讲得比较浅显易懂  
[link](http://www.360doc.com/content/16/0412/16/478627_550039413.shtml)

+ 如何设置swap大小  
根据centos官网介绍可以得出如下公式：M = Amount of RAM in GB, and S = Amount of swap in GB, then If M < 2, S = M *2 Else S = M + 2。而且其最小不应该小于32M(never less than 32 MB.)  
[link](https://www.centos.org/docs/5/html/5.1/Deployment_Guide/s1-swap-what-is.html)

+ linux内存介绍以及C与C++内存管理  
可以关注一下这个人的其他博文  
[link](http://blog.csdn.net/zwan0518/article/details/9040467)