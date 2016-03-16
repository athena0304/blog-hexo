---
title: '[月影]1-1-3 reverse string'
date: 2016-03-16 17:23:37
tags: Javascript
---
### 题目 ###
编写一个JavaScript函数，给定任意一个字符串，返回它的逆序字符串。


### 解读 ###

先用split把字符串分割成数组，再对数组reverse，再用join组成一个字符串


``` javascript
function reverse(testStr) {
    return testStr.split("").reverse().join("");
}
```

[源码](https://github.com/athena0304/training/tree/master/1-1-3-reverse%20string)

