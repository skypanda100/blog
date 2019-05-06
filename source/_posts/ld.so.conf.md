---
layout: post
title: 向环境中添加动态库路径
date: 2017-05-19 21:15:20
---
如果想改变`LD_LIBRARY_PATH`的值，如下操作：  
+ 在`/etc/ld.so.conf`中添加你的path  
+ 执行`ldconfig`