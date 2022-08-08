---
title: du
layout: post
categories: linux
tags: linux
abbrlink: 10219
date: 2021-09-18 16:44:44
---
# du
du(Disk Usage) - 报告磁盘空间使用情况
<!--more-->
```
-a, --all
    显示对所有文件的统计，而不只是包含子目录。
-b, --bytes
    输出以字节为单位的大小，替代缺省时1024字节的计数单位。
--block-size=size
    输出以块为单位的大小，块的大小为 size 字节。( file- utils-4.0 的新选项)
-c, --total
    在处理完所有参数后给出所有这些参数的总计。这个选项被 用给出指定的一组文件或目录使用的空间的总和。
-D, --dereference-args
    引用命令行参数的符号连接。但不影响其他的符号连接。 这对找出象 /usr/tmp 这样的目录的磁盘使用量有用， /usr/tmp 等通常是符号连接。 译住：例如在 /var/tmp 下建立一个目录test, 而/usr/tmp 是指向 /var/tmp 的符号连接。du /usr/tmp 返回一项 /usr/tmp , 而 du - D /usr/tmp 返回两项 /usr/tmp，/usr/tmp/test。
--exclude=pattern
    在递归时，忽略与指定模式相匹配的文件或子目录。模式 可以是任何 Bourne shell 的文件 glob 模式。( file- utils-4.0 的新选项)
-h, --human-readable
    为每个数附加一个表示大小单位的字母，象用M表示二进制 的兆字节。
-H, --si
    与 -h 参数起同样的作用，只是使用法定的 SI 单位( 用 1000的幂而不是 1024 的幂，这样 M 代表的就是1000000 而不是 1048576)。(fileutils-4.0 的新选项)
-k, --kilobytes
    输出以1024字节为计数单位的大小。
-l, --count-links
    统计所有文件的大小，包括已经被统计过的(作为一个硬连接)。
-L, --dereference
    引用符号连接(不是显示连接点本身而是连接指向的文件或 目录所使用的磁盘空间)。
-m, --megabytes
    输出以兆字节的块为计数单位的大小(就是 1,048,576 字节)。
--max-depth=n
    只输出命令行参数的小于等于第 n 层的目录的总计。 --max-depth=0的作用同于-s选项。(fileutils-4.0的新选项)
-s, --summarize
    对每个参数只显示总和。
-S, --separate-dirs
    单独报告每一个目录的大小，不包括子目录的大小。
-x, --one-file-system
    忽略与被处理的参数不在同一个文件系统的目录。
-X file, --exclude-from=file
    除了从指定的文件中得到模式之外与 --exclude 一样。 模式以行的形式列出。如果指定的文件是'-',那么从标准输 入中读出模式。(fileutils-4.0 的新选项) GNU 标准选项
--help
    在标准输出上输出帮助信息后正常退出。
--version
    在标准输出上输出版本信息后正常退出。
```

### 常用命令
* 统计当前目录下，第一层的文件大小

```
du -ah --max-depth=1 ./cdn
```
* 统计当前目录大小，并安大小排序

```
du -sm * | sort -n
```
* 按大小排序目录(查看文件目录大小)

```
du -h --time --max-depth=1 | sort -hr
```
* 列出当前目录中的目录名不包括xyz字符串的目录的大小

```
du -h --exclude='*xyz*'
```
* 显示几个文件或目录各自占用磁盘空间的大小，并统计它们的总和

```
du -c log30.tar.gz log31.tar.gz
```
* 查看各文件夹大小命令

```
du -h --max-depth=1
```
