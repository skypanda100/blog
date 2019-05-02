---
layout: post
title: npm更换源
---
有很多方法来配置npm的registry地址，下面根据不同情境列出几种比较常用的方法。
以淘宝npm 镜像举例： 

1. 临时使用
```Bash
$ npm --registry https://registry.npm.taobao.org install express  
```

2. 持久使用
```Bash
$ npm config set registry https://registry.npm.taobao.org
```
配置后可通过下面方式来验证是否成功
```Bash
$ npm config get registry 
```
或者
```Bash
$ npm info express
```

3. 通过cnpm使用，这也是我自己常使用的 
```Bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org 
```

[http://www.jianshu.com/p/0deb70e6f395](http://www.jianshu.com/p/0deb70e6f395)