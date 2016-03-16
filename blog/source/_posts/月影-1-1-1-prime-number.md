---
title: '[月影]1-1-1 prime number'
date: 2016-03-16 16:27:53
tags: Javacscipt
---
### 题目 ###
编写一个JavaScript函数，判断一个整数是否是素数，如果是，返回true，否则，返回false。


### 解读 ###

除了1和它本身以外不再有其他的因数。根据算术基本定理，每一个比1大的整数，要么本身是一个质数，要么可以写成一系列质数的乘积，最小的质数是2。


### 方法 ###
在一般领域，对正整数n，如果用2到根号n之间的所有整数去除，均无法整除，则n为质数。
质数大于等于2 不能被它本身和1以外的数整除。

``` javascript
function IsPrime(testNum) {
    if (testNum < 2) return false;
    var len = Math.sqrt(testNum);
    for (var i = 2; i <= len; i++) {
        if (testNum % i === 0) {
            return false;
        }
    }
    return true;
}
```

[源码](https://github.com/athena0304/training/tree/master/1-1-1-prime%20number)
