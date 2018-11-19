---
title: vue-markdown-loader源码解析
date: 2018-11-16 11:48:15
tags:
---
项目中遇到了需要单独加载某个 markdown 文件显示在页面中，类似于操作指引的感觉，于是找到了 vue-markdown-loader 这个工具，觉得很好用，是我清伟大神写的，于是我打算开个专题看一下里面都做了些什么，有助于我对 webpack loader 和 vue 的理解。

## 准备工作

这里需要先了解 webpack loader 的原理，后面的代码需要配合 webpack 的[官网](https://webpack.js.org/api/loaders/)对照着来看。中文版在[这里](https://webpack.docschina.org/concepts/loaders)。

首先我们需要知道 loader 是什么，那么 loader 其实就是一个 JavaScript module，导出的是一个函数。[loader runner](https://github.com/webpack/loader-runner) 会调用这个函数，将前一个 loader 执行的结果或者源文件作为参数传递进来。函数中的 this 上下文由 webpack 填充，loader runner 有一些实用的方法可以允许 loader 改变其触发方式为异步，或者获取 query 参数。

在这里摘录一下 loader 的特性：

- loader 支持链式传递。loader 链中每个 loader，都对前一个 loader 处理后的资源进行转换。loader 链会按照相反的顺序执行。第一个 loader 将（应用转换后的资源作为）返回结果传递给下一个 loader，依次这样执行下去。最终，在链中最后一个 loader，返回 webpack 所预期的 JavaScript。
- loader 可以是同步的，也可以是异步的。
- loader 运行在 Node.js 中，并且能够执行任何可能的操作。
- loader 接收查询参数。用于对 loader 传递配置。
- loader 也能够使用 `options` 对象进行配置。
- 除了使用 `package.json` 常见的 `main` 属性，还可以将普通的 npm 模块导出为 loader，做法是在 `package.json` 里定义一个 `loader` 字段。
- 插件(plugin)可以为 loader 带来更多特性。
- loader 能够产生额外的任意文件。

了解原理之后，我们还要再看一篇文档，[如何编写一个loader](https://webpack.docschina.org/contribute/writing-a-loader/)。

### 调试工具

如果你想更方便的调试代码，需要配置一下调试环境，这里我用的 VSCode 自带的调试工具（我找了一圈现阶段觉得这种方式最靠谱）

在 debug 模式下点击配置按钮，就会生成一个 `.vscode/launch.json` 的文件，里面的配置改成如下即可：

``` json
{
      "type": "node",
      "request": "launch",
      "name": "启动程序",
      "program": "${workspaceFolder}/node_modules/webpack/bin/webpack.js",
      "cwd": "${workspaceFolder}/example"
    }
```

其中 program 设置成 webpack 本身的文件执行的位置，cwd 设置成执行 webpack 所在的根目录。点击启动程序按钮就可以断点调试了，想看啥看啥。

下面就可以开始看源码了。

## 目录结构
首先看下目录结构：

```bash
.
├── README.md
├── example
│   ├── index.html
│   ├── src
│   │   ├── app.vue
│   │   ├── custom.css
│   │   ├── entry.js
│   │   └── markdown.md
│   └── webpack.config.js
├── index.js
├── lib
│   ├── core.js
│   └── markdown-compiler.js
├── package-lock.json
└── package.json
```

结构很清晰啊，东西也不算太多，涉及到的源码就是 `index.js`和`lib`下的两个js文件，`example`里面的是示例。

那我们就先从 index.js 开始看起吧。

就一句话：

```js
module.exports = require('./lib/core');
```

这个文件是你引入这个包的入口，这里直接去找的`./lib/core`，于是我们继续去 `./lib/core`看看。

在声明阶段：

```js
var path = require('path');
var loaderUtils = require('loader-utils');

var markdownCompilerPath = path.resolve(__dirname, 'markdown-compiler.js');
```

这里引用了 [loader-utils](https://www.npmjs.com/package/loader-utils) 和 同级目录下的 `markdown-compiler.js`

在这里我们逐行解析 `core.js` 里面的代码，遇到什么就去查什么。

```js
module.exports = function(source) {
  // （1）是否可缓存
  this.cacheable(); 
  // （2）获取 options 
  var options = loaderUtils.getOptions(this) || {}; 
  // （3）为 Compilation 对象添加 __vueMarkdownOptions__ 属性
  Object.defineProperty(this._compilation, '__vueMarkdownOptions__', {
    value: options,
    enumerable: false,
    configurable: true
  })
  // (4) 获取资源文件的路径
  var filePath = this.resourcePath;
  // (5) 生成 result
  var result =
    'module.exports = require(' +
    loaderUtils.stringifyRequest(
      this,
      '!!vue-loader!' +
        markdownCompilerPath +
        '?raw!' +
        filePath +
        (this.resourceQuery || '')
    ) +
    ');';

  console.log(result)

  return result;
};
```

首先可以看到这个文件最后是导出了一个 result，参数传进来的是 source，也就是之前 loader 产生过的结果或者是源文件，有个上下文 this。

**（1）是否可缓存**

`this.cacheable();` 对应的是[是否可缓存](https://webpack.docschina.org/api/loaders/#this-cacheable)。

> 默认情况下，loader 的处理结果会被标记为可缓存。调用这个方法然后传入 `false`，可以关闭 loader 的缓存。
>
> 一个可缓存的 loader 在输入和相关依赖没有变化时，必须返回相同的结果。这意味着 loader 除了 `this.addDependency` 里指定的以外，不应该有其它任何外部依赖。

**（2）获取 options** 

参考 loader-utils 的[文档](https://www.npmjs.com/package/loader-utils#getoptions)

是检索调用 loader 的 options 的推荐方式。

本文中为`{}`

**（3）为 Compilation 对象添加` __vueMarkdownOptions__` 属性**

**（4）获取资源文件的路径**

获取到的路径是

```
"vue-markdown-loader/example/src/markdown.md"
```

**（5）生成 result**

```
"module.exports = require("!!vue-loader!../../lib/markdown-compiler.js?raw!./markdown.md");"
```

**参数 source**

这里传进来的 source 就是原始文件：

```
"# Hello`<span>{{sss}}</span>`> This is test.- How are you?- Fine, Thank you, and you?- I'm fine， too. Thank you.- 🌚```javascriptimport Vue from 'vue'Vue.config.debug = true```<div class="test">  {{ model }} test</div><compo>{{ model }}</compo><div  class="abc"  @click="show = false">  啊哈哈哈</div>> All script or style tags in html mark will be extracted.Script will be excuted, and style will be added to document head.> Notice if there is a string instance which contains special word "&lt;/script>", it will fetch a SyntaxError.> Due to the complexity to solve it, just don't do that.```html<style scoped>  .test {    background-color: green;  }</style><style scoped>  .abc {    background-color: yellow;  }</style><script>  let a=1<2;  let b="<-forget it-/script>";  console.log("***This script tag is successfully extracted and excuted.***")  module.exports = {    components: {      compo: {        render(h) {          return h('div', {            style: {              background: 'red'            }          }, this.$slots.default);        }      }    },    data () {      return {        model: 'abc'      }    }  }</script>jjjjjjjjjjjjjjjjjjjjjj<template>  <div></div></template>```<div></div>sadfsfs大家哦哦好啊谁都发生地方上的冯绍峰s> sahhhh<compo>{{ model }}</compo>```html<compo>{{model }}{{model }}{{model }}{{model }}{{ model }}</compo>```<style src="./custom.css"></style>## 引入 style 文件<div class="custom">  原谅色</div>"
```

通过 require 里的参数，我们知道，这个 vue-markdown-loader loader 首先会加载 `.md` 文件，然后通过 `markdown-compiler.js?raw` 来处理该文件，再通过 `vue-loader` 处理。

所以我们需要继续去 `markdown-compiler.js` 里一探究竟。

### markdown-compiler.js

首先我们看一下引入声明部分：

```js
var loaderUtils = require('loader-utils');
var hljs = require('highlight.js');
var cheerio = require('cheerio');
var markdown = require('markdown-it');
var Token = require('markdown-it/lib/token');
```

再看下面的代码之前，有必要了解一下上面引入的东西

#### [markdown-it](https://www.npmjs.com/package/markdown-it)

将 markdown 转换成 html 的本尊。

有两种使用方式：render函数和传递 options

```js
// node.js, "classic" way:
var MarkdownIt = require('markdown-it'),
    md = new MarkdownIt();
var result = md.render('# markdown-it rulezz!');

// node.js, the same, but with sugar:
var md = require('markdown-it')();
var result = md.render('# markdown-it rulezz!');
```

```js
// full options list (defaults)
var md = require('markdown-it')({
  html:         false,        // Enable HTML tags in source
  xhtmlOut:     false,        // Use '/' to close single tags (<br />).
                              // This is only for full CommonMark compatibility.
  breaks:       false,        // Convert '\n' in paragraphs into <br>
  langPrefix:   'language-',  // CSS language prefix for fenced blocks. Can be
                              // useful for external highlighters.
  linkify:      false,        // Autoconvert URL-like text to links

  // Enable some language-neutral replacement + quotes beautification
  typographer:  false,

  // Double + single quotes replacement pairs, when typographer enabled,
  // and smartquotes on. Could be either a String or an Array.
  //
  // For example, you can use '«»„“' for Russian, '„“‚‘' for German,
  // and ['«\xA0', '\xA0»', '‹\xA0', '\xA0›'] for French (including nbsp).
  quotes: '“”‘’',

  // Highlighter function. Should return escaped HTML,
  // or '' if the source string is not changed and should be escaped externaly.
  // If result starts with <pre... internal wrapper is skipped.
  highlight: function (/*str, lang*/) { return ''; }
});
```