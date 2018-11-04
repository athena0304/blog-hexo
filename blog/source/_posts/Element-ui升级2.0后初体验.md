---
title: Element-ui升级2.0后初体验
---

如果你在使用Vue和Element-ui进行前端开发，那么就会知道Element在2017年10月27日升级到了2.0.0 Carbon的版本，改动可以说还是有些大的。可想而知，有的人选择按兵不动，有的人选择先睹为快。

现在打开Element-ui的官网，切换到1.4.12的文档，可以看到一个提示消息框：

> Element 1.x 已停止维护，请升级至 2.x

所以还是建议尽早升级比较好，顺便也可以一起升级一下vue到2.5版本以上。下面我会介绍一些我在升级项目的过程中遇到的问题和需要注意的地方。

升级后的版本：
element-ui：2.0.4（最新版本已经到2.0.7）
vue：2.5.4

首先说一下整体印象，由于我做的这个项目是管理控制台类型的，升级后第一印象是页面变“白”了，也可以说变淡了，尤其是侧边栏、导航菜单，以前是可以选择深色的主题，现在没有这个`theme-default`预设了，新增了新的主题`chalk`，导致整个页面都是白白的。个人理解以后官方可能会出别的系列的主题，这样方便拆解和设计。
表格的表头原来也是有背景颜色区分的，现在的chalk主题下表头的颜色区分不是很明显。然后之前用的tag、按钮之类的都变大了，这是在什么都没设置，直接升级以后的第一眼效果。

然后下面说一下我遇到的需要修改的地方。

#### 1. 引入的css文件路径变了
原来是：
`import 'element-ui/lib/theme-default/index.css'`
现在需要修改成：
`import 'element-ui/lib/theme-chalk/index.css'`

#### 2. 样式引入的优先级问题

之前遇到这样一个问题，有的组件样式需要进行定制覆盖，于是就在组件里面用css scoped进行了同类名的样式替换，这样在开发环境下效果是符合预期的，但是打包编译后，优先级就变了。于是发现是在`main.js`引入文件路径顺序的问题，之前一度以为不需要顺序，但其实还是有影响的。

原来的配置：

``` js
import Vue from 'vue'
import App from './App.vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.use(ElementUI)
```

修改后的：

``` js
import Vue from 'vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
import App from './App.vue' //这里要把App的引用顺序放到最后

Vue.use(ElementUI)
```

#### 3. Icon的变化

原来在`input`组件中可以用`icon`这个属性指定icon，例如

```
<el-input
  icon="search">
</el-input>
```

现在的话，这样写是不生效的：
> 可以通过`prefix-icon` 和 `suffix-icon` 属性在 input 组件首部和尾部增加显示图标，也可以通过 slot 来放置图标。

所以如果你在项目中的`input`里用到了`icon`的属性，需要改成`prefix-icon` 或 `suffix-icon`：

```
<el-input
	suffix-icon="el-icon-search">
</el-input>
```

但是button组件就还是可以使用`icon`属性，但需要传入完整的图标类名：
```
<el-button type="primary" icon="el-icon-search">搜索</el-button>
```
原因是：

> 为了方便使用第三方图标，Button 的 `icon` 属性、Input 的 `prefix-icon` 和 `suffix-icon` 属性、Steps 的 `icon` 属性现在需要传入完整的图标类名

#### 4. modal的变化

之前给不需要宽度的modal设了`width: auto;`，这样如果里面内容为空的时候基本没有宽度，更新后，如果没有内容，默认会铺满整个屏幕。

#### 5. NavMenu 导航菜单
原来的menu是有两个样式供选择的，`theme`有两个可选值`light, dark`，现在没有这个属性了，默认就是chalk主题的白色，如果想要定制，需要另外设置。

| 参数                | 说明                      | 类型     | 可选值  | 默认值     |
| ----------------- | ----------------------- | ------ | ---- | ------- |
| background-color  | 菜单的背景色（仅支持 hex 格式）      | string | —    | #ffffff |
| text-color        | 菜单的文字颜色（仅支持 hex 格式）     | string | —    | #409EFF |
| active-text-color | 当前激活菜单的文字颜色（仅支持 hex 格式） | string | —    | #409EFF |

但是这样设置会有弊端，如果项目经过定制样式改过主题颜色，那么这里就需要进行单独设置，而且还仅支持 hex 格式，这就需要计算出来颜色的具体值，而不能通过scss变量来控制。

#### 6. slot-scope
这其实是[vue在2.5.0里的变化][1]
把`scope`换成了`slop-scope`
所以在element里面升级后，也把相应的用到scope的地方做修改就行了

#### 7. checkbox
在更新后测试的时候发现checkbox挂了，于是查看，发现Checkbox Events的`change`事件的参数变了：

1.0:

![clipboard.png](https://segmentfault.com/img/bVY0CM?w=854&h=167)

2.0:

![clipboard.png](https://segmentfault.com/img/bVY0CQ?w=905&h=185)

所以，原来1.0判断change函数里面是这么写的:

```
    handleCheckAllChange(event) {
        this.checkedCities = event.target.checked ? cityOptions : [];
        this.isIndeterminate = false;
      },
```
2.0是这样的：

```
    handleCheckAllChange(val) {
        this.checkedCities = val ? cityOptions : [];
        this.isIndeterminate = false;
      },
```



目前就是这么多，随着项目深入，可能还会遇到更多的问题和兼容版本的问题，我们可以共同探讨。