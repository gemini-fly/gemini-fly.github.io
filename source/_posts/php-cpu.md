---
title: php进程不及时释放进程导致CPU跑满
layout: post
abbrlink: 45349
date: 2021-06-01 18:08:43
categories: php
tags: php
---
# 前言
公司一台服务器安装了php，很稳定的跑了3年，突然有一天CPU报警，排查为php-fpm进程CPU利用率为100%，遂排查原因
<!--more-->
# 理解
php-fpm进程不但占用内存，也会占用cpu。实际服务器运行观察中，我们可能经常发现这种现象，内存占用很少，cpu却跑满了，导致运行堵塞。当然，出现这种情况的原因可能有多种，一种原因就是php进程没有得到及时释放。

对于这种情况，可以设置最大请求数max_requests，即当一个 PHP-CGI 进程处理的请求数累积到 max_requests 个后，则自动重启该进程，这样达到了释放内存的目的了，当然cpu占用也同步降低了。

以4G内存服务器为例进行设置，加入FPM配置文件：
pm.max_requests = 128
pm.max_children = 20 #表示 php-fpm 能启动的子进程的最大数量
当然，以上数值应该根据需要测试调整。

以上参数可以参考这个文章 [php-fpm的配置文件详解](http://localhost:4000/2021/06/01/php-fpm-conf/)

