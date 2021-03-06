---
title: 怎样在vue项目下添加Eslint
date: 2017-11-15 15:13:40
tags:
---



## 简易搭建

[ESLint官网网址](https://eslint.org/)

[ESLint中文官网](http://eslint.cn/)

如果你是想在自己的项目里搭建ESLint，就可以按照官网的指示，

以全局安装举例，

``` js
npm install -g eslint
```

然后初始化

``` js
eslint --init
```

它会问你一些问题，你可以按照你的喜好进行配置，我选的是popular下面的standard，生成的文件是js格式，那么就会创建出`eslintrc.js`文件：

``` js
module.exports = {
"extends": "standard"
};
```

然后就可以简单的lint某个文件了：

``` js
$ eslint yourfile.js
```



## 在vue的项目里新添加ESLint

有的时候，早期的时候，我们建立vue项目的时候，可能图简便，并没有初始化ESLint、单元测试等等模块，那么就需要后添加进去。

如果是现在新建一个项目，通过vue-cli的问答就可以轻松初始化ESLint的配置。

这里说一下怎样在老项目里新添加ESLint。

首先，我先用vue-cli创建了一个新项目，在初始化的时候，选择安装eslint，

选择standard规则，然后就生成了`eslintrc.js`，把生成的这个文件拷贝到要加ESlint的老项目里。

``` js
// https://eslint.org/docs/user-guide/configuring
module.exports = {
  //默认情况下，ESLint 会在所有父级目录里寻找配置文件，一直到根目录。如果你想要你所有项目都遵循一个特定的约定时，这将会很有用，但有时候会导致意想不到的结果。为了将 ESLint 限制到一个特定的项目，在你项目根目录下的 package.json 文件或者 .eslintrc.* 文件里的 eslintConfig 字段下设置 "root": true。ESLint 一旦发现配置文件中有 "root": true，它就会停止在父级目录中寻找。
  root: true,
  parser: 'babel-eslint',
  parserOptions: {
    sourceType: 'module'
  },
  env: {
    browser: true,
  },
  // https://github.com/standard/standard/blob/master/docs/RULES-en.md
  extends: 'standard',
  // required to lint *.vue files
  plugins: [
    'html'
  ],
  // add your custom rules here
  'rules': {
    // allow paren-less arrow functions 要求箭头函数的参数使用圆括号
    'arrow-parens': 0,
    // allow async-await 强制 generator 函数中 * 号周围使用一致的空格
    'generator-star-spacing': 0,
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0
  }
}
```

然后找到`package.json`，把ESLint相关的依赖加进去（也可以一个一个进行安装，或者有更好的办法。。）

![img](file:///var/folders/tw/l4f5twr93wl5fcvpnrwd8nvr0000gp/T/WizNote/6ef8389b-88cf-4ee1-aa09-772bb5afb6f1/index_files/68513208.png)

``` 
    "babel-eslint": "^7.1.1",   "eslint": "^3.19.0",

    "eslint-friendly-formatter": "^3.0.0",
    "eslint-loader": "^1.7.1",
    "eslint-plugin-html": "^3.0.0",
    "eslint-config-standard": "^10.2.1",
    "eslint-plugin-promise": "^3.4.0",
    "eslint-plugin-standard": "^3.0.1",
    "eslint-plugin-import": "^2.7.0",
    "eslint-plugin-node": "^5.2.0", 
```

然后在`webpack.base.conf.js`的rules里添加

``` js
 {
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
```

再`npm install`一下，应该就可以了。

这里的编辑器推荐用vscode，可以非常智能的显示出哪里出错，方便修改。