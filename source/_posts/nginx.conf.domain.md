---
layout: post
title: 修改nginx配置文件允许跨域访问
date: 2017-10-09 12:30:31
---

```
location / {
	if ($request_method = 'OPTIONS') {
		add_header 'Access-Control-Allow-Origin' '*';
		add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
		#
		# Custom headers and headers various browsers *should* be OK with but aren't
		#
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
	root   /home/data;
	index  index.html index.htm;
}
```

