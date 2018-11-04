---
title: 简单理解一下 Object.create() 与 new 之间的区别
---
首先看一下 `Object.create`：

```js
var dog = {
    eat: function() {
        console.log(this.eatFood)
    }
};

var maddie = Object.create(dog);
console.log(dog.isPrototypeOf(maddie)); //true
maddie.eatFood = 'NomNomNom'; 
maddie.eat(); //NomNomNom
```

上面的代码遵循以下步骤：

1. 创建一个字面量对象 `dog`，并且有一个方法 `eat`。
2. 使用 `Object.create(dog)` 初始化 `maddie`，创建了一个全新的对象，并将该对象的原型设置成 `dog`。
3. 测试 `dog` 是否是 `maddie` 的原型。
4. 通过 `maddie.eatFood` 设置将要输出的变量。
5. 在新创建的 `maddie` 上调用 `eat` 方法。
6. 查找原型链，找到 `dog` 里面的 `eat` 方法，此时 `this` 指向的是 `maddie`
7. 控制台输出 `NomNomNom`。



现在再看一下 `**new**` 操作符：

```js
var Dog = function(){
    this.eatFood = 'NomNomNom';
    this.eat = function(){
        console.log(this.eatFood)
    }
};

var maddie = new(Dog);
console.log(maddie instanceof Dog); // True
maddie.eat(); //NomNomNom
```

这里遵循以下步骤：

1. 创建一个新的对象，`maddie`。
2. `maddie` 继承了构造函数的原型。
3. 执行构造函数，将 `this` 设置成第一步创建的对象。
4. 返回创建的对象（除非构造函数返回了其它对象）。

乍一看上去两者都创建了一个新的对象，然后继承了原型，下面这个例子可以看出区别：

```js
function Dog(){
    this.pupper = 'Pupper';
};

Dog.prototype.pupperino = 'Pups.';
var maddie = new Dog();
var buddy = Object.create(Dog.prototype);

//Using Object.create()
console.log(buddy.pupper); //Output is undefined
console.log(buddy.pupperino); //Output is Pups.

//Using New Keyword
console.log(maddie.pupper); //Output is Pupper
console.log(maddie.pupperino); //Output is Pups.
```

这里有一个关键点就是这句话：

```js
console.log(buddy.pupper); //Output is undefined
```

这里我们发现 `buddy.pupper` 输出的是 `undefined`。即使 `Object.create()` 将 `buddy` 的原型设置成了 `Dog`，但是 `buddy` 还是无法访问到狗在函数中的 `**this.pupper**`。这是因为 `new Dog` 实际上执行了构造函数，而 `Object.create` 没有执行。



原文：https://medium.com/@jonathanvox01/understanding-the-difference-between-object-create-and-the-new-operator-b2a2f4749358



















