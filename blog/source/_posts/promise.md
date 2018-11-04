---
title: '[译]Promise-basics'
date: 2018-05-23 16:33:58
tags: 
- Promise
---

原文：[Promise](https://javascript.info/promise-basics)

想象一下，你是一个顶级歌手，你的粉丝们成天管要你的下一张专辑。
为了得到解脱，你向他们承诺，一旦专辑发行了就会立即寄给他们。你给粉丝们一张表让他们填写地址，这样一来一旦歌曲发行了，所有预定的人都能马上拿到。如果发生了什么意外，比如歌曲再也不会发行了，他们也会收到通知。

这样就皆大欢喜了，再也不会有人来烦你了，粉丝们也不会错过你的专辑单曲。

以上是一个真实世界的类比，我们在编程的过程中经常会遇到类似的情景：

1. “生产代码”会做一些事情，会需要一些时间。例如，加载异步脚本。这就相当于那个“歌手”。
2. “消费代码”则是等准备好了之后获取结果。很多函数都可能需要这个结果。这就相当于那些“粉丝”。
3. *promise* 就是一个特殊的JavaScript对象，它把它们连接起来。这就相当于那个“列表”。生产代码做出来之后发给每个人，这样他们就都能订阅这个结果了。

虽然这个类比并不是非常的准确，因为JavaScript promise比一个简单的列表复杂得多，它们有额外的特性和限制，但是这个比喻还是可以帮助我们理解的。

一个promise对象的构建语法如下：

``` js
let promise = new Promise(function(resolve, reject) {
  // 执行者 (生产代码, "歌手")
});
```

传给`new Promise`的函数叫做*执行者*。当promise被创建之后，*执行者*就被立即调用。它包括了生产代码，也就是最终要通过得到一个结果来结束。基于上面的类比，这个执行者就是一个“歌手”。

生成的`promise`对象有内部属性：

- `state` -- 初始值是“pending”，之后会变成“fulfilled”或者“rejected”
- `result` -- 任意值，初始值是`undefined`

当执行者结束这个任务后，就会调用下面两个函数的一个：

- `resolve(value)` -- 表明任务成功结束:
  - 将 `state` 置成 `"fulfilled"`,
  - 将 `result` 置成 `value`.
- `reject(error)` -- 表明发生了错误:
  - 将 `state` 置成 `"rejected"`,
  - 将 `result` 置成 `error`.

{% asset_img promise-resolve-reject.png %}

下面是一个简单的执行者的例子，

```js
let promise = new Promise(function(resolve, reject) {
  // 当promise被构造的时候，这个函数就被立即执行

  alert(resolve); // function () { [native code] }
  alert(reject);  // function () { [native code] }

  // 1秒之后这个任务完成，输出结果"done!"
  setTimeout(() => resolve("done!"), 1000);
});
```

通过运行上面的代码，我们可以看出两件事情：

1. 这个执行者被立即自动调用了（通过 `new Promise`）
2. 执行者接收两个参数：`resolve` 和 `reject` -- 这些函数来自JavaScript引擎。我们不需要去创建它们。而且该由执行者来调用它们。

1秒之后，执行者调用`resolve("done")`生成结果：

{% asset_img promise-resolve-1.png %}

这是任务成功完成的一个例子。

现在再看一个执行者抛出异常的例子：

``` js
let promise = new Promise(function(resolve, reject) {
  // 1秒之后任务结束，抛出异常
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});
```

{% asset_img promise-reject-1 %}

总结一下就是，执行者会执行一个任务（通常是需要花费一定时间的），然后调用 `resolve` 或者 `reject`来改变相应的promise对象的状态。

不管是`resolve` 还是 `reject`都会被叫做 "settled"，与 "pending"相对应。

#### 只能有一个结果或者异常

执行者只能调用一个`resolve` 或者 `reject`。promise的状态改变就是最终形态。
后面的任何调用都是无效的：

```js
let promise = new Promise(function(resolve, reject) {
  resolve("done");

  reject(new Error("…")); // 无效
  setTimeout(() => resolve("…")); // 无效
});
```

至于一个任务执行完成只能有一个结果或者异常这个事情来说，在编程领域，还存在其它数据结构允许很多“流”结果，例如数据流（streams）和队列（queues）。相对于promise，它们也是各有利弊。Javascript核心不支持它们，而且缺少一些只有promise能提供的语言特性，这里我们不做过多的涉及。

还有就是如果我们调用`resolve/reject`时传入了多个参数，只有第一个参数有用，后面的都会被忽略的。

#### Reject 使用 `Error` 对象

通常来说我们可以像调用`resolve`一样传入任何参数来调用`reject`。但还是推荐使用`Error` 对象。原因的话后面会提。

#### Resolve/reject 可以是立即执行的

在练习中，执行者通常是做一些异步操作，在一段时间之后才调用`resolve/reject`，但其实并不一定是这样。我们可以立即调用`resolve` or `reject`函数，例子如下：

```js
let promise = new Promise(function(resolve, reject) {
  resolve(123); // 立即给出结果：123
});
```

一般这种情况发生在，当我们开始要执行一个任务的时候，来看一下是不是多有的事情都准备好了。然后得到一个resolvedpromise表示可以做下面的事情了。

### `state` 和 `result` 是内部变量

`state` 和 `result`属性是一个promise对象的内部变量。我们不能在代码中直接访问它们，但是可以使用`.then/catch`方法，下面会说。

## 消费者: ".then" 和 ".catch"

promise对象作为生产代码（执行者）和消费函数（那些想要接收结果或者错误的函数）之间的连接桥梁，消费函数可以使用方法`promise.then` and `promise.catch`来注册。

`.then`的语法如下：

```js
promise.then(
  function(result) { /* 处理成功的 result */ },
  function(error) { /* 处理 error */ }
);
```

如果promise执行成功，得到result，就会执行第一个参数函数，如果promise返回错误就会执行第二个参数函数

例如：

```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("done!"), 1000);
});

// resolve runs the first function in .then
promise.then(
  result => alert(result), // 1秒之后显示 "done!"
  error => alert(error) // 不会执行
);
```

reject的例子:

```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

// reject runs the second function in .then
promise.then(
  result => alert(result), // 不会执行
  error => alert(error) // 1秒之后显示 "done!" "Error: Whoops!"
);
```

如果我们只需要成功的情况，那么也可以只提供一个参数：

```js
let promise = new Promise(resolve => {
  setTimeout(() => resolve("done!"), 1000);
});

promise.then(alert); // 1秒之后显示 "done!"
```

如果只对errors感兴趣，那么可以使用`.then(null, function)`或者`.catch(function)`

```js
let promise = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

// .catch(f) 与 promise.then(null, f)是一样的
promise.catch(alert); // 1秒之后显示 "done!" "Error: Whoops!"

```

`.catch(f)` 与 `.then(null, f)`是完全等同的。

### settled的 promise会立即执行 `then`

如果一个promise在pending，`.then/catch`处理器会等待result。另外，如果一个promise状态已经是settled， 它们会立即执行。

```js
// 一个立即 resolved 的 promise
let promise = new Promise(resolve => resolve("done!"));

promise.then(alert); // done! (立即显示出来)
```

对于那些有时需要时间，有时立即完成的任务，这样处理是非常方便的，并且能够保证两种情况都能正常运行。

### `.then/catch`的处理器永远是异步的

更严格来说，当`.then/catch`处理器要执行的时候，首先会加入一个内部序列。当当前的代码执行完毕之后，JavaScript引擎从序列中取出这些处理器并执行，和`setTimeout(..., 0)`类似。

换句话说，当`.then(handler)`将要触发的时候，它的行为类似于`setTimeout(handler, 0)`

在下面的例子中，promise立即resolved，所以`.then(alert)`也会马上触发：`alert`回调加入序列，然后等到代码执行结束后，立即执行。

```js
// 一个立即resolved的promise
let promise = new Promise(resolve => resolve("done!"));

promise.then(alert); // done! (在当前代码运行结束之后)

alert("code finished"); // 这个alert会先显示出来
```

所以在`.then`后面的代码通常是先于then执行的，即使是在立即resolve的promise情况下。通常这不是很重要，只有一些情况可能会有用。

下面我们来看一些更多的例子，promise如何帮助我们编写异步代码

## 例子: loadScript

比如下面的`loadScript`函数，用的是callback回调函数的写法，用来加载一个脚本：

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error ` + src));

  document.head.append(script);
}
```

让我们用promise重写一下

`loadScript`新方法不需要回调函数作为参数了。当加载文件完成后，会创建并返回一个promise对象。外面的代码可以用`.then`来添加后续处理的方法。

```js
function loadScript(src) {
  return new Promise(function(resolve, reject) {
    let script = document.createElement('script');
    script.src = src;

    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error("Script load error: " + src));

    document.head.append(script);
  });
}
```

使用:

```js
let promise = loadScript("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/3.2.0/lodash.js");

promise.then(
  script => alert(`${script.src} is loaded!`),
  error => alert(`Error: ${error.message}`)
);

promise.then(script => alert('One more handler to do something else!'));
```

通过与这种基于回调函数语法的对比，我们可以立即找出一些优点：

callback：

- 我们必须提前准备好`callback`函数，也就是说我们必须在`loadScript`调用之前就要处理结果。
- 只能有一个callback。

Promise：

- Promise可以让我们用一种非常自然的顺序来编写代码。首先我们运行`loadScript`， 然后再继续写处理结果的逻辑。
- 我们可以事后多次调用`.then`

所以promise给我们带来了更好的代码流和灵活性。但是不只是这些，我们下章再继续。
