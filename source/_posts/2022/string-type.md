---
title: string类型
layout: post
categories: go
tags: go
abbrlink: 33404
date: 2022-03-18 15:33:13
---

# 1.字符串类型

## 基本介绍

字符串就是一串固定长度的字符连接起来的字符序列。Go 的字符串是由单个字节连接起来的。Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本

## 1.1 案例演示

```
package main

import (
	"fmt"
)

func main() {
	var ssr string = "哈哈哈哈，hello world"
	fmt.Println(ssr)
}
```

结果

<font color=#FF000>哈哈哈哈，hello world</font>

## 1.2 string 使用注意事项和细

### 1.2.1 Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本，这样 Golang 统一使用 UTF-8 编码,中文 乱码问题不会再困扰程序员

### 1.2.2 字符串一旦赋值了，字符串就不能修改了：在 Go 中字符串是不可变的

```
package main

import (
	"fmt"
)

func main() {
	var ssr string = "哈哈哈哈，hello world"
	fmt.Println(ssr[0])
}
```

结果

<font color=#FF0000>229</font>

```
package main

import (
	"fmt"
)

func main() {
	var ssr string = "哈哈哈哈，hello world"
	ssr[0] = 'a'
	fmt.Println(ssr[0])
}
```

结果

<font color=#ff0000>cannot assign to ssr[0]</font>，标识无法重新给ssr的第0个元素进行修改

### 1.2.3 字符串的两种表示形式 

(1) 双引号, 会识别转义字符 

(2) 反引号，以字符串的原生形式输出，包括换行和特殊字符，可以实现防止攻击、输出源代码等效果

```
package main

import "fmt"

func main() {
	var ssr string = `
	package main

import (
	"fmt"
)

func main() {
	var ssr string = "哈哈哈哈，hello world"
	ssr[0] = 'a'
	fmt.Println(ssr[0])
}

	`
	fmt.Println(ssr)
}
```

结果

<font color=#ff0000>package main</font>

<font color=#ff0000>import (</font>

<font color=#ff0000>    "fmt"</font>

<font color=#ff0000>)</font>

<font color=#ff0000>func main() {</font>

<font color=#ff0000>    var ssr string = "哈哈哈哈，hello world"</font>

<font color=#ff0000>    ssr[0] = 'a'</font>

<font color=#ff0000>    fmt.Println(ssr[0])</font>

<font color=#ff0000>}</font>

### 1.2.4 字符串的拼接方式

```
package main

import "fmt"

func main() {
	var nayaur = "hello" + "world"
	nayaur += "nayaur!"
	fmt.Println(nayaur)
}
```

结果

<font color=#ff0000>helloworldnayaur!</font>

发现其实是没有空格的，这样可以在每个字符串中间加空格，拼接的字符串就会有空格

```
package main

import "fmt"

func main() {
	var nayaur = "hello" + " world"
	nayaur += " nayaur!"
	fmt.Println(nayaur)
}
```

结果

<font color=#ff0000>hello world nayaur!</font>

### 1.2.5 当一行字符串太长时，需要使用到多行字符串，可以如下处理

```
package main

import "fmt"

func main() {
	var nayaur = "hello" + " world" + 
	"hello" + " world"
	fmt.Println(nayaur)
}
```

结果

<font color=#ff0000>hello worldhello world</font>

