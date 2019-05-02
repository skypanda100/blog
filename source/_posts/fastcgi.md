---
layout: post
title: nginx + spawn-fastcgi + fcgi
---
**以下配置应该针对自己环境进行相应改变**
* nginx  
  安装略过。  
  nginx.conf配置如下，其中request_method配置是为了允许跨域访问：
  ```
  location ~ modelsection\.cgi$ {
      if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
      }
      if ($request_method = 'POST') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
      }
      if ($request_method = 'GET') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
      }
      fastcgi_pass   127.0.0.1:9003;
      fastcgi_index  index.cgi;
      fastcgi_param  SCRIPT_FILENAME  fcgi$fastcgi_script_name;
      include        fastcgi.conf;
  }
  ```

* spawn-fastcgi  
  下载地址[http://www.lighttpd.net/download/spawn-fcgi-1.6.2.tar.gz](http://www.lighttpd.net/download/spawn-fcgi-1.6.2.tar.gz)  
  编译略过

* fastcgi  
  下载地址[https://download.csdn.net/download/wangkangluo1/3268722](https://download.csdn.net/download/wangkangluo1/3268722)  
  编译略过

**运行命令：`spawn-fcgi -a 127.0.0.1 -p 9003 -f ./modelsection`**


***参考地址：***
[https://blog.csdn.net/ygm_linux/article/details/51261288](https://blog.csdn.net/ygm_linux/article/details/51261288)


