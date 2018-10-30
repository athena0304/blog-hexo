---
title: addEventListener
date: 2018-10-29 21:21:31
tags:
---
今天我在看一篇讲MVC和观察者模式的文章的时候，里面有一个例子代码，是这样的：
``` js
export class Controller {
  constructor(model) {
    this.model = model;
  }
  //EVENTLISTENER INTERFACE
  handleEvent(e) {
    e.stopPropagation();
    switch (e.type) {
      case "click":
        this.clickHandler(e.target);
        break;
      default:
        console.log(e.target);
    }
  }
  //GET MODEL HEADING
  getModelHeading() {
    return this.model.heading;
  }
  //CHANGE THE MODEL
  clickHandler(target) {
    this.model.heading = "World";
    this.model.notifyAll();
  }
}
```
中间实例化了这个class，为controller
DOM事件监听的调用：
``` js
this.heading.addEventListener("click", controller);
```

可以发现，这里的addEventListener传的第二个参数是一个对象，那么这个对象怎么就能解析事件呢，以前的我只知道第二个参数应该传一个函数，而从没使用过对象的这种方式。于是查了一下MDN，果然不简单。

## EventTarget

要说 addEventListener，首先要从 [EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget) 说起。

> EventTarget 是一个由对象实现的接口，这些对象可以接收事件，并为它们提供侦听器。
>
> [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element)、 [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Document)和 [`window`](https://developer.mozilla.org/en-US/docs/Web/API/Window) 是最常见的 event targets, 但其它对象也可成为 event targets， 例如 [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) [`AudioNode`](https://developer.mozilla.org/en-US/docs/Web/API/AudioNode) [`AudioContext`](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext) 等。

> Web平台提供了几种获取DOM事件通知的方法。两种常见的方式是：通用 `addEventListener()` 和一组特定的 **on-event** 处理程序。

然后 MDN 给出了一个 EventTarget 的简单实现：

```js
var EventTarget = function() {
  this.listeners = {};
};

EventTarget.prototype.listeners = null;
EventTarget.prototype.addEventListener = function(type, callback) {
  if (!(type in this.listeners)) {
    this.listeners[type] = [];
  }
  this.listeners[type].push(callback);
};

EventTarget.prototype.removeEventListener = function(type, callback) {
  if (!(type in this.listeners)) {
    return;
  }
  var stack = this.listeners[type];
  for (var i = 0, l = stack.length; i < l; i++) {
    if (stack[i] === callback){
      stack.splice(i, 1);
      return;
    }
  }
};

EventTarget.prototype.dispatchEvent = function(event) {
  if (!(event.type in this.listeners)) {
    return true;
  }
  var stack = this.listeners[event.type].slice();

  for (var i = 0, l = stack.length; i < l; i++) {
    stack[i].call(this, event);
  }
  return !event.defaultPrevented;
};
```

我这一看，这不就是这几天正在研究的观察者模式么，EventTarget 是一个构造函数，是被观察的主体，在原型上绑定了三个方法：

* `addEventListener()` - 在 EventTarget 上注册特定事件类型的事件处理程序。
* `removeEventListener()` - 从 EventTarget 中删除一个事件侦听器。
* `dispatchEvent()` - 将事件分派到此 EventTarget。

其中的 listener 就是监听的事件，类型和回调函数。

## EventListener

接下来，我们再来看 **[EventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventListener)**

> EventListener 接口表示一个对象，可以处理 EventTarget 对象发出的事件。
>
> 由于需要与遗留内容兼容，EventListener 接受一个函数，或者带有 handleEvent() 属性函数的对象。如下例所示。

```html
<button id="btn">Click here!</button>
```

```js
const buttonElement = document.getElementById('btn');

// 通过提供回调函数为'click'事件添加处理程序。
// Whenever the element is clicked, a pop-up with "Element clicked!" will
// appear.
buttonElement.addEventListener('click', function (event) {
  alert('Element clicked through function!');
});

// For compatibility, a non-function object with a `handleEvent` property is
// treated just the same as a function itself.
buttonElement.addEventListener('click', {
  handleEvent: function (event) {
    alert('Element clicked through handleEvent property!');
  }
});
```

这里我们看到一开始抛出的，addEventListener 的第二个参数可以是一个对象，里面有一个函数属性，名字为 `handleEvent`。

按理说这篇文章到这，就差不多了，但是我发现好多跟 `addEventListener` 相关的东西，于是再进一步探索一下。

## addEventListener

先列一下语法：

```
target.addEventListener(type, listener[, options]);
target.addEventListener(type, listener[, useCapture]);
target.addEventListener(type, listener[, useCapture, wantsUntrusted  ]); // Gecko/Mozilla only
```

再看一下每个参数：

* type-表示要侦听的事件类型，是一个大小写敏感的字符串

* listener - 当发生指定类型的事件时接收通知(实现 Event 接口的对象)的对象。这必须是实现EventListener 接口的对象，或 JavaScript 函数。

* options

  一个指定有关 listener 属性的可选参数对象。可用的选项如下：

  - `capture`:  `Boolean`，表示 `listener` 会在该类型的事件捕获阶段传播到该 `EventTarget` 时触发。
  - `once`:  `Boolean`，表示 `listener 在添加之后最多只调用一次。如果是` `true，` `listener` 会在其被调用之后自动移除。
  - `passive`: `Boolean`，表示 `listener` 永远不会调用 `preventDefault()。如果 listener 仍然调用了这个函数，客户端将会忽略它并抛出一个控制台警告。`
  - ` mozSystemGroup`: 只能在 XBL 或者是 Firefox' chrome 使用，这是个 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，表示 `listener `被添加到 system group。

在说到第三个参数的时候，就必须了解 DOM 的事件流，这里我们从W3C工作草案 [DOM Level 3 Events](http://www.w3.org/TR/DOM-Level-3-Events/#event-flow) 开始了解。

## DOM Event Architecture

### 事件分发和DOM事件流-Event dispatch and DOM event flow

本章节会简单介绍事件分发机制，并描述事件是如何通过DOM树进行传播的。应用程序可以使用 `dispatchEvent()` 方法分发事件对象，事件对象将通过DOM事件流确定的DOM树传播。

![Graphical representation of an event dispatched in a DOM tree using the DOM event flow](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)

事件对象被分发到一个event target上。但是在开始调度之前，必须先确定事件对象的传播路径。

传播路径是事件通过的当前事件目标的有序列表。









