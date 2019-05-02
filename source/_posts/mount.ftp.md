---
layout: post
title: centos挂载ftp目录
---
+ 安装`rpmforge-releae`  
去[某个地址](http://www.rpm-find.net/linux/rpm2html/search.php?query=rpmforge-release(x86-64))下载`rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm`，然后执行命令`rpm -Uhv rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm`  

+ 安装`curlftpfs`  
`$ yum install curlftpfs`  

+ 挂载  
将挂载后的地址先创建好，例：  
```Bash
$ mkdir -p /mod/t639/gmf
$ curlftpfs -o rw,allow_other ftp://nwpdata:123456@10.20.49.99/mod/t639/gmf /mod/t639/gmf
```

+ 卸载  
`$ fusermount -u /mod/t639/gmf`