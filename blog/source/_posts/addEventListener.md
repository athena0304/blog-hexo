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

可以发现，这里的addEventListener传的第二个参数是一个对象，那么这个对象怎么就能解析事件呢，以前的我只知道第二个参数应该传一个函数，而从没使用过对象的这种方式。于是查了一下MDN，果然：
