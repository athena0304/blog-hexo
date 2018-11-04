---
title: 一个由addEventListener参数问题所引发的阅读
date: 2018-10-29 21:21:31
tags: ['addEventListener']
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

* listener - 当发生指定类型的事件时接收通知（实现 Event 接口的对象）的对象。这必须是实现 EventListener 接口的对象，或 JavaScript 函数。

* options

  指定事件侦听器特征的选项对象。可用的选项如下：

  - `capture`:  `Boolean`，表示 `listener` 会在该类型的事件捕获阶段传播到该 `EventTarget` 时触发。
  - `once`:  `Boolean`，表示 `listener 在添加之后最多只调用一次。如果是` `true，` `listener` 会在其被调用之后自动移除。
  - `passive`: `Boolean`，如果为真，则表示 `listener` 永远不会调用 `preventDefault()。如果 listener 仍然调用了这个函数，客户端将会忽略它并抛出一个控制台警告。`
  - ` mozSystemGroup`: 只能在 XBL 或者是 Firefox' chrome 使用，这是个 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，表示 `listener `被添加到 system group。

在说到第三个参数的时候，就必须了解 DOM 的事件流，这里我们从W3C工作草案 [DOM Level 3 Events](http://www.w3.org/TR/DOM-Level-3-Events/#event-flow) 开始了解。

* useCapture

  一个布尔值，默认为false。指示在将此类型的事件分派到DOM树中它下面的任何EventTarget之前，是否会将此类型的事件分派给已注册的侦听器。通过树向上冒泡的事件不会触发指定使用捕获的侦听器。当两个元素都为该事件注册了句柄时，事件冒泡和捕获是传播在嵌套在另一个元素中的元素中发生的事件的两种方法。事件传播模式确定元素接收事件的顺序。

## DOM Event Architecture

### 术语表

在看规范正文之前，不得不说，还得看一下一些术语和翻译的对应，不然用以被这些名字搞懵。

#### Event-事件

看一下[接口描述](https://dom.spec.whatwg.org/#interface-event)：

```idl
[Constructor(DOMString type, optional EventInit eventInitDict),
 Exposed=(Window,Worker,AudioWorklet)]
interface Event {
  readonly attribute DOMString type;
  readonly attribute EventTarget? target;
  readonly attribute EventTarget? srcElement; // historical
  readonly attribute EventTarget? currentTarget;
  sequence<EventTarget> composedPath();

  const unsigned short NONE = 0;
  const unsigned short CAPTURING_PHASE = 1;
  const unsigned short AT_TARGET = 2;
  const unsigned short BUBBLING_PHASE = 3;
  readonly attribute unsigned short eventPhase;

  void stopPropagation();
           attribute boolean cancelBubble; // historical alias of .stopPropagation
  void stopImmediatePropagation();

  readonly attribute boolean bubbles;
  readonly attribute boolean cancelable;
           attribute boolean returnValue;  // historical
  void preventDefault();
  readonly attribute boolean defaultPrevented;
  readonly attribute boolean composed;

  [Unforgeable] readonly attribute boolean isTrusted;
  readonly attribute DOMHighResTimeStamp timeStamp;

  void initEvent(DOMString type, optional boolean bubbles = false, optional boolean cancelable = false); // historical
};

dictionary EventInit {
  boolean bubbles = false;
  boolean cancelable = false;
  boolean composed = false;
};
```

事件对象简单地称为事件。它允许发送发生了什么事情的信号，例如，图像已经完成下载。

```
event = new Event(type [, eventInitDict])
```

返回一个新事件，其 `type` 属性值设置为 type。eventInitDictargument 允许通过同名对象成员设置 `bubbles` 和 `cancelable` 属性。

```
event . type
```

返回事件的类型， e.g. "`click`", "`hashchange`", or "`submit`".

```
event . target
```

返回发送事件的对象(其目标)。

```
event . currentTarget
```

返回当前正在调用其事件侦听器回调的对象。

```
event . composedPath()
```

Returns the [item](https://dom.spec.whatwg.org/#event-path-item) objects of event’s [path](https://dom.spec.whatwg.org/#event-path) (objects on which listeners will be invoked), except for any [nodes](https://dom.spec.whatwg.org/#concept-node) in [shadow trees](https://dom.spec.whatwg.org/#concept-shadow-tree) of which the [shadow root](https://dom.spec.whatwg.org/#concept-shadow-root)’s [mode](https://dom.spec.whatwg.org/#shadowroot-mode) is "`closed`" that are not reachable from event’s `currentTarget`.

```
event . eventPhase
```

Returns the [event](https://dom.spec.whatwg.org/#concept-event)’s phase, which is one of `NONE`, `CAPTURING_PHASE`, `AT_TARGET`, and `BUBBLING_PHASE`.

```
event . stopPropagation()
```

在树中分派时，调用此方法可防止事件到达当前对象以外的任何对象。

```
event . stopImmediatePropagation()
```

Invoking this method prevents event from reaching any registered [event listeners](https://dom.spec.whatwg.org/#concept-event-listener) after the current one finishes running and, when [dispatched](https://dom.spec.whatwg.org/#concept-event-dispatch) in a [tree](https://dom.spec.whatwg.org/#concept-tree), also prevents eventfrom reaching any other objects.

```
event . bubbles
```

根据事件初始化的方式返回 true 或 false。如果事件以相反的树顺序通过目标的祖先，则为真，否则为假。

```
event . cancelable
```

根据事件初始化的方式返回 true 或 false。它的返回值并不总是有意义，但是 true 可以指示在事件被分派期间，可以通过调用 preventDefault() 方法来取消部分操作。

```
event . preventDefault()
```

If invoked when the `cancelable` attribute value is true, and while executing a listener for the event with `passive` set to false, signals to the operation that caused event to be [dispatched](https://dom.spec.whatwg.org/#concept-event-dispatch) that it needs to be canceled.

```
event . defaultPrevented
```

Returns true if `preventDefault()` was invoked successfully to indicate cancelation, and false otherwise.

```
event . composed
```

Returns true or false depending on how event was initialized. True if event invokes listeners past a `ShadowRoot` [node](https://dom.spec.whatwg.org/#boundary-point-node) that is the [root](https://dom.spec.whatwg.org/#concept-tree-root) of its [target](https://dom.spec.whatwg.org/#event-target), and false otherwise.

```
event . isTrusted
```

Returns true if event was [dispatched](https://dom.spec.whatwg.org/#concept-event-dispatch) by the user agent, and false otherwise.

```
event . timeStamp
```

Returns the event’s timestamp as the number of milliseconds measured relative to the [time origin](https://w3c.github.io/hr-time/#dfn-time-origin).

#### [dispatch](https://www.w3.org/TR/DOM-Level-3-Events/#dispatch)-分派、派发、分发

创建一个事件，该事件具有与其类型和上下文相匹配的属性和方法，并以指定的方式通过DOM树进行传播。可以与术语 fire 互换，例如，触发一个单击事件（fire a click event）或分派一个加载事件（dispatch a load event）。

#### [event target](https://www.w3.org/TR/DOM-Level-3-Events/#event-target)-事件目标

事件的目标对象，使用事件派发和DOM事件流。**事件目标**是 event 的 target 属性的值。也就是 event.target。

#### [propagation path](https://www.w3.org/TR/DOM-Level-3-Events/#propagation-path)-传播路径

**当前事件目标**的有序集合，事件对象将在进出事件目标的过程中按顺序传递。当事件传播时，**传播路径**中的每个当前事件目标依次设置为currentTarget。**传播路径**最初由事件类型定义的一个或多个事件阶段组成，但可能会被中断。也称为事件目标链。

#### [current event target](https://www.w3.org/TR/DOM-Level-3-Events/#current-event-target)-当前事件目标

在事件流中，**当前事件目标**是与当前派发的事件处理程序关联的对象。此对象可能是目标事件本身或其祖先之一。当事件通过事件流的各个阶段从一个对象传播到另一个对象时，**当前事件目标**会发生变化。**当前事件目标**是 currentTarget 属性的值。

#### [event handler/event listener](https://www.w3.org/TR/DOM-Level-3-Events/#event-handler)-事件处理程序/事件侦听器

实现EventListener接口并提供handleEvent()回调方法的对象。事件处理程序是特定于语言的。事件处理程序在特定对象(**当前事件目标**)的上下文中调用，并提供事件对象本身。

#### phase-阶段

在事件的上下文中，阶段是一组沿着DOM树的逻辑遍历，从 Window 到 `Document` 对象、根元素，然后下到事件对象（捕获阶段），再到事件对象本身（目标阶段），然后沿原路返回（冒泡阶段）。

### 事件分发和DOM事件流-Event dispatch and DOM event flow

本章节会简单介绍**事件分派**机制，并描述事件是如何通过DOM树进行传播的。应用程序可以使用 `dispatchEvent()` 方法分派事件对象，事件对象将通过DOM事件流确定的DOM树传播。

![Graphical representation of an event dispatched in a DOM tree using the DOM event flow](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)

事件对象被分发到一个**事件目标**上。但是在开始分派之前，必须先确定事件对象的**传播路径**。

**传播路径**是事件通过**当前事件目标**的有序列表。这个传播路径反映了文档的层次树结构。列表中的最后一项是**事件目标**，在这之前的都称为目标的祖先项，挨着事件目标的称作目标的父项。

一旦传播路径决定了，事件对象就会经过一个或多个**事件阶段**。有三个事件阶段：捕获阶段、目标阶段和冒泡阶段。事件对象会像下面描述的这样来完成这些阶段。如果不支持某个阶段，或者事件对象的传播已经停止，则跳过该阶段。例如，如果 `bubbles` 属性设成了 false，就会跳过冒泡阶段，如果在分派之前就调用了 `stopPropagation()`，所有阶段都会被跳过。

* 捕获阶段（capture phase）：事件对象通过对象的祖先项，从 Window 传播到目标的父项。该阶段也被称为 capturing phase （反正中文翻译过来都是捕获阶段）

* 目标阶段（target phase）：事件对象到达事件对象的事件目标上面。该阶段也被称作 at-target phase （反正翻译过来还是目标阶段）。如果事件类型指出该事件不进行冒泡，则事件对象将在此阶段结束后停止。
* 冒泡阶段（bubble phase）：事件对象以相反的顺序通过目标的祖先项传播，从目标的父项开始，到 Window 结束。该阶段也被称为 bubbling phase（反正翻译过来还是冒泡阶段）

### 默认动作和可取消事件

事件通常由实现作为用户操作的结果进行分派，比如为了响应任务的完成，或者在异步活动期间发送进程信号（例如网络请求）。可以使用一些事件来控制实现接下来可能采取的行为(或撤消实现已经采取的动作)。这个类别中的事件称为可取消的（cancelable），它们取消的行为称为默认行为（default action）。可取消事件对象可以与一个或多个“默认动作”相关联。要取消事件，请调用 preventDefault() 方法。

> 例子1：当用户按下指向设备(通常是鼠标)上的按钮后，立即会发出mousedown事件。实现可能采取的一个默认操作是设置一个状态机，允许用户拖动图像或选择文本。默认操作取决于接下来会发生什么——例如，如果用户的指向设备位于文本之上，可能会开始文本选择。如果用户的指向设备位于图像上方，则可以开始图像拖动操作。防止mousedown事件的默认操作将阻止这些操作的发生。

默认操作通常在事件分派完成之后执行，但在特殊情况下，它们也可以在事件分派之前立即执行。

总结：感觉光看是不行的，还是要多实践，才能明白。



