---
title: go-变量
layout: post
abbrlink: 29354
date: 2022-03-10 10:47:14
categories: go
tags: go
password: 111
abstract: 有东西被加密了，请输入密码查看
message: 请输入密码
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# 前言
开始系统性的学习go语言，以此记录
<!--more-->
# 变量

## 1. 变量的介绍

### 1.1 变量的概念

量相当于内存中一个数据存储空间的表示，你可以把变量看做是一个房间的门 牌号，通过门牌号我们可以找到房间，同样的道理，通过变量名可以访问到变量 (值)。

### 1.2 2变量的使用步骤

1）声明变量(也叫:定义变量) 

2）非变量赋值 

3）使用变量

### 1.3 变量快速入门案例

```
package main

import "fmt"

func main() {
	//声明变量
	var n1 int
	//变量赋值
	n1 = 100
	//使用变量
	fmt.Println("n1=", n1)
}
```

结果

n1= 100

### 1.4 Golang 变量使用的三种方式

(1) 第一种：指定变量类型，声明后若不赋值，使用默认值

```
package main

import "fmt"

func main() {
	//指定变量类型，声明后若不赋值，使用默认值，int的默认值为0
	var n1 int
	fmt.Println("n1=", n1)
}
```

结果

n1= 0

(2) 第二种：根据值自行判定变量类型(类型推导)

```
package main

import "fmt"

func main() {
	//不指定变量类型，根据值自定判断变量类型
	var n1 = 100
	fmt.Println("n1=", n1)
}
```

结果

n1= 100

(3) 第三种：省略 var, 注意 :=左侧的变量不应该是已经声明过的，否则会导致编译错误

```
package main

import "fmt"

func main() {
	//省略var，注意 := 左侧的变量不应该是已经声明过的，否则会导致编译失败
	//:= 不能省略
	n1 := "name"
	fmt.Println("n1=", n1)
}
```

结果

n1= name

### 1.5 多变量声明

该例子演示golang如何一次声明多个变量

```go
package main

import "fmt"

func main() {
    var n1, n2, n3 int
    fmt.Println("n1=",n1,"n2=",n2,"n3=",n3)
}
```

结果
n1= 0 n2= 0 n3= 0

### 1.6 该例子演示golang如何一次声明多个变量2

```go
package main

import "fmt"

func main() {
    var n1, n2, n3 = 100,"tom",200
    fmt.Println("n1=",n1,"n2=",n2,"n3=",n3)
}
```

结果

n1= 100 n2= tom n3= 200

### 1.7 该例子演示golang如何一次声明多个变量2

```
package main

import "fmt"

func main() {
    n1, n2, n3 := 100,"tom2",200
    fmt.Println("n1=",n1,"n2=",n2,"n3=",n3)
}
```

结果

n1= 100 n2= tom2 n3= 200

### 1.8 如何一次性申请多个全局变量

```
package main

import "fmt"

var (
	n1 = 200
	n3 = 600
	n4 = 800
)

func main() {
	n1, n2, n3 := 100, "tom2", 200
	fmt.Println("n1=", n1, "n2=", n2, "n3=", n3, "n4=", n4)
}
```

结果

n1= 100 n2= tom2 n3= 200 n4= 800

由此可以看出，在函数内局部变量>全局变量，函数内没有的变量使用全局变量

