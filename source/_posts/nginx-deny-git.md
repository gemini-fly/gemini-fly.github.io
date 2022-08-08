---
title: nginx禁止外部访问到隐藏文件（.git或.env）
layout: post
abbrlink: 61888
date: 2021-06-03 11:21:29
categories: nginx
tags: nginx
---

# 前言

安全部门说线上有域名可以直接下载到.env文件，这类文件中是有数据库等账号信息，所以需要隐藏掉

<!--more-->

# 解决方法

在server的配置中，加上以下这段，重新reload即可

```nginx
    location ~ /\. {
            deny all;
    }
```
