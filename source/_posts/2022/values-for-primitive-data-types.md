---
layout: post
title: 基本数据类型的默认值
tags: go
abbrlink: 4118
date: 2022-04-07 14:16:55
---

# 1.基本介绍

在 go 中，数据类型都有一个默认值，当程序员没有赋值时，就会保留默认值，在 go 中，默认值 又叫零值。

<!--more-->

# 2. 基本数据类型的默认值如下

| 数据类型 | 默认值 |
| :------: | :----: |
|   整型   |   0    |
|  浮点型  |   0    |
|  字符串  |   ""   |
|  布尔型  | False  |

## 2.1 案例

```
package main

import "fmt"

func main() {
	var a int
	var b float32
	var c float64
	var d bool
	var f string
	fmt.Printf("a=%d,b=%v,c=%v,d=%v,f=%v", a, b, c, d, f)
}

```

结果

<font color=#ff0000>a=0,b=0,c=0,d=false,f=</font>

