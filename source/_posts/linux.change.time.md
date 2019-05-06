---
layout: post
title: 修改linux系统时间而不用重启
date: 2017-10-20 22:00:54
---
**查看时间是否正确，若不正确再修改**
* 查看目前本地的时间  
```
date
```
* 查看硬件的时间  
```
hwclock --show
```

**如果硬件时间和系统时间不同，那就对硬件的时间进行修改**
* 修改时区  
```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
* 设置硬件时间为14年12月15日15点15分15秒  
```
hwclock --set --date '2017-04-17 14:44:15' 
```
* 设置系统时间和硬件时间同步  
```
hwclock --hctosys
```
* 保存时钟  
```
clock -w
```