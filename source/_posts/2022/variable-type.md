---

title: 整数类型
layout: post
abbrlink: 2493
date: 2022-03-11 16:58:40
categories: go
tags: go
---

# 1.整数类型

## 基本介绍

简单的说，就是用于存放整数值的，比如 0, -1, 2345 等等。

<!--more-->

## 1.1整数有符号类型

| 类型  | 有无符号 | 占用存储空间 |             表数范围             |
| :---: | :------: | :----------: | :------------------------------: |
| Int8  |    有    |    1字节     |             -128~127             |
| Int16 |    有    |    2字节     | -2<sup>15</sup>~2<sup>15</sup>-1 |
| Int32 |    有    |    4字节     | -2<sup>31</sup>~2<sup>31</sup>-1 |
| int64 |    有    |    8字节     | -2<sup>63</sup>~2<sup>63</sup>-1 |

### 1.1.1案例演示

```
package main

import "fmt"

func main() {
	var i int = 1
	fmt.Println("i=", i)
}
```

结果

<font color=#FF0000>i= 1

## 1.2整数无符号类型

|  类型  | 有无符号 | 占用存储空间 |      表数范围      |
| :----: | :------: | :----------: | :----------------: |
| uInt8  |    无    |    1字节     |       0~255        |
| uInt16 |    无    |    2字节     | 0~2<sup>16</sup>-1 |
| uInt32 |    无    |    4字节     | 0~2<sup>32</sup>-1 |
| uInt64 |    无    |    8字节     | 0~2<sup>64</sup>-1 |

### 1.2.1案例演示

```
package main

import "fmt"

func main() {
	var i int = 1
	fmt.Println("i=", i)
}
```

结果

<font color=#FF0000>i= 1

## 1.3整数的其它类型的说明:

| 类型 | 有无符号 |             占用存储空间             |                           表数范围                           |
| :--: | :------: | :----------------------------------: | :----------------------------------------------------------: |
| int  |    有    | 32位系统4个字节<br />64位系统8个字节 | -2<sup>31</sup> ~ 2<sup>31</sup>-1<br />-2<sup>63 </sup>~ 2<sup>63</sup>-1 |
| uInt |    无    | 32位系统4个字节<br />64位系统8个字节 |        0 ~ 2<sup>32</sup>-1<br />0 ~ 2<sup>64</sup>-1        |
| rune |    有    |             与int32一样              |              -2<sup>31</sup> ~ 2<sup>31</sup>-1              |
| byte |    无    | 与uint8一样，当要存储字符时选用byte  |                            0~255                             |

### 1.3.1案例演示

```
package main

import (
	"fmt"
)

func main() {
	var n1 int = 9000
	var n2 uint = 2
	var n3 byte = 255
	fmt.Println("n1=", n1, "n2=", n2, "n3=", n3)
}
```

结果

<font color=#FF0000>n1= 9000 n2= 2 n3= 255</font>

## 1.4整型的使用细节

1）Golang 各整数类型分：有符号和无符号，int uint 的大小和系统有关

2）Golang 的整型默认声明为 int 型

```
package main

import (
	"fmt"
)

func main() {
	var n1 = 900
	fmt.Printf("n1的类型%T", n1)
}
```

结果

<font color=#FF0000>n1的类型int</font> 

3）如何在程序查看某个变量的字节大小和数据类型（unsafe.Sizeof）

```
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var n1 int = 900
	fmt.Printf("n1的类型%T,n1占用的字节数是%d", n1, unsafe.Sizeof(n1))
}
```

结果

<font color=#FF0000>n1的类型int,n1占用的字节数是8</font> 

4）Golang 程序中整型变量在使用时，遵守保小不保大的原则，即：在保证程序正确运行下，尽量使用占用空间小的数据类型。

5）bit: 计算机中的最小存储单位。byte:计算机中基本存储单元。1byte = 8 bit
