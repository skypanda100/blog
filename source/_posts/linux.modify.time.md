---
layout: post
title: 修改系统时间
---
1. 修改时区
```Bash
$ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

2. 看看时间是否正确若不正确就按如下操作
```Bash
$ date {查看目前本地的时间}
$ hwclock --show {查看硬件的时间}
```

3. 如果硬件时间和系统时间不同，那就对硬件的时间进行修改
```Bash
$ #设置硬件时间为17年4月17日14点44分15秒
$ hwclock --set --date '2017-04-17 14:44:15'
$ #设置系统时间和硬件时间同步
$ hwclock --hctosys
$ #保存时钟
$ clock -w
```