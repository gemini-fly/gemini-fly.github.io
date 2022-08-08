---
title: NGINX禁止解析IP访问
layout: post
abbrlink: 6321
date: 2021-05-20 13:56:14
categories: nginx
tags: nginx
---
nginx配置禁止ip访问，只能域名访问，这样可以增加一定的安全性，并且在nginx内配置80跳转443，或者在公有云负载均衡配置。
<!--more-->


# 配置如下

```
server{ 
listen 80 default; 
server_name _; 
return 404; 
}

server {
listen 80;
server_name xxx;
rewrite ^(.*)$ https://${server_name}$1 permanent;
}

server {
listen 443;
server_name xxx;
    #ssl_certificate ;
    #ssl_certificate_key ;

}
```
