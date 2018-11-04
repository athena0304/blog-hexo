---
title: 【译】简单理解 JavaScript 中的设计模式
---

当你开始一个新的项目时，一般不会立即开始编码，首先必须要定义项目的目的和范围，然后列出项目的特性或规格。在可以开始编写代码之后，或者如果你正在处理一个更复杂的项目，那么你应该选择一个最适合该项目的设计模式。

## 什么是设计模式？

在软件工程中，设计模式是软件设计中常见问题的可重用解决方案。设计模式代表了经验丰富的软件开发人员所使用的最佳实践。设计模式可以看作是编程模板。

### 为什么要使用设计模式？

很多程序员要么认为设计模式浪费时间，要么不知道如何恰当地应用它们。但是使用适当的设计模式可以帮助你编写更好、更易于理解的代码，且更容易维护。

最重要的是，设计模式为软件开发人员提供了一个可以讨论的通用词汇表。它们会立即向学习代码的人显示代码的意图。

例如，如果你在项目中使用装饰者模式，那么新加入的开发人员将立即知道这段代码在做什么，所以他们可以更关注于解决业务问题，而不是试图理解代码在做什么。

现在我们已经知道了什么是设计模式，以及它们为什么重要，接下来让我们深入研究 JavaScript 中使用的各种设计模式。

## 模块模式-Module Pattern

模块是一段自包含的代码，因此我们可以在不影响代码其他部分的情况下更新模块。模块还允许我们通过为变量创建单独的作用域来避免命名空间污染。当模块与其他代码片段分离时，我们还可以在其他项目中重用它们。

模块是任何现代 JavaScript 应用程序的组成部分，它有助于保持代码的整洁、分离和组织。用 JavaScript 创建模块有很多方法，模块模式是其中之一。

与其他编程语言不同，JavaScript没有访问修饰符，也就是说，不能将变量声明为私有或公共。因此，模块模式也被用来模拟封装的概念。

模块模式使用IIFE(立即调用的函数表达式)、闭包和函数作用域来模拟此概念。例如:

```js
const myModule = (function() {
  
  const privateVariable = 'Hello World';
  
  function privateMethod() {
    console.log(privateVariable);
  }
  return {
    publicMethod: function() {
      privateMethod();
    }
  }
})();
myModule.publicMethod();
```

由于它是 IIFE，代码立即被执行，返回的对象被分配给 `myModule` 变量。因为闭包，所以返回的对象仍然可以访问在IIFE中定义的函数和变量，即使在 IIFE 结束之后。

在 IIFE 中定义的变量和函数在外部作用域是不可见的，于是它们成为了 `myModule` 的私有变量。

代码执行过后，`myModule` 变量看上去是这样的：

```js
const myModule = {
  publicMethod: function() {
    privateMethod();
  }};
```

因此，我们可以调用 `publicMethod()`，然后依次调用 `privateMethod()`。例如:

```js
// Prints 'Hello World'
module.publicMethod();
```



## 揭示模块模式-Revealing Module Pattern

揭示模块模式是由 Christian Heilmann 基于模块模式略微改进的版本。模块模式的问题是，我们必须创建新的公共函数，仅仅是用来调用私有函数和变量。

在这个模式中，我们将返回对象的属性映射到我们想要公开的私有函数。这就是为什么它被称为揭示模块模式。例如:

```js
const myRevealingModule = (function() {
  
  let privateVar = 'Peter';
  const publicVar  = 'Hello World';
  function privateFunction() {
    console.log('Name: '+ privateVar);
  }
  
  function publicSetName(name) {
    privateVar = name;
  }
  function publicGetName() {
    privateFunction();
  }
  /** reveal methods and variables by assigning them to object properties */
return {
    setName: publicSetName,
    greeting: publicVar,
    getName: publicGetName
  };
})();
myRevealingModule.setName('Mark');
// prints Name: Mark
myRevealingModule.getName();
```

这种模式使我们更容易理解哪些函数和变量可以公开访问，有助于代码的可读性。

执行代码之后，`myRevealingModule` 如下：

```js
const myRevealingModule = {
  setName: publicSetName,
  greeting: publicVar,
  getName: publicGetName
};
```

我们可以调用 `myRevealingModule.setName('Mark')`，它是对内部 `publicSetName` 的引用，还可以调用 `myRevealingModule.getName()` ，是对内部 `publicGetName` 的引用。例如:

```javascript
myRevealingModule.setName('Mark');
// prints Name: Mark
myRevealingModule.getName();
```

### 揭示模块模式相比模块模式的优势：

- 通过修改 return 语句中的一行代码，我们可以将成员从 public 更改为 private，反之亦然。
- 返回的对象不包含任何函数定义，所有右侧表达式都在IIFE中定义，使代码清晰且易于阅读。

## ES6模块-ES6 Modules

在 ES6 之前，JavaScript 没有内置模块，因此开发人员不得不依赖第三方库或模块模式来实现模块。但是在 ES6中，JavaScript 有原生模块。

ES6模块存储在文件中。每个文件只能有一个模块。默认情况下，模块中的所有内容都是私有的。使用 `export` 关键字公开函数、变量和类。模块内的代码总是在严格模式下运行。

### 导出模块

导出函数和变量声明有两种方法：

- 通过在函数和变量声明前添加 `export` 关键字。例如：

```js
// utils.js
export const greeting = 'Hello World';
export function sum(num1, num2) {
  console.log('Sum:', num1, num2);
  return num1 + num2;
}
export function subtract(num1, num2) {
  console.log('Subtract:', num1, num2);
  return num1 - num2;
}
// This is a private function
function privateLog() {
  console.log('Private Function');
}
```

- 通过在包含要导出的函数和变量名称的代码末尾添加 `export` 关键字。例如：

```javascript
// utils.js
function multiply(num1, num2) {
  console.log('Multiply:', num1, num2);
  return num1 * num2;
}
function divide(num1, num2) {
  console.log('Divide:', num1, num2);
  return num1 / num2;
}
// This is a private function
function privateLog() {
  console.log('Private Function');
}
export {multiply, divide};
```

### 导入模块

与导出模块类似，有两种方法可以使用 `import` 关键字导入模块。例如：

- 一次导入多个项

```js
// main.js
// importing multiple items
import { sum, multiply } from './utils.js';
console.log(sum(3, 7));
console.log(multiply(3, 7));
```

- 导入所有内容

```js
// main.js
// importing all of module
import * as utils from './utils.js';
console.log(utils.sum(3, 7));
console.log(utils.multiply(3, 7));
```

### 导入和导出可以重命名

如果希望避免命名冲突，可以在导入和导出时修改命名，例如：

- 重命名导出

```js
// utils.js
function sum(num1, num2) {
  console.log('Sum:', num1, num2);
  return num1 + num2;
}
function multiply(num1, num2) {
  console.log('Multiply:', num1, num2);
  return num1 * num2;
}
export {sum as add, multiply};
```

- 重命名导入

```js
// main.js
import { add, multiply as mult } from './utils.js';
console.log(add(3, 7));
console.log(mult(3, 7));
```

## 单例模式-Singleton Pattern

单例对象是只能实例化一次的对象。如果一个类不存在，单例模式会创建一个新的类实例。如果实例存在，它只返回对该对象的引用。对构造函数的任何重复调用都将获取相同的对象。

在 JavaScript 中，一直都有内置的单例。我们只是不称它们为单例，而是对象字面量（object literal）。例如：

```javascript
const user = {
  name: 'Peter',
  age: 25,
  job: 'Teacher',
  greet: function() {
    console.log('Hello!');
  }
};
```

因为 JavaScript 中的每个对象都占用一个唯一的内存位置，当我们调用 `user` 对象时，实际上是返回对这个对象的引用。

如果我们试图将 `user` 变量复制到另一个变量中并修改该变量。例如：

```js
const user1 = user;
user1.name = 'Mark';
```

我们会看到两个对象都被修改了，因为 JavaScript 中的对象是通过引用而不是值传递的。所以内存中只有一个对象。例如：

```js
// prints 'Mark'
console.log(user.name);
// prints 'Mark'
console.log(user1.name);
// prints true
console.log(user === user1);
```

单例模式可以使用构造函数实现。例如：

```js
let instance = null;
function User() {
  if(instance) {
    return instance;
  }
  instance = this;
  this.name = 'Peter';
  this.age = 25;
  
  return instance;
}
const user1 = new User();
const user2 = new User();
// prints true
console.log(user1 === user2); 
```

当调用这个构造函数时，检查 `instance` 对象是否存在。如果对象不存在，则将 `this` 变量赋给 `instance` 变量。如果对象存在，则只返回那个对象。

单例模式也可以使用模块模式来实现。例如：

```js
const singleton = (function() {
  let instance;
  
  function init() {
    return {
      name: 'Peter',
      age: 24,
    };
  }
  return {
    getInstance: function() {
      if(!instance) {
        instance = init();
      }
      
      return instance;
    }
  }
})();
const instanceA = singleton.getInstance();
const instanceB = singleton.getInstance();
// prints true
console.log(instanceA === instanceB);
```

在上面的代码中，我们通过调用 `singleton.getInstance` 方法来创建一个新的实例。如果实例已经存在，此方法只返回该实例，如果实例不存在，则通过调用 `init()` 函数创建新实例。

## 工厂模式-Factory Pattern

工厂模式使用工厂方法创建对象，而不指定创建对象的确切类或构造函数。

工厂模式用于创建对象，而不公开实例化逻辑。当我们需要根据特定条件生成不同的对象时，可以使用此模式。例如：

```js
class Car{
  constructor(options) {
    this.doors = options.doors || 4;
    this.state = options.state || 'brand new';
    this.color = options.color || 'white';
  }
}
class Truck {
  constructor(options) {
    this.doors = options.doors || 4;
    this.state = options.state || 'used';
    this.color = options.color || 'black';
  }
}
class VehicleFactory {
  createVehicle(options) {
    if(options.vehicleType === 'car') {
      return new Car(options);
    } else if(options.vehicleType === 'truck') {
      return new Truck(options);
      }
  }
}
```

这里我创建了一个 `Car` 和一个 `Truck` 类（带有一些默认值），用于创建新的 `car` 和 `truck` 对象。我还定义了一个 `VehicleFactory` 类，它根据 `options` 对象中接收到的 `vehicleType` 属性创建并返回一个新的对象。

```js
const factory = new VehicleFactory();
const car = factory.createVehicle({
  vehicleType: 'car',
  doors: 4,
  color: 'silver',
  state: 'Brand New'
});
const truck= factory.createVehicle({
  vehicleType: 'truck',
  doors: 2,
  color: 'white',
  state: 'used'
});
// Prints Car {doors: 4, state: "Brand New", color: "silver"}
console.log(car);
// Prints Truck {doors: 2, state: "used", color: "white"}
console.log(truck);
```

在上面的代码中，我们基于 `VehicleFactory` 类创建了一个新的对象 `factory`。然后通过调用 `factory.createVehicle`，传入带有 `vehicleType` 属性的 `options` 对象参数，创建了新的 `car` 和 `truck` 对象。

## 装饰者模式-Decorator Pattern

装饰者模式用于扩展对象的功能，而无需修改现有的类或构造函数。此模式可用于向对象添加特性，而无需修改底层代码。

一个简单的例子如下:

```js
function Car(name) {
  this.name = name;
  // Default values
  this.color = 'White';
}
// Creating a new Object to decorate
const tesla= new Car('Tesla Model 3');
// Decorating the object with new functionality
tesla.setColor = function(color) {
  this.color = color;
}
tesla.setPrice = function(price) {
  this.price = price;
}
tesla.setColor('black');
tesla.setPrice(49000);
// prints black
console.log(tesla.color);
```

这种模式的一个更实际的例子是：

比方说，一辆车的价格取决于它有多少功能。如果没有装饰者模式，我们将不得不为不同的特性组合创建不同的类，每个类都有计算成本的方法。例如：

```js
class Car() {
}
class CarWithAC() {
}
class CarWithAutoTransmission {
}
class CarWithPowerLocks {
}
class CarWithACandPowerLocks {
}
```

但是使用装饰者模式，我们可以创建一个基类 `Car`，并使用装饰函数向其对象添加不同配置的成本。例如:

```js
class Car {
  constructor() {
  // Default Cost
  this.cost = function() {
  return 20000;
  }
}
}
// Decorator function
function carWithAC(car) {
  car.hasAC = true;
  const prevCost = car.cost();
  car.cost = function() {
    return prevCost + 500;
  }
}
// Decorator function
function carWithAutoTransmission(car) {
  car.hasAutoTransmission = true;
   const prevCost = car.cost();
  car.cost = function() {
    return prevCost + 2000;
  }
}
// Decorator function
function carWithPowerLocks(car) {
  car.hasPowerLocks = true;
  const prevCost = car.cost();
  car.cost = function() {
    return prevCost + 500;
  }
}
```

首先，我们创建一个基类 `Car` 用来创建 `car` 对象。然后，为要添加的特性创建装饰器，并将 `Car` 对象作为参数传递。然后，我们重写该对象的 `cost` 函数，该函数返回 `car` 更新后的成本，并向该对象添加一个新属性，用来表明添加了哪些特性。

要添加新特性，我们可以这样做：

```js
const car = new Car();
console.log(car.cost());
carWithAC(car);
carWithAutoTransmission(car);
carWithPowerLocks(car);
```

最后，我们可以这样计算汽车的成本：

```js
// Calculating total cost of the car
console.log(car.cost());
```

## 总结

我们已经了解了 JavaScript 中使用的各种设计模式，但是还有一些设计模式没有在这里介绍，它们可以在JavaScript 中实现。

虽然了解各种设计模式很重要，但同样重要的是不要过度使用它们。在使用设计模式之前，你应该仔细考虑你的问题是否符合设计模式。要想知道某个模式是否适合你的问题，请了解该模式解决了哪些问题，并检查你是否实际面临类似的问题。

原文：https://blog.bitsrc.io/understanding-design-patterns-in-javascript-13345223f2dd





















