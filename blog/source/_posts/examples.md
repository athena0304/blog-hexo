---
title: 【译】通过例子解释 Debounce 和 Throttle
---

**Debounce** 和 **Throttle** 是两个很相似但是又不同的技术，都可以控制一个函数在一段时间内执行的次数。

当我们在操作 DOM 事件的时候，为函数添加 debounce 或者 throttle 就会尤为有用。为什么？因为我们在事件和函数执行之间加了一个我们自己的控制层。记住，我们是不去控制这些 DOM 事件触发的频率的，因为这个可能会有变化。

下面我们以滚动事件举例：

<iframe height='265' scrolling='no' title='Scroll events counter' src='//codepen.io/athena0304/embed/Yjbqar/?height=265&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/athena0304/pen/Yjbqar/'>Scroll events counter</a> by Athena (<a href='https://codepen.io/athena0304'>@athena0304</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

当使用触控板、鼠标滚轮，或者直接拽动滚动条，每秒都可以轻易触发至少30次事件，而且在触屏的移动端，甚至会达到每秒100次，面对这样高的执行频率，你的滚动事件处理程序能否很好地应对？

在2011年，Twitter 网站提出了一个 issue：当向下滚动 Twitter 信息流的时候，整个页面的响应速度都会变慢。 John Resig 基于该问题发表了一篇[博客](http://ejohn.org/blog/learning-from-twitter)，文中指出，直接在 `scroll` 事件里挂载一些计算量大的函数是件多么不明智的行为。

John 当时提出的解决方案是在 `onScroll event` 的外部设置一个每 250ms 执行一次的循环。这样处理程序就与事件解耦了。使用这样一个简单的技术就可以避免破坏用户体验。

::: tip 译者注

文中的核心代码如下

```js
var outerPane = $details.find(".details-pane-outer"),
    didScroll = false;

$(window).scroll(function() {
    didScroll = true;
});

setInterval(function() {
    if ( didScroll ) {
        didScroll = false;
        // Check your page position and then
        // Load in more results
    }
}, 250);
```

:::

如今，处理事件的方式稍微复杂了一些。下面我们结合用例，一一介绍 Debounce、 Throttle 和requestAnimationFrame。

### Debounce

Debounce 允许我们将多个连续的调用合并成一个。

![img](https://css-tricks.com/wp-content/uploads/2016/04/debounce.png)



想象一个进电梯的场景，你走进了电梯，门刚要关上，这时另一个人想要进来，于是电梯没有移动楼层（处理函数），而是将门打开让那个人进来。这时又有一个人要进来，就又会上演刚才那一幕。也就是说，电梯延迟了它的函数（移动楼层）执行，但是优化了资源。

在下面的例子中，尝试快速点击按钮或者在上面滑动：

<iframe height='265' scrolling='no' title='Debounce. Trailing' src='//codepen.io/athena0304/embed/NBVjRB/?height=265&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/athena0304/pen/NBVjRB/'>Debounce. Trailing</a> by Athena (<a href='https://codepen.io/athena0304'>@athena0304</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

你可以看到连续快速事件是怎样被一个单独的 debounce 事件所替代的。但是如果事件触发时间间隔较长，就不会发生 debounce。

#### Leading 边缘 (或者 "immediate")

在上面的例子中，你会发现 debounce 事件会等到快速事件停止发生后才会触发函数执行。为什么不在每次一开始就立即触发函数执行呢，这样它的表现就和原始的没有去抖的处理器一样了。直到快速调用出现停顿的时候，才会再次触发。

下面是使用 `leading` 标识符的例子：

![img](https://css-tricks.com/wp-content/uploads/2016/04/debounce-leading.png)

在 underscore.js 中，该选项叫作 `immediate` ，而不是  `leading`。

自己试一下：

<iframe height='265' scrolling='no' title='Debounce. Leading' src='//codepen.io/athena0304/embed/mGbgGo/?height=265&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/athena0304/pen/mGbgGo/'>Debounce. Leading</a> by Athena (<a href='https://codepen.io/athena0304'>@athena0304</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



#### Debounce 的实现

Debounce 的概念和实现最早是由 [John Hann](http://unscriptable.com/2009/03/20/debouncing-javascript-methods/) 在2009年提出来的。

不久之后，Ben Alman 就写了一个 [jQuery 插件](http://benalman.com/projects/jquery-throttle-debounce-plugin/)（现在已经不再维护了），一年之后 Jeremy Ashkenas 把它添加进了 underscore.js。再后来被添加进 Lodash。

这三个实现在内部有一点不同，但是接口几乎是相同的。

曾经有一段时间，underscore 采取了 Lodash 里面的 debounce/throttle 实现，但是后来我在2013年发现了  `_.debounce` 函数的一个 bug。从那时起，这两种实现就出现分化了。

Lodash 为  `_.debounce` 和 `_.throttle`  添加了更多的特性。最初的  `immediate` 标识符被 `leading` 和 `trailing `所替代。你可以选择一个选项，也可以两个都要。默认情况下 `trailing` 是被开启的。

新的  `maxWait` 选项（目前只存在于Lodash）在本文中没有提及，但是它也是一个很有用的选项。实际上，throttle 函数就是使用  `_.debounce` 带着 `maxWait` 的选项来定义的，你可以在这里查看[源码](https://github.com/lodash/lodash/blob/4.8.0-npm/throttle.js)。

#### Debounce 举例

##### Resize 举例

通过拖拽浏览器窗口，可以触发很多次 `resize` 事件。

例子如下：

<iframe height='265' scrolling='no' title='Debounce Resize Event Example' src='//codepen.io/athena0304/embed/KxPLZy/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/athena0304/pen/KxPLZy/'>Debounce Resize Event Example</a> by Athena (<a href='https://codepen.io/athena0304'>@athena0304</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
可以看到，我们在 resize 事件上使用的是默认的  `trailing` 选项，因为我们只需要关心用户停止调整浏览器后的最终结果就可以了。

##### 敲击键盘，通过 Ajax 请求自动填充表单

为什么要在用户还在输入的时候每隔 50ms 就发送一次 Ajax请求？`_.debounce` 可以帮助我们避免额外的开销，只有当用户停止输入了再发送请求。

这里没有必要设置  `leading`，我们是想要等到最后一个字符输入完再执行函数的。

<iframe height='265' scrolling='no' title='Debouncing keystrokes Example' src='//codepen.io/athena0304/embed/NLKVZw/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/athena0304/pen/NLKVZw/'>Debouncing keystrokes Example</a> by Athena (<a href='https://codepen.io/athena0304'>@athena0304</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
还有一个类似的使用场景就是表单校验，当用户输入完再进行校验、提示信息等。

#### 如何使用 debounce 和 throttle，以及常见问题

说了这么多，你可能已经想自己来写 debounce/throttle 函数了，或者是从网上随便一篇博客上拷贝一份下来。**但是我给你的建议是直接使用 underscore 或者 Lodash。** 如果你只是需要 `_.debounce` 和 `_.throttle` 函数，可以使用 [Lodash custom builder](https://lodash.com/custom-builds) 来输出一个自定义的压缩后为 2KB 的库。可以使用下列命令来进行构建：

```bash
npm i -g lodash-cli
lodash include = debounce, throttle
```

也就是说，最好是使用模块化的形式，通过 webpack/browserify/rollup 来引用，如 `lodash/throttle` 和 `lodash/debounce` 或 `lodash.throttle` 和 `lodash.debounce` 。

使用 `_.debounce` 函数的一个常见错误就是多次调用它：

```js
// 错误
$(window).on('scroll', function() {
   _.debounce(doSomething, 300); 
});

// 正确
$(window).on('scroll', _.debounce(doSomething, 200));
```

为 debounced 函数创建一个变量可以让我们调用私有函数 `debounced_version.cancel()`，如果有需要，lodash 和 underscore.js 都可以供你使用。

```js
var debounced_version = _.debounce(doSomething, 200);
$(window).on('scroll', debounced_version);

// 如果你需要的话
debounced_version.cancel();
```

### Throttle

使用 `_.throttle` 则不允许函数每 X 毫秒的执行次数超过一次。

Throttle 和 debounce 最主要的区别就是 throttle 保证函数每 X 毫秒至少执行一次。

和 debounce 一样， throttle 也用在了 Ben 的插件、underscore.js 和 lodash里面。

#### Throttling 举例

##### 无限滚动

这是一个非常常见的例子。用户在一个无限滚动的页面里向下滚动，你需要知道当前滚动的位置距离底部还有多远，如果接近底部了，我们就得通过 Ajax 请求获取更多的内容，将其添加到页面里。

此时我们之前的 `_.debounce` 就派不上作用了。使用 debounce 只有当用户停止滚动时才能触发，而我们需要的是在用户滚动到底部之前就开始获取内容。

使用 `_.throttle` 就能确保实时检查距离底部还有多远。

### requestAnimationFrame (rAF)

`requestAnimationFrame` 是另一种限制函数执行速度的方法。

它可以被看做 `_.throttle(dosomething, 16)`。但它有着更高的保真度，因为它是浏览器的原生 API，有着更好的精度。

我们可以使用 rAF API，作为 throttle 函数的替代，考虑下面的优缺点：

#### 优点

- 目标是 60fps（每帧 16ms），但是会在浏览器内部决定如何安排渲染的最佳时机。
- 非常简单，而且是标准 API，在未来也不会改变。更少的维护成本。

#### 缺点

- rAFs 的开始/取消由我们自己来管理，而不像  `.debounce` 和 `.throttle` 是在内部管理的。
- 如果浏览器的 tab 页面不活跃了，它就不会再执行。
- 虽然所有的现代浏览器都提供了 rAF， 但是 IE9、Opera Mini 和一些老的安卓版本还不支持。如果需要，现在还是要使用 [polyfill](http://www.paulirish.com/2011/requestanimationframe-for-smart-animating/) 。
- Node.js 不支持 rAF，所以不能在服务端用于 throttle 文件系统事件。

根据经验，如果你的 JavaScript 函数是在绘制或者直接改变属性，所有涉及到元素位置重新计算的，我会建议使用  `requestAnimationFrame`，

如果是处理 Ajax 请求，或者决定是否添加/删除某个 class（可能会触发一个 CSS 动画），我会考虑 `_.debounce` 和 `_.throttle`，这里可以设置更低一些的执行速度（例如 200ms，而不是16ms）。

这时你可能会想，为什不把 rAF 集成到 underscore 或 lodash 里呢，那他俩都是拒绝的，因为这只是一个特殊的使用场景，而且已经足够简单，可以被直接调用。

#### rAF 举例

受[这篇文章](http://www.html5rocks.com/en/tutorials/speed/animations/)的启发，在这里我会举一个滚动的例子，在这篇文章中有每个步骤的逻辑解释。

我做了一个对比实验，一边是 rAF，一边是 16ms 间隔的 `_.throttle`。它们性能很相似，但是 rAF 可能会在更复杂的场景下性能更高一些。

<iframe height='265' scrolling='no' title='Scroll comparison requestAnimationFrame vs throttle' src='//codepen.io/dcorb/embed/pgOKKw/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/dcorb/pen/pgOKKw/'>Scroll comparison requestAnimationFrame vs throttle</a> by Corbacho (<a href='https://codepen.io/dcorb'>@dcorb</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

还有一个更高级的例子，在 headroom.js 中，逻辑被[解耦](https://github.com/WickyNilliams/headroom.js/blob/3282c23bc69b14f21bfbaf66704fa37b58e3241d/src/Debouncer.js)了，并且包裹在了对象中。

### 总结

使用 debounce、throttle 和  `requestAnimationFrame`  来优化你的事件处理程序。每种技术都有些许的不同，但是三个都是很有用的，而且能够互补。

总结：

- **debounce**：将一系列迅速触发的事件（例如敲击键盘）合并成一个单独的事件。
- **throttle**：确保一个持续的操作流以每 X 毫秒执行一次的速度执行。例如每 200ms 检查一下滚动条的位置来触发某个 CSS 动画。
- **requestAnimationFrame**：throttle的一个替代品。适用于需要计算元素在屏幕上的位置和渲染的时候，能够保证动画或者变化的平滑性。注意：IE9 不支持。



原文链接：https://css-tricks.com/debouncing-throttling-explained-examples/