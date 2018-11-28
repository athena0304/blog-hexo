---
title: vue-markdown-loader源码解析
date: 2018-11-16 11:48:15
tags:
---
项目中遇到了需要单独加载某个 markdown 文件显示在页面中，类似于操作指引的感觉，于是找到了 vue-markdown-loader 这个工具，觉得很好用，于是我打算开个专题看一下里面都做了些什么，有助于对 webpack loader 的理解。

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

结构很清晰，东西也不算太多，涉及到的源码就是 `index.js`和 `lib` 下的两个 js 文件，`example` 里面的是示例。

那我们就先从 `index.js` 开始看起吧。

就一句话：

```js
module.exports = require('./lib/core');
```

这个文件是你引入这个包的入口，这里直接去找的 `./lib/core`，于是我们继续去 `./lib/core` 看看。

### core.js

在声明阶段：

```js
var path = require('path');
var loaderUtils = require('loader-utils');

var markdownCompilerPath = path.resolve(__dirname, 'markdown-compiler.js');
```

这里引用了 [loader-utils](https://www.npmjs.com/package/loader-utils) 和同级目录下的 `markdown-compiler.js`

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

#### [highlight.js](https://highlightjs.org/)

语法高亮工具

#### [cheerio](https://www.npmjs.com/package/cheerio)

Fast, flexible & lean implementation of core jQuery designed specifically for the server.

接下来，我们先看几个声明的函数：

#### addVuePreviewAttr

```js
/**
 * `<pre></pre>` => `<pre v-pre></pre>`
 * `<code></code>` => `<code v-pre></code>`
 * @param  {string} str
 * @return {string}
 */
var addVuePreviewAttr = function(str) {
  return str.replace(/(<pre|<code)/g, '$1 v-pre');
};
```

这个函数就是查找 html 标签中所有 `<pre` 或者 `<code`，替换成  `<pre v-pre ` 和 `<code v-pre`

#### renderHighlight

```js
/**
 * renderHighlight
 * @param  {string} str
 * @param  {string} lang
 */
var renderHighlight = function(str, lang) {
  if (!(lang && hljs.getLanguage(lang))) {
    return '';
  }

  return hljs.highlight(lang, str, true).value;
};
```

返回通过 highlight.js 高亮后的数据

#### renderVueTemplate

```js
/**
 * html => vue file template
 * @param  {[type]} html [description]
 * @return {[type]}      [description]
 */
var renderVueTemplate = function(html, wrapper) {
  // 用 cheerio 提取参数传进来的要进行处理的 html
  var $ = cheerio.load(html, {
    decodeEntities: false, // 是否解码文档实体，默认为 false
    lowerCaseAttributeNames: false, // 是否将所有属性名设置成小写，会对速度有影响，默认为 false
    lowerCaseTags: false // 是否将所有标签转换成小写
  });
  // 将原 html 中的 style 和第一个 script 标签缓存起来
  var output = {
    style: $.html('style'),
    // get only the first script child. Causes issues if multiple script files in page.
    script: $.html($('script').first())
  };
  var result;

  $('style').remove();
  $('script').remove();
  // 生成最后的结果
  result =
    `<template><${wrapper}>` +
    $.html() +
    `</${wrapper}></template>\n` +
    output.style +
    '\n' +
    output.script;

  return result;
};
```

这是整个 loader 的核心功能，就是把 html 包裹一层 vue 的语法变成一个 vue 的组件，然后再让后面的 vue-loader 接收。这里用到了 cheerio 来做一些简单的 DOM 操作。

说完了函数声明，就继续来看整个处理过程：

```js
module.exports = function(source) {
  // (1) 是否可缓存
  this.cacheable && this.cacheable();
  var parser, preprocess;
  // （2）获取参数，此时把外面传进来的解析成 Object {raw: true}(来自core.js)
  var params = loaderUtils.getOptions(this) || {};
  // （3）获取 __vueMarkdownOptions__(来自core.js)
  var vueMarkdownOptions = this._compilation.__vueMarkdownOptions__;
  // （4）继承 vueMarkdownOptions 的原型，赋值给opts
  var opts = vueMarkdownOptions ? Object.create(vueMarkdownOptions.__proto__) : {}; // inherit prototype
  var preventExtract = false;
  // （5）合并所有来源的参数、属性，汇总给opts
  opts = Object.assign(opts, params, vueMarkdownOptions); // assign attributes
  // （6）判断 options 中 preventExtract 是都为 true
  if (opts.preventExtract) {
    delete opts.preventExtract;
    preventExtract = true;
  }
  // （7）判断 options 中 render 的类型
  if (typeof opts.render === 'function') {
    // （8）如果是 function，parser 就是 opts
    parser = opts;
  } else {
    // （9）如果不是 function，为 opts 添加一些属性，以及后面的一系列操作。opts 最后是作为 option 传入 markdown-it 中的。
    opts = Object.assign(
      {
        preset: 'default',
        html: true,
        highlight: renderHighlight,
        wrapper: 'section'
      },
      opts
    );
    // （10）初始化 插件 plugins
    var plugins = opts.use;
    // （11）初始化 预处理 preprocess
    preprocess = opts.preprocess;

    delete opts.use;
    delete opts.preprocess;
      
    // （12）在这里初始化 markdown-it
    parser = markdown(opts.preset, opts);

    // （13）添加 ruler：从 html token 中提取 script 和 style
    //add ruler:extract script and style tags from html token content
    !preventExtract &&
      parser.core.ruler.push('extract_script_or_style', function replace(
        state
      ) {
        let tag_reg = new RegExp('<(script|style)(?:[^<]|<)+</\\1>', 'g');
        let newTokens = [];
        state.tokens
          .filter(token => token.type == 'fence' && token.info == 'html')
          .forEach(token => {
            let tokens = (token.content.match(tag_reg) || []).map(content => {
              let t = new Token('html_block', '', 0);
              t.content = content;
              return t;
            });
            if (tokens.length > 0) {
              newTokens.push.apply(newTokens, tokens);
            }
          });
        state.tokens.push.apply(state.tokens, newTokens);
      });
    // （14）如果有插件，就应用一下
    if (plugins) {
      plugins.forEach(function(plugin) {
        if (Array.isArray(plugin)) {
          parser.use.apply(parser, plugin);
        } else {
          parser.use(plugin);
        }
      });
    }
  }

  // （15）覆盖默认的 parser rules，在 'code' 和 'pre' 标签上添加 v-pre 属性
  /**
   * override default parser rules by adding v-pre attribute on 'code' and 'pre' tags
   * @param {Array<string>} rules rules to override
   */
  function overrideParserRules(rules) {
    if (parser && parser.renderer && parser.renderer.rules) {
      var parserRules = parser.renderer.rules;
      rules.forEach(function(rule) {
        if (parserRules && parserRules[rule]) {
          var defaultRule = parserRules[rule];
          parserRules[rule] = function() {
            return addVuePreviewAttr(defaultRule.apply(this, arguments));
          };
        }
      });
    }
  }
  // （16）覆盖这三种默认规则
  overrideParserRules(['code_inline', 'code_block', 'fence']);

  // （17）如果有预处理，执行一下
  if (preprocess) {
    source = preprocess.call(this, parser, source);
  }

  // （18）将 source 中所有的 @ 替换成 '__at__'
  source = source.replace(/@/g, '__at__');

  var content = parser.render(source).replace(/__at__/g, '@');
  var result = renderVueTemplate(content, opts.wrapper);

  if (opts.raw) {
    return result;
  } else {
    return 'module.exports = ' + JSON.stringify(result);
  }
};
```

![image-20181121120525127](/Users/athena/Library/Application Support/typora-user-images/image-20181121120525127.png)

**（6）preventExtract**

preventExtract 是 vue-markdown-loader 提供的一个[选项](https://www.npmjs.com/package/vue-markdown-loader?activeTab=readme#preventextract)

> Since `v2.0.0`, this loader will automatically extract script and style tags from html token content (#26). If you do not need, you can set this option

**（13）添加 ruler：从 html token 中提取 script 和 style**

走到这里，就需要对 token 有一个认识才行，所以建议先看一下[这篇文章](https://www.jianshu.com/p/fb0ee355915c)。我摘录一部分：

> 当你创建了一个 `md = require('markdown-it')()` 对象之后，就可以用它来渲染 MD 文档了，例如: `md.render("# I'm H1 ")`。这个渲染过程分为主要的两步:
>
> 1. 将 MD 文档 Parsing 为 Tokens。
> 2. 渲染这个 Tokens。
>
> Parsing 的过程是，首先创建一个 Core Parser，这个 Core Parser 包含一系列的缺省 Rules。这些 Rules 将顺序执行，每个 Rule 都在前面的 Tokens 的基础上，要么修改原来的 Token，要么添加新的 Token。这个 Rules 的链条被称为 Core Chain。
>
> 在所有 Tokens 都获得之后，就可以渲染了。渲染就是把特定 Token 转变为特定的 HTML 的过程。
>
> Markdown-It 允许你为特定的 Token Type 挂载自己的渲染函数，这个函数称为 Renderer Rule。Markdown-It 已经定义了几个 缺省的 Renderer Rules

**（14）如果有插件，就应用一下**

[MarkdownIt.use](https://markdown-it.github.io/markdown-it/#MarkdownIt.use)

在当前的解析实例中应用指定的插件。

最终输出的结果如下：

```
"<template><section><h1>Hello</h1><p><code v-pre="">&lt;span&gt;{{sss}}&lt;/span&gt;</code></p><blockquote><p>This is test.</p></blockquote><ul><li>How are you?</li><li>Fine, Thank you, and you?</li><li>I'm fine， too. Thank you.</li><li>🌚</li></ul><pre v-pre=""><code v-pre="" class="language-javascript"><span class="hljs-keyword">import</span> Vue <span class="hljs-keyword">from</span> <span class="hljs-string">'vue'</span>Vue.config.debug = <span class="hljs-literal">true</span></code></pre><div class="test">  {{ model }} test</div><p><compo>{{ model }}</compo></p><div class="abc" @click="show = false">  啊哈哈哈</div><blockquote><p>All script or style tags in html mark will be extracted.Script will be excuted, and style will be added to document head.Notice if there is a string instance which contains special word &quot;&lt;/script&gt;&quot;, it will fetch a SyntaxError.Due to the complexity to solve it, just don't do that.</p></blockquote><pre v-pre=""><code v-pre="" class="language-html"><span class="hljs-tag">&lt;<span class="hljs-name">style</span> <span class="hljs-attr">scoped</span>&gt;</span><span class="css">  <span class="hljs-selector-class">.test</span> {    <span class="hljs-attribute">background-color</span>: green;  }</span><span class="hljs-tag">&lt;/<span class="hljs-name">style</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">style</span> <span class="hljs-attr">scoped</span>&gt;</span><span class="css">  <span class="hljs-selector-class">.abc</span> {    <span class="hljs-attribute">background-color</span>: yellow;  }</span><span class="hljs-tag">&lt;/<span class="hljs-name">style</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">  <span class="hljs-keyword">let</span> a=<span class="hljs-number">1</span>&lt;<span class="hljs-number">2</span>;  <span class="hljs-keyword">let</span> b=<span class="hljs-string">"&lt;-forget it-/script&gt;"</span>;  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"***This script tag is successfully extracted and excuted.***"</span>)  <span class="hljs-built_in">module</span>.exports = {    <span class="hljs-attr">components</span>: {      <span class="hljs-attr">compo</span>: {        render(h) {          <span class="hljs-keyword">return</span> h(<span class="hljs-string">'div'</span>, {            <span class="hljs-attr">style</span>: {              <span class="hljs-attr">background</span>: <span class="hljs-string">'red'</span>            }          }, <span class="hljs-keyword">this</span>.$slots.default);        }      }    },    data () {      <span class="hljs-keyword">return</span> {        <span class="hljs-attr">model</span>: <span class="hljs-string">'abc'</span>      }    }  }</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>jjjjjjjjjjjjjjjjjjjjjj<span class="hljs-tag">&lt;<span class="hljs-name">template</span>&gt;</span>  <span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">template</span>&gt;</span></code></pre><div></div><p>sadfsfs</p><p>大家哦哦好啊谁都发生地方上的冯绍峰s</p><blockquote><p>sahhhh</p></blockquote><p><compo>{{ model }}</compo></p><pre v-pre=""><code v-pre="" class="language-html"><span class="hljs-tag">&lt;<span class="hljs-name">compo</span>&gt;</span>{{model }}{{model }}{{model }}{{model }}{{ model }}<span class="hljs-tag">&lt;/<span class="hljs-name">compo</span>&gt;</span></code></pre><h2>引入 style 文件</h2><div class="custom">  原谅色</div></section></template><style src="./custom.css"></style><style scoped>  .test {    background-color: green;  }</style><style scoped>  .abc {    background-color: yellow;  }</style><script>  let a=1<2;  let b="<-forget it-/script>";  console.log("***This script tag is successfully extracted and excuted.***")  module.exports = {    components: {      compo: {        render(h) {          return h('div', {            style: {              background: 'red'            }          }, this.$slots.default);        }      }    },    data () {      return {        model: 'abc'      }    }  }</script>"
```

如果中间哪步不太明白，也可以自行断点调试。总的思路还是很清晰的，就是把 `.md` 文件通过 `markdown-it` 转成 `html`，中间通过选项设置高亮，然后再包裹上 vue 组件的语法形式即可，后续再应用 `vue-loader` 做后面的处理。

