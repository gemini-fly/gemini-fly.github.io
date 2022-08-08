---
title: php-fpm配置文件详解
layout: post
abbrlink: 2439
date: 2021-06-01 18:19:46
categories: php
tags: php
---
# 前言

php一直都搞不懂，今天彻底把php-fpm的配置文件解释一下
<!--more-->

# 配置

```
[www]
user = www
group = www

;listen = 127.0.0.1:9000
listen.owner = www
listen.group = www
listen.mode = 0660
listen = /dev/shm/php-fpm.sock
listen.allowed_clients = 127.0.0.1

;pm = static
pm = dynamic

pm.max_children = 40
pm.start_servers = 20
pm.min_spare_servers = 20
pm.max_spare_servers = 40
pm.max_requests = 128
;pm.max_requests = 500
```

# 详解

1、pm = dynamic 或 pm = static 表示使用哪种进程数量管理方式，dynamic表示php-fpm进程数是动态的，最开始是pm.start_servers指定的数量，如果请求较多，则会自动增加，保证空闲的进程数不小于pm.min_spare_servers，如果进程数较多，也会进行相应清理，保证多余的进程数不多于pm.max_spare_servers；

2、pm.max_children：静态方式下开启的php-fpm进程数量，在动态方式下他限定php-fpm的最大进程数(<font color='red'>这里要注意pm.max_spare_servers的值只能小于等于pm.max_children</font>）
pm.start_servers：动态方式下的起始php-fpm进程数量。
pm.min_spare_servers：动态方式空闲状态下的最小php-fpm进程数量。
pm.max_spare_servers：动态方式空闲状态下的最大php-fpm进程数量。

区别：

如果dm设置为static，那么其实只有pm.max_children这个参数生效。系统会开启设置数量的php-fpm进程。

如果dm设置为dynamic，那么pm.max_children参数失效，后面3个参数生效。系统会在php-fpm运行开始的时候启动pm.start_servers个php-fpm进程，然后根据系统的需求动态在pm.min_spare_servers和pm.max_spare_servers之间调整php-fpm进程数。

设置：

数值设置，参考自己的实际硬件配置，可以参考 总内存/30M 来计算。

# 计算



如何判断我选择“pm = dynamic”还是“pm = static”呢？哪一种更好呢？

事实上，跟Apache一样，运行的PHP程序在执行完成后，或多或少会有内存泄露的问题。

这也是为什么开始的时候一个php-fpm进程只占用3M左右内存，运行一段时间后就会上升到20-30M的原因了。

 

对于内存大的服务器（比如8G以上）来说，用静态的max_children实际上更为妥当，因为这样不需要进行额外的进程数目控制，会提高效率。因为频繁开关php-fpm进程也会有时滞，所以内存够大的情况下开静态效果会更好。数量也可以根据 总内存/30M 得到，比如8GB内存可以设置为100，那么php-fpm耗费的内存就能控制在 2G-3G的样子。

如果内存稍微小点，比如1~2G，那么指定静态的进程数量更加有利于服务器的稳定。这样可以保证php-fpm只获取够用的内存，将不多的内存分配给其他应用去使用，会使系统的运行更加畅通。

对于小内存的服务器来说，比如256M内存的VPS，即使按照一个20M的内存量来算，10个php-cgi进程就将耗掉200M内存，那系统的崩溃就应该很正常了。

因此应该尽量地控制php-fpm进程的数量，大体明确其他应用占用的内存后，给它指定一个静态的小数量，会让系统更加平稳一些。

或者使用动态方式，因为动态方式会结束掉多余的进程，可以回收释放一些内存，所以推荐在内存较少的服务器或VPS上使用，具体最大数量根据 总内存/20M 得到。

比如说512M的VPS，建议pm.max_spare_servers设置为20。至于pm.min_spare_servers，则建议根据服务器的负载情况来设置，比较合适的值在5~10之间。

# 总结

总结：内存小的建议用动态（pm = dynamic），内存大的建议用静态（pm = static）

