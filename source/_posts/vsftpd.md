---
layout: post
title: 修改系统时间
date: 2018-10-24 17:00:25
---
1. 本地安装
```Bash
$ yum localinstall vsftpd-3.0.2-28.el7.x86_64.rpm
```
2. 关闭SELINUX
```Bash
$ setenforce 0
```
```Bash
#修改为SELINUX=disabled
$ vi /etc/sysconfig/selinux
```
3. 修改vsftpd配置文件
```Bash
#添加pasv_min_port=4000，被动模式最小端口
#添加pasv_max_port=4100，被动模式最大端口
#将chroot_local_user=YES的注释去掉，将本地用户限制到自己的主目录
$ vi /etc/vsftpd/vsftpd.conf
```
4. 添加用户
```Bash
$ useradd ftp_cmacast
#密码设置为cmacast112
$ passwd ftp_cmacast
```
5. 将该用户设置为不可登录
```Bash
$ usermod -s /sbin/nologin ftp_cmacast
```
6. 创建目录
```Bash
$ mkdir -p /mnt/data
$ chmod 777 -R /mnt
```
7. 设置用户的主目录
```Bash
$ usermod -d /mnt/data ftp_cmacast
```
8. 设置防火墙
```Bash
$ systemctl restart firewalld
$ firewall-cmd --permanent --add-port=20-21/tcp
$ firewall-cmd --permanent --add-port=4000-4100/tcp
$ firewall-cmd --reload
```
9. 启动ftp
```Bash
$ systemctl restart vsftpd
```