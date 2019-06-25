---
layout: post
title: Nmap
date: 2019-06-25 17:50:24
---
# Nmap

Nmap 是一款网络扫描和主机侦测的非常有用的工具。

* 扫描端口，侦测哪些端口是否开放

  open为开放端口；closed为防火墙开放的端口，但是为使用；filtered为防火墙禁止的端口。

  ```
  nmap -sS -p 12300-12315 -v IP地址
  ```

* 扫描端口，侦测某个端口是否开放

  ```
  nmap -p 12300 IP地址
  ```
  
* 主机发现，Ping扫描

  ```
  nmap -sP 192.168.1.0/24
  ```

* 主机发现，无Ping，慢得很

  ```
  nmap -P0 192.168.1.0/24
  ```

  

*ip后的24的含义是子网掩码前24位为1，也就是255.255.255.0*

*更多内容可以参考nmap的manual*

# 参考
> https://baike.baidu.com/item/nmap/1400075?fr=aladdin