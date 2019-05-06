---
layout: post
title: 用nginx的反向代理机制解决前端跨域问题
date: 2017-09-16 19:52:02
---
# 什么是跨域以及产生原因
------------------
跨域是指a页面想获取b页面资源，如果a、b页面的协议、域名、端口、子域名不同，或是a页面为ip地址，b页面为域名地址，所进行的访问行动都是跨域的，而浏览器为了安全问题一般都限制了跨域访问，也就是不允许跨域请求资源。
跨域情况如下：

|url          |说明          |是否跨域        |
|-------------|-------------|----------------|
|http://www.cnblogs.com/a.js<br>http://www.a.com/b.js|不同域名|是|
|http://www.a.com/lab/a.js<br>http://www.a.com/script/b.js|同一域名下不同文件夹|否|
|http://www.a.com:8000/a.js<br>http://www.a.com/b.js|同一域名，不同端口|是|
|http://www.a.com/a.js<br>https://www.a.com/b.js|同一域名，不同协议|是|
|http://www.a.com/a.js<br>http://70.32.92.74/b.js|域名和域名对应ip|是|
|http://www.a.com/a.js<br>http://script.a.com/b.js|主域相同，子域不同|是|
|http://www.a.com/a.js<br>http://a.com/b.js|同一域名，不同二级域名|是|


# 跨域的常见解决方法
------------------
目前来讲没有不依靠服务器端来跨域请求资源的技术
* jsonp 需要目标服务器配合一个callback函数。
* window.name+iframe 需要目标服务器响应window.name。
* window.location.hash+iframe 同样需要目标服务器作处理。
* html5的postMessage+ifrme这个也是需要目标服务器或者说是目标页面写一个postMessage，主要侧重于前端通讯。
* [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)需要服务器设置header ：Access-Control-Allow-Origin。(GZT使用的是这个方法)
* nginx反向代理 这个方法一般很少有人提及，但是他可以不用目标服务器配合，不过需要你搭建一个中转nginx服务器，用于转发请求。

# nginx反向代理解决跨域
------------------
上面已经说到，禁止跨域问题其实是浏览器的一种安全行为，而现在的大多数解决方案都是用标签可以跨域访问的这个漏洞或者是技巧去完成，但都少不了目标服务器做相应的改变，而我最近遇到了一个需求是，目标服务器不能给予我一个header，更不可以改变代码返回个script，所以前5种方案都被我否决掉。最后因为我的网站是我自己的主机，所以我决定搭建一个nginx并把相应代码部署在它的下面，由页面请求本域名的一个地址，转由nginx代理处理后返回结果给页面，而且这一切都是同步的。
假如www.a.com/html/msg.html想请求www.b.com/api/?method=1&para=2。

* www.a.com/html/msg.html如下：
```javascript
var url = 'http://www.b.com/api/msg?method=1&para=2'；
<br>$.ajax({
type: "GET",
url:url,
success: function(res){}
})
```

* 上面的请求必然会遇到跨域问题，这时我们需要修改一下我们的请求url，让请求发在nginx的一个url下：
```javascript
var url = 'http://www.c.com/proxy/html/api/msg?method=1&para=2'； //www.c.com是nginx主机地址
 $.ajax({
type: "GET",
url:url,
success: function(res){}
})　　
```

* nginx配置如下：
```javascript
location ^~/proxy/html/{
	rewrite ^/proxy/html/(.*)$ /$1 break;
	proxy_pass http://www.b.com/;
}
```
	1. ^~ /proxy/html/  
		用于拦截请求，匹配任何以 /proxy/html/开头的地址，匹配符合以后，停止往下搜索正则。
	2. rewrite ^/proxy/html/(.*)$ /$1 break;  
		* 代表重写拦截进来的请求，并且只能对域名后边的除去传递的参数外的字符串起作用，例如在www.c.com/proxy/html/api/msg?method=1&para=2中只对/proxy/html/api/msg重写。
		* rewrite后面的参数是一个简单的正则 ^/proxy/html/(.*)$，$1代表正则中的第一个()，$2代表第二个()的值,以此类推。
		* break代表匹配一个之后停止匹配。
	3. proxy_pass  
		既是把请求代理到其他主机。