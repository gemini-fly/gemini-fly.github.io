---
title: wireshark合并多个pacp文件
layout: post
abbrlink: 45387
date: 2021-05-27 10:21:10
categories: wireshark 网络
tags: wireshark 
---

## 前言

当出现网络问题的时候，可能会抓很多个pcap包，当分析的时候时间点很不容易查找，所以要将多个pcap包合并为一个，方便我们运维人员查找

<!--more-->

## 命令

macos命令

cd /Applications/Wireshark.app/Contents/MacOS

./mergecap -w ~/Downloads/compare.pcap ~/Downloads/*.pcap

这样，我们在Downloads文件夹能看到一个compare.pcap的文件，这就是整合的pacp文件
