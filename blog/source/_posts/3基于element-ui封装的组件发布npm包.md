---
title: 使用Vue CLI 3将基于element-ui二次封装的组件发布到npm
date: 2018-11-06 19:00:19
tags: ["vue", "npm"]
---

## 安装依赖
首先用[Vue CLI 3](https://cli.vuejs.org/zh/)来初始化项目
```
yarn global add @vue/cli
vue create qiyun-el-ui
vue ui
```

### 安装element-ui
这里使用官方提供的的插件安装：
http://element.eleme.io/#/zh-CN/component/quickstart#shi-yong-vue-cli-at-3

https://github.com/ElementUI/vue-cli-plugin-element

在插件列表搜索element
![image](https://user-images.githubusercontent.com/10095631/43555082-b9414998-962a-11e8-83ab-cda066a61093.png)
在这里我选的手动导入，图中是全部导入
![image](https://user-images.githubusercontent.com/10095631/43555119-f486f034-962a-11e8-9862-dcaca0e3ebb3.png)

这样在项目中，就会新建一个plugins文件夹，里面有个element.js 文件，如果想手动引入，就在这里添加要依赖的组件，这里是为了调试组件：
``` js
import Vue from 'vue'
import { 
  Button, 
  Dialog 
} from 'element-ui'

Vue.use(Button)
Vue.use(Dialog)
```

由于我们是基于`element-ui`的部分组件做的二次封装，所以最好还是按需引入所依赖的组件比较好。

## 编写组件
在 `src` 的同级下面新建 `packages` 目录，在这里添加自己封装的要发布的组件。
例如，新建 `qe-modal` 文件夹，再接着新建 `src` 文件夹，里面新建 `qe-modal.vue`，在这里写组件的代码：

``` vue
<template>
  <el-dialog
    :title="title"
    :visible="dialogVisible"
    @close="$emit('update:dialogVisible', false)"
    :width="width">
    <slot name="modal-body"></slot>

    <div slot="footer" class="dialog-footer">
      <slot name="modal-footer">
        <el-button @click="$emit('update:dialogVisible', false)" size="small">取 消</el-button>
        <el-button type="primary" @click="$emit('confirm')" size="small" :disabled="confirmDisable || beforeSendDisable">{{ beforeSendDisable? "处理中..." : "确 定" }}</el-button>
      </slot>
    </div>

  </el-dialog>
</template>

<script>
export default {
  name: 'qeModal',
  props: {
    dialogVisible: Boolean,
    title: String,
    width: {
      type: String,
      default: '580px'
    },
    beforeSendDisable: {
      type: Boolean,
      default: false
    },
    confirmDisable: {
      type: Boolean,
      default: false
    }
  }
}
</script>

```

在 `qe-modal` 根目录下新建 `index.js` ，里面注册单独的该组件，方便使用时可以单独引用：
``` js
import qeModal from './src/qe-modal'

qeModal.install = function(Vue) {
  Vue.component(qeModal.name, qeModal)
}

export default qeModal
```

这样一个组件就添加完成了，然后需要在 `packages` 的根目录下添加一个总的 `index.js`，这里是全局注册的地方，使用时可以全局引入，其实就跟 `element-ui` 的两种方式一样：
``` js
import qeModal from './qe-modal'

const components = [
  qeModal
]

const install = function(Vue) {
  components.forEach(component => {
    Vue.component(component.name, component);
  });
}

if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue);
}

export default {
  install,
  qeModal
}
```

后面再添加组件，在这里也要再注册一下，而`element-ui` 源码中是动态引入的，我们的项目组件还没那么多，可以先一个个手动引入，如果后面数量多了，不好维护，可以参考 `element-ui`  的源码实现，我在[这里](https://athena0304.github.io/element-analysis/build/bin/build-entry.js.html)做了一些简单的解释。

## 配置 npm
在 package.json 里面的 script 里面加一个 lib选项，方便每次构建：
``` js
"scripts": {
    // ...,
    "lib": "vue-cli-service build --target lib --name qiyun-el-ui --dest lib ./packages/index.js"
  },
```
其中 `--name` 后面是你最后想要生成文件的名字，并用 `--dest lib` 修改了构建的目录。
然后在 `package.json` 里面添加一些npm包发布的相关信息，比如作者、版本等：
其中最重要的是：

```
"main": "lib/qiyun-el-ui.common.js",
```
这里的路径要和上面构建出来的目录和文件名对应上。

里面的配置项，在网上找了个例子：
``` js
{
  "name": "maucash",
  "description": "maucash中常用组件抽取",
  "version": "1.0.2",
  "author": "kuangshp <kuangshp@126.com>",
  // 开源协议
  "license": "MIT",
  // 因为组件包是公用的，所以private为false
  "private": false,
  // 配置main结点，如果不配置，我们在其他项目中就不用import XX from '包名'来引用了，只能以包名作为起点来指定相对的路径
  "main": "dist/maucash.js",
  "scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot",
    "build": "cross-env NODE_ENV=production webpack --progress --hide-modules"
  },
  "dependencies": {
    "axios": "^0.18.0",
    "iview": "^2.14.1",
    "style-loader": "^0.23.1",
    "url-loader": "^1.1.2",
    "vue": "^2.5.11"
  },
  // 指定代码所在的仓库地址
  "repository": {
    "type": "git",
    "url": "git+git@git.wolaidai.com:maucash/maucash.git"
  },
  // 指定打包后,包中存在的文件夹
  "files": [
    "dist",
    "src"
  ],
  // 指定关键词
  "keywords": [
    "vue",
    "maucash",
    "code",
    "maucash code"
  ],
  // 项目官网的地址
  "homepage": "https://github.com/kuangshp/maucash",
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not ie <= 8"
  ],
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-preset-env": "^1.6.0",
    "babel-preset-stage-3": "^6.24.1",
    "cross-env": "^5.0.5",
    "css-loader": "^0.28.7",
    "file-loader": "^1.1.4",
    "node-sass": "^4.5.3",
    "sass-loader": "^6.0.6",
    "vue-loader": "^13.0.5",
    "vue-template-compiler": "^2.4.4",
    "webpack": "^3.6.0",
    "webpack-dev-server": "^2.9.1"
  }
}

```

## 发布到npm

到这块后面的网上有很多更细致的教程，我就不在这里赘述了。下面给出两个文章的链接，供参考。

1、到[npm](https://www.npmjs.com/)上注册一个账号

2、登录
`npm login`

3、添加用户信息
`npm adduser`

4、发布到远程仓库(npm)上
`npm publish`

5、删除远程仓库的包
`npx force-unpublish package-name '原因描述'`

参考：
https://juejin.im/post/5bc44175f265da0a906f9869
[Vue cli3 库模式搭建组件库并发布到 npm的流程_vue.js_脚本之家](https://www.jb51.net/article/148692.htm)