---
title: 你不知道的js笔记-this和原型
date: 2017-04-11 11:06:01
tags:
---

## this的调用

每个函数的this是在调用时被绑定的，完全取决于函数的调用位置，也就是函数的调用方法。
那么就要分析调用栈，可以用浏览器的调试工具查看调用栈。打断点后，显示的调用栈的第二个元素，就是真正的调用位置。

### 绑定规则

#### 1. 默认绑定

``` js
funciton foo(){
  console.log(this.a)
}
var a = 2;
foo(); //2
```

如果使用严格模式，则不能将全局对象用于默认绑定，因此this会绑定到undefined

#### 2. 隐式绑定

在一个对象内部包含一个指向函数的属性，并通过这个属性间接引用函数，从而把this间接（隐式）绑定到这个对象上。
当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。

``` js
funciton foo() {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
};

obj.foo(); //2
```

#### 3. 显式绑定

call(...)和apply(...)
它们的第一个参数是一个对象，是给this准备的，接着在调用函数时将其绑定到this。

``` js
funciton foo() {
  console.log(this.a)
}
var obj = {
  a:2
};
foo.call(obj); //2
```

如果传入了一个原始值，也就是字符串类型、布尔类型或者数字类型来当做this的绑定对象，这个原始值会被转换成它的对象形式，也就是new String(...), new Boolean(...), new Number(...)

##### 3.1 硬绑定

ES5提供了内置的方法Function.prototype.bind，它的用法如下：

``` js
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}
var obj = {
  a:2
};

var bar = foo.bind(obj);

var b = bar (3); //2 3
console.log(b); //5
```

bind会返回一个硬解码的新函数，它会把你指定的参数设置为this的上下文并调用原始函数。

##### 3.2 API调用的上下文

``` js
fonction foo(el) {
  console.log(el, this.id);
}

var obj = {
  id: "awesome"
}

//调用foo(...)时把this绑定到obj
[1, 2, 3].forEach(foo, obj)
```

#### 4. new绑定
