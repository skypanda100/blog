---
layout: post
title: nginx相关的一些命令
date: 2017-05-20 14:00:25
---
+ 检查nginx的配置  
`$ nginx -t`  

+ 启动nginx  
`$ nginx  -c /usr/local/nginx/conf/nginx.conf`  
**conf位置得自己找找**  

+ 重启nginx  
`$ nginx  -s reload`  

+ 启动cgi  
`$ spawn-fcgi -a 127.0.0.1 -p 9002 -C 25 -f /home/zhengdongtian/CLionProjects/Bump/cmake-build-debug/Bump`  
**9002是对应的conf中的端口**