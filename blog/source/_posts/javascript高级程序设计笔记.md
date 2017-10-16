---
title: javascript高级程序设计笔记
date: 2017-04-24 18:52:56
tags:
---

## 第四章 变量、作用域和内存问题

### 4.1 基本类型和引用类型

基本类型：Undefined, Null, Boolean, Number, String，按值访问，可以操作保存在变量中的实际值。

引用类型： 保存在内存中的对象，按引用访问。

如果从一个变量向另一个变量复制**基本类型**的值，会在变量对象上创建一个新值，然后把该值复制到为新变量分配的位置上。

``` js
var num1 = 5;
var num2 = num;
```
![4.1 复制基本类型值的过程](/images/4.1 复制基本类型值的过程.png)

当一个变量向另一个变量复制**引用类型**的值时，同样也会将存储在变量对象中的值复制一份放到为新变量分配的空间中。不同的是，这个值得副本实际上是一个指针，而这个指针指向存储在堆中的一个对象。

*用我的话理解就是，引用类型的变量本体会存在堆中，而复制的是指向堆内存的指针，两个指针都指向一个堆内存，也就是引用了同一个对象。*

指针指向对象，变量引用同一个对象

``` js
var obj1 = new Object();
var obj2 = obj1;
obj1.name = "Athena";
alert(obj2.name); //"Athena"
```
![4.2 保存在变量对象中的变量和保存在堆中的对象之间的关系](/images/4.2 保存在变量对象中的变量和保存在堆中的对象之间的关系.png)

ECMAScript中所有的参数都是按值传递的。举几个例子就一目了然了。

参数为对象的情况

``` js
function setName(obj) {
  obj.name = "Nicholas";
}

var person = new Object();
setName(person);
alert(person.name); //"Nicholas"
```
``` js
function setName(obj) {
  obj.name = "Nicholas";
  obj = new Object();
  obj.name = "Greg";
}

var person = new Object();
setName(person);
alert(person.name); //"Nicholas"
```

#### 4.1.4 检测类型

检测基本数据类型用typeof

``` js
var s = "Nicholas";
var b = true;
var i = 22;
var u;
var n = null;
var o = new Object()

alert(typeof s); //string
alert(typeof b); //boolean
alert(typeof i); //number
alert(typeof u); //undefined
alert(typeof n); //Object
alert(typeof o); //Object
```

instanceof用来检测是什么类型的对象

`result = variable instanceof constructor`

### 4.3 垃圾收集

#### 4.3.1 标记清除（主流）
javascript中最常用的垃圾收集方式。当变量进入环境时，就将这个变量标记为“进入环境”。从逻辑上讲，永远不能释放进入环境的变量所占用的内存，因为只要执行流进入相应的环境，就可能会用到它们。而当变量离开环境时，则将其标记为“离开环境”

思想是给当前不使用的值加上标记，然后再回收其内存
#### 4.3.2 引用计数（罕见 IE）


## 第五章 引用类型

### 5.1 Object类型
创建有两种方式：

1) 使用new操作符后跟Object构造函数

``` js
var person = new Object();
person.name = "athena";
person.age = 26;
```

2) 使用对象字面量表示法

``` js
var person = {
  name: "athena",
  age: 26
}
```
也可以
``` js
var person = {};
person.name = "athena";
person.age = 26;
```

然后再说花括号的问题。在js中，有表达式上下文（expression context）和语句上下文（statement context）。表达式上下文指的是能够返回一个值（表达式），语句上下文例如跟在if语句条件后面，表达一个语句块的开始。在这里花括号处于表达式上下文中，则表示对象字面量的开始。

在封装函数时，对象字面量可以用来传递大量可选参数，而对那些必需值使用命名参数

``` js
function displayInfo (args) {
  if(typeof args.age == "number") {
    output = args.age;
  }
}
displayInfo({
  age: 29 //这个参数可选
})
```

### 5.2 Array类型

两种创建方式

1) 使用Array构造函数
``` js
var colors = new Array();
var colors = Array(30) //这个方法不能用forEach 可以用原生的for循环
var colors = Array("red", "blue", "green")
```

2) 使用数组字面量表示法

``` js
var colors = [];
var colors = ["red", "blue", "green"]
```

#### 5.2.1 检测数组

对于一个网页，或者一个全局作用域而言，只有一个全局执行环境，使用`instanceof`操作符就可以：

``` js
if(value instanceof Array) {
  //对数组执行某些操作
}
```
如果网页中包含多个框架，那实际上就存在两个以上不同的全局执行环境，从而存在两个以上不同版本的Array构造函数。如果你从一个框架向另一个框架传入一个数组，那么传入的数组与在第二个框架中原生创建的数组分别具有各自不同的狗在函数。

所以ECMAScript5新增了`Array.isArray()`方法

``` js
if(Array.isArray(value)) {
  //对数组执行某些操作
}
```

### 5.5 Function类型

*函数是对象，函数名是指针*

使用不带圆括号的函数名是访问函数指针，而非调用函数

#### 函数声明提升
``` js
alert(sum(10, 10));
function sum(num1, num2) {
  return num1 + num2;
}
```
