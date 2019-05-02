---
layout: post
title: 查看系统开机时间
---
+ 复杂的命令  
`$ date -d "$(awk -F. '{print $1}' /proc/uptime) second ago" +"%Y-%m-%d %H:%M:%S"`  

+ 简单的命令  
执行`top`，左上角就是开机时间