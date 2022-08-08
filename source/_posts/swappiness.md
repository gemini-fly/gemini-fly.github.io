---
title: Linux下设置swappiness参数来配置内存使用到多少才开始使用swap分区
layout: post
abbrlink: 60984
date: 2021-05-20 14:52:32
categories: linux
tags: linux
---

# 前言

阿里云 ECS的 CPU 使用率有时候到100%，top 命令查看进程发现是 kswapd0 进程占 cpu 达到70%。

<!--more-->

swap分区的作用是当物理内存不足时，会将一部分硬盘当做虚拟内存来使用。

kswapd0 占用过高是因为 物理内存不足，使用swap分区与内存换页操作交换数据，导致CPU占用过高。
swap分区的介绍可以看这个文章：[swap分区](https://running-dpf.github.io/2021/05/20/swap/)
# 解释

swappiness的值的大小对如何使用swap分区是有着很大的联系的。swappiness=0的时候表示最大限度使用物理内存，然后才是swap空间，swappiness＝100的时候表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。linux的基本默认设置为60，具体如下：

```
cat /proc/sys/vm/swappiness
#60
```

也就是说，你的内存在使用到 100-60=40% 的时候，就开始出现有交换分区的使用。大家知道，内存的速度会比磁盘快很多，这样子会加大系统IO，同时造的成大量页的换进换出，严重影响系统的性能，所以我们在操作系统层面，要尽可能使用内存，对该参数进行调整。



# 解决方法

临时调整的方法如下，我们调成10：

```
sysctl vm.swappiness=10
#vm.swappiness=10
cat /proc/sys/vm/swappiness
#10
```

这只是临时调整的方法，重启后会回到默认设置的.

要想永久调整的话，需要在/etc/sysctl.conf修改，加上：

```
sudo vim /etc/sysctl.conf
```

加上

```
# Controls the maximum number of shared memory segments, in pages
kernel.shmall = 4294967296 #这一个可以不用设置
vm.swappiness = 10
```

 生效

```
sudo sysctl -p
```

这样便完成修改设置！
