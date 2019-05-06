---
layout: post
title: centos7以上添加端口
date: 2017-08-02 10:26:31
---
1. 先查看firewalld是否开启
```Bash
$ systemctl status firewalld
```

2. 要是没有开启则开启
```Bash
$ systemctl start firewalld
```

3. 假设添加8000端口
```Bash
$ firewall-cmd --permanent --add-port=8000/tcp
```

4. 重启firewalld
```Bash
$ systemctl restart firewalld
```

5. 查看是否正确开启了端口
```Bash
$ firewall-cmd --query-port=8000/tcp
```

5. firewalld开机自启
```Bash
$ systemctl enable firewalld
```