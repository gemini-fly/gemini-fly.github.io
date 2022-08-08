---
title: 布尔类型
layout: post
categories: go
tags: go
abbrlink: 18267
date: 2022-03-18 15:16:21
---

# 1.布尔类型

## 基本介绍

1、布尔类型也叫 bool 类型，bool 类型数据只允许取值 true 和 false

2、bool 类型占1个字节

3、bool 类型适于逻辑运算，一般用于程序流程控制

## 1.1案例演示

```
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var n1 = false
	fmt.Println("n1=", n1)
	fmt.Println("n1占用空间：", unsafe.Sizeof(n1))
}

```

结果

<font color=#FF000>n1= false</font>

<font color=#FF0000>n1占用空间： 1</font>



