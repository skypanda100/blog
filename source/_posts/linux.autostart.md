---
layout: post
title: Linux开机自启
date: 2018-02-07 22:28:44
---
* /etc/rc.local
  
  执行的程序需要写绝对路径。
  
* shell脚本

  将写好的脚本（.sh文件）放到目录 /etc/profile.d/ 下，系统启动后就会自动执行该目录下的所有shell脚本。

* chkconfig
  
  将启动文件cp到 /etc/init.d/或者/etc/rc.d/init.d/（前者是后者的软连接）下，vim 编辑文件，编辑完成后执行`chkconfig --add 脚本文件名` 操作后就已经添加了，文件前面务必添加如下三行代码，否侧会提示chkconfig不支持：
  
  1. #!/bin/sh	// 告诉系统使用的shell,所以的shell脚本都是这样
  2. #chkconfig: 35 20 80	// 分别代表运行级别，启动优先权，关闭优先权，此行代码必须
  3. #description: http server	//（自己随便发挥），此行代码必须

