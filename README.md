# 小程序技术分享

## 小程序框架

### 配置

文档参考： https://mp.weixin.qq.com/debug/wxadoc/dev/framework/config.html

### 页面

小程序的一个页面由4个文件组成: `js, wxml, wxss, json`。其中的`js`是逻辑层所在，`wxml, wxss`是视图层
所在。`wxss`是样式，`json`是该页面的配置，用来自定义覆盖全局的一些配置。

小程序找一个页面的时候，比如要找一个名字为`index`的页面，它会根据配置文件中的配置，找到这个
页面代码文件的所在，然后去匹配 `index.{js, wxml, wxss, json}` 等文件，使用这些文件来渲染
这个页面。

### 路由

文档参考：https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/route.html

这里简单说下路由的路径，小程序的路由路径不是配置的，而是根据页面路径来的。比如一个页面的名字是`index`，根据全局的`app.json`的配置，页面在`/pages/index/index.{js, wxml, wxss, json}`，
因此如果你要链接到这个页面（显示这个页面），你可以通过小程序的api来跳转到这个页面：`wx.navigateTo({ url: '/pages/index/index' })`，同时，你可以给这个url提供一些query参数。


### 数据绑定

数据绑定由两部分组成：视图层(view)和逻辑层(App service)。逻辑层提供数据和事件处理函数给视图层，
供视图层的渲染和事件绑定。视图层返回给逻辑层是一些事件，逻辑层根据这些事件做数据的修改或者其他操作，
控制视图的更新和页面的操作。所以这里的数据绑定是单向的，总是逻辑层流向视图层。

要触发数据的更新并通知视图层，需要调用页面的 `this.setData` 方法。要访问这些数据，可以通过
`this.data.myData` 来访问。直接通过`this.data.myData =  'value'`来修改数据是不会造成视图
更改的。

示例：

```js
Page({
	data: {
		myData: 3,
	},

	onClick(e) {
		this.setData({
			myData: 4,
		});

		console.log(this.data.myData);
	}
});
```

## 样式

小程序的样式文件是以`wxss`结尾的，样式语法和css基本差不多。在小程序里，一般的页面样式可以放在和某个
页面对应的样式文件中，如果有一些共用的样式，可以放在一个单独的文件中，比如`common.wxss`，然后要用到
某个共用的样式，你需要在页面样式文件中通过`import`语法来将共用样式引进来。

### rpx单位

如果要考虑设备适配，可以在样式文件中使用`rpx`单位，rpx是相对于设备宽度的，计算公式如下（来自官方文档）：

|设备          |  rpx换算px(屏幕宽度/750)  | px换算rpx (750/屏幕宽度) |
|--------------|------------------------|------------------------|
| iPhone5      | 1rpx = 0.42px  | 1px = 2.34rpx |
| iPhone6      | 1rpx = 0.5px   | 1px = 2rpx    |
| iPhone6 Plus | 1rpx = 0.552px | 1px = 1.81rpx |

所以假设视觉稿是以iPhone6为基准的，那么我们要处理的是宽度为`375px`的iPhone6设备，所以屏幕宽度
是`375`。然后我们在视觉稿中测出某个button的宽度为`20px`，则其rpx的值是 `(20 * 750 / 375)px = 40rpx`。

因为在retina屏幕上，iPhone6的设计稿中的设备宽是750px，测量的长度也都是x2倍的。所以如果以
iPhone6为标准设计小程序，可以不用计算直接使用rpx单位，对开发人员很友好。

## 元素

以下只列出了我在这个项目中用到的元素。

1. `view`: 类似于`div`，用来布局。
2. `text`: 用来表示一段文本，和`p`元素不同，小程序中的`text`是行内元素，margin和padding只在横向上有用. `text`元素中只能包含`text`元素，其他的会不显示。
3. `image`: 类似于`img`, 有一个`mode`标签属性，来规定图片填充显示的样式，有aspectFit, aspectFill等值。类似于背景图样式background-size中的cover等。
4. `page`: page可以看作是`body`元素，一般你不会写这个标签。但是如果要对全局的样式做控制，可以将样式
写到这个标签下面，就和样式写在`body`下面一样的。(我是这么做的)
5. `button`: 小程序的按钮组件，默认的button样式`margin-left`和`margin-right`的值是`auto`，所以在布局中左右会有一些间距。比如使用flexbox编排一些按钮布局的时候，当`justify-content: space-between`时，最好将默认按钮的`margin`设置为`0`，让flexbox来处理这些按钮元素之间的间距。
6. `navigator`: 相当于一个链接元素a。

### 不是元素的元素

1. `block` 是一个包装元素，不会被渲染。它的作用是，在做`if`判断或者`for`渲染列表的时候，有时候
你不希望将`if`或者`for`写在一个真正的元素上，你可以使用block来包含一组元素。


## 其他

1. 似乎一些HTML字符实体不能在小程序的模板中使用, 比如我想用 `&lt;`来表示 `<`，但是不管用，如果直接用
`<`的话肯定会报错。我目前的解决办法是使用模板语法 `{{ "<<" }}` 来生成一个字符串的符号。


## 对于设计师的建议和要求？

如果公司刚开始开发，而设计师又对小程序不熟悉的话，开发最好和设计师对于小程序应用的设计做一个沟通。

1. 视觉稿以iPhone6为标准，2x的尺寸，即视觉稿上的设备宽度为750px。
2. 组件样式参考小程序官网上的组件样式，目前组件样式似乎可定制的不多。
3. 待续...
