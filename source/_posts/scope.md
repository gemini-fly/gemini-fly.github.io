---
layout: post
title: JavaScript 作用域和作用域链
categories: JavaScript
tags: JavaScript
excerpt: JavaScript 作用域和作用域链学习笔记。
abbrlink: 11398
date: 2015-05-20 14:06:05
---


## JavaScript 作用域

作用域就是变量与函数的可访问范围。在JavaScript中，变量的作用域有全局作用域和局部作用域两种。

---

### 全局作用域(Global Scope)

在代码中任何地方都能访问到的对象拥有全局作用域，一般来说以下 3 种情形拥有全局作用域。

1. 最外层函数和在最外层函数外面定义的变量拥有全局作用域

```js
var outVariable = "我是最外层变量"; //最外层变量
function outFun() { //最外层函数
    var inVariable = "内层变量";
    function innerFun() { //内层函数
        console.log(inVariable);
    }
    innerFun();
}
console.log(outVariable); //我是最外层变量
outFun(); //内层变量
console.log(inVariable); //inVariable is not defined
innerFun(); //innerFun is not defined
```

2. 所有末定义直接赋值的变量自动声明为拥有全局作用域

```js
function outFun2() {
    variable = "未定义直接赋值的变量";
    var inVariable2 = "内层变量2";
}
outFun2();//要先执行这个函数，否则根本不知道里面是啥
console.log(variable); //未定义直接赋值的变量
console.log(inVariable2); //inVariable2 is not defined
```

3. 所有window对象的属性拥有全局作用域   
    window对象的内置属性都拥有全局作用域，例如 `window.name`、`window.location`、`window.top` 等。

### 局部作用域(Local Scope)

和全局作用域相反，局部作用域一般只在固定的代码片段内可访问到，最常见的例如函数内部，所有在一些地方也会看到有人把这种作用域称为函数作用域
。

如 1. 中的变量 `inVariable` 和函数 `innerFun` 都只拥有局部作用域。

---

### 再补充几个例子

* 全局作用域，函数作用域，eval作用域

```js
var a = 10; // 全局

(function() {
    var b = 20;// 函数
})();
console.log(a); // 10
console.log(b); // error, b in not defined

eval("var a = 1;"); // eval
```

* JavaScript 中没有块作用域

```js
for (var item in {
    a: 1,
    b: 2
}) {
    console.log(item);
}
console.log(item); // item still in scope
```

---

## 作用域链(Scope Chain)

在 JavaScript 中，函数也是对象，实际上，JavaScript 里一切都是对象。函数对象和其它对象一样，拥有可以通过代码访问的属性和一系列仅供 JavaScript 引擎访问的内部属性。其中一个内部属性是 [[Scope]]，由 ECMA-262 标准第三版定义，该内部属性包含了函数被创建的作用域中对象的集合，这个集合被称为函数的作用域链，它决定了哪些数据能被函数访问。

1. 在函数创建时，它的作用域链中会填入一个全局对象，该全局对象包含了所有全局变量。
2. 函数执行时会创建一个称为“运行期上下文(execution context)”的内部对象，运行期上下文定义了函数执行时的环境。每个运行期上下文都有自己的作用域链，用于标识符解析，当运行期上下文被创建时，而它的作用域链初始化为当前运行函数的[[Scope]]所包含的对象。
3. 这些值按照它们出现在函数中的顺序被复制到运行期上下文的作用域链中。它们共同组成了一个新的对象，叫“活动对象(activation object)”，该对象包含了函数的所有局部变量、命名参数、参数集合以及this，然后此对象会被推入作用域链的前端。
4. 当运行期上下文被销毁，活动对象也随之销毁。

在函数执行过程中，每遇到一个变量，都会经历一次标识符解析过程以决定从哪里获取和存储数据。该过程从作用域链头部，也就是从活动对象开始搜索，查找同名的标识符，如果找到了就使用这个标识符对应的变量，如果没找到继续搜索作用域链中的下一个对象，如果搜索完所有对象都未找到，则认为该标识符未定义。函数执行过程中，每个标识符都要经历这样的搜索过程。

---

### 例子

举一个作用域链的例子。

```js
var outVariable = "我是最外层变量"; //最外层变量
function outFun() { //最外层函数
    var inVariable = "内层变量";
    function innerFun() { //内层函数
        console.log(inVariable);
        var tempVariable = inVariable;
    }
    innerFun();
}
```

对最开始的代码稍加修改

其作用域链为：

```
window
├──outVariable
└──outFun()
   ├──inVariable
   └──innerFun()
      └──tempVariable
```

对于 `innerFun()`，其作用域链包含 3 个对象：innerFun() 自己的变量对象、outFun()的变量对象、全局变量对象。

---

#### 特例-构造器创建的函数

```js
function outer() {
    var i = 1;
    var func = new Function("console.log(typeof i);");
    func(); // undefined
}
outer();
```

通过构造器创建的函数是访问不到外层的局部变量的。

---

### 延长作用域链

有些语句可以在作用域链的前端临时增加一个变量对象，该变量对象会在代码执行后被移除。有两种情况下会发生这种现象。

* `try-catch` 语句中的 `catch` 块
* `with` 语句

对 with 来说，将会指定对象添加到作用域链中。对 catch 来说，会创建一个新的变量对象，其中包含的是被抛出的错误对象的声明。

---

### 代码优化

从作用域链的结构可以看出，在运行期上下文的作用域链中，标识符所在的位置越深，读写速度就会越慢。因为全局变量总是存在于运行期上下文作用域链的最末端，因此在标识符解析的时候，查找全局变量是最慢的。所以，在编写代码的时候应尽量少使用全局变量，尽可能使用局部变量。一个好的经验法则是：如果一个跨作用域的对象被引用了一次以上，则先把它存储到局部变量里再使用。

---

## VO & AO

### 变量初始化阶段

* JS解释器如何找到我们定义的函数和变量？

    变量对象 (Variable Object, 缩写为VO) 是一个抽象概念中的“对象”，它用于存储执行上下文中的：

    1. 变量
    2. 函数声明
    3. 函数参数

* VO按照如下顺序填充:

    1. 函数参数 (若未传入，初始化该参数值为undefined)    
    2. 函数声明 (若发生命名冲突，会覆盖)   
    3. 变量声明 (初始化变量值为 undefined，若发生命名冲突，会忽略。)

```js
function foo(x, y, z) {
    function x() {};
    console.log(x);
}
foo(100); // funtion x()

function foo2(x, y, z) {
    var x = 50;
    console.log(x);
}
foo2(110); // 50

function foo3() {
    function x() {};
    var x = 100;
    console.log(x);
}
foo3(); // 100

function foo4() {
    function x() {};
    var x;
    console.log(x);
}
foo4(); // funtion x()
```

**注：函数表达式不会影响 VO**

---

### 代码执行阶段

给 VO 对象内的变量赋值。

VO 变为 AO （活动对象 activation object）。

---

### 例

```js
alert(x); // function

var x = 10;
alert(x); // 10
x = 20;

function x() {}
alert(x); // 20

if (true) {
    var a = 1;
} else {
    var b = true;
}

alert(a); // 1
alert(b); // undefined
```

---

## 参考资料

* [鸟哥：Javascript作用域原理](http://www.laruence.com/2009/05/28/863.html)
* [理解 JavaScript 作用域和作用域链](http://www.cnblogs.com/lhb25/archive/2011/09/06/javascript-scope-chain.html)
* [JavaScript 深入浅出-慕课网](http://www.imooc.com/learn/277)
