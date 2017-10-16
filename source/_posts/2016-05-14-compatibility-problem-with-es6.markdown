---
layout: post
title: "使用 ES6 的浏览器兼容性问题"
date: 2016-05-14 16:35:34 +0800
comments: true
categories:
---

以前对浏览器兼容性问题只是大概知道一些点，没想到这次真正着手去做的时候，还是碰到了很多问题。刚开始的时候一边解决问题，一边想着：用 IE8 的都是神经病，到后来，我发现完了，I LOVE IE。

## 0x00 起源

在这次做小蜜 PC 版的时候，由于早于 PC 版，无线版已经重新设计了全新版，做了很多架构上的优化调整。所以在做的时候把无线版的前端架构拿了过来，主要的考虑就是品牌和功能保持跟无线版统一的同时，技术上也可相互支持以及组件复用。

无线版整个架构设计是同事做的，技术上主要采用 ES6 + Webpack + Babel 的方式，由于项目的独特性和特殊需求，并没有使用任何框架，只引入 zepto 作为一个标准支撑库。

而 PC 版的架构跟无线版基本保持一致，主要是把 zepto 换成了 jQuery。

下面是一些基本的开发依赖：

```json
{
  "devDependencies": {
    "babel-core": "~6.3.15",
    "babel-loader": "~6.2.0",
    "babel-preset-es2015": "~6.3.13",
    "babel-preset-stage-0": "~6.3.13",
    "babel-runtime": "~6.3.13",
    "extract-text-webpack-plugin": "~0.9.1",
    "less-loader": "~2.2.1",
    "nunjucks-loader": "~1.0.7",
    "style-loader": "~0.10.2",
    "webpack": "~1.12.9",
    "webpack-dev-server": "^1.10.1"
  }
}
```

## 0x01 polyfill

由于 Babel 默认只转换转各种 ES2015 语法，而不转换新的 API，比如 Promise，以及 Object.assign、Array.from 这些新方法，这时我们需要提供一些 ployfill 来模拟出这样一个提供原生支持功能的浏览器环境。

主要有两种方式：`babel-runtime` 和 `babel-polyfill`。

### babel-runtime

babel-runtime 的作用是模拟 ES2015 环境，包含各种分散的 polyfill 模块，我们可以在自己的模块里单独引入，比如 promise：

```js
import 'babel-runtime/core-js/promise'
```

它们不会在全局环境添加未实现的方法，只是这样手动引用每个 polyfill 会非常低效，我们可以借助 `Runtime transform` 插件来自动化处理这一切。

首先使用 npm 安装：

```sh
npm install babel-plugin-transform-runtime --save-dev
```

然后在 webpack 配置文件的 babel-loader 增加选项：

```js
loader: ["babel-loader"],
query: {
  plugins: [
	"transform-runtime"
  ],
  presets: ['es2015', 'stage-0']
}
```

### babel-polyfill

而 `babel-polyfill` 是针对全局环境的，引入它浏览器就好像具备了规范里定义的完整的特性，一旦引入，就会跑一个 `babel-polyfill` 实例。用法如下：

1.安装 babel-polyfill

```sh
npm install babel-polyfill --save
```

2.在入口文件中引用：

```js
import 'babel-polyfill'
```

### 小结：

其实做到这些，在大部分浏览器就可以正常跑了，但我们做的是一个用户环境很不确定的产品，对一些年代久远但又不容忽视的运行环境，比如 IE8，我们做的还不够。

接下来将开始讲述我们在兼容性方面遇到的一些问题，和解决方法。


## 0x02 开始在 IE8 运行

最开始做的时候并没有针对 IE 做一些兼容性方面的处理，结果在 IE8 上一跑一堆问题。

第一步，我们把 `jQuery` 换成 1.12.1 ，因为 2.X 已经不再支持 IE8。

但并没有像我们想象中的那样，只是简单换一下 `jQuery` 版本就可以正常运行了。


## 0x03 default or catch

这是遇到的第一个问题。在兼容性测试过程中，对下面的代码：

```js
function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
```

或者这种：

```js
module.exports = _main2.default;
```

在 IE8 下会直接报”缺少标识符、字符串或数字”的错。

我们得在对象的属性上加 `''` 才可以。就像下面这样：

```js
function _interopRequireDefault(obj) {
  return obj && obj.__esModule ? obj : { ‘default’: obj };
}

module.exports = _main2['default'];
```

至于原因，并不是 IE8 下对象的属性必须得加 `''` 才行，而是 `default` 的问题，作为一个关键字，同样的问题还包括 `catch`。

这两种情况，可以通过使用 `transform-es3-property-literals` 和 `transform-es3-member-expression-literals` 这两个插件搞定。

总之，在平时写代码的时候避免使用关键字，或者保留字作为对象的属性值，尤其是在习惯不加引号的情况下。相关讨论：[Allow reserved words for properties](https://github.com/airbnb/javascript/issues/61)



## 0x04 es5-shim、es5-sham

为了兼容像 IE8 这样的老版本浏览器，我们引入 `es5-shim` 作为 polyfill。

但在遇到 `Object.defineProperty` 仍提示 "对象不支持此操作"

> As currently implemented, the Object.defineProperty shim will not install on IE8 because IE8 already has such a method. However, the built-in IE8 method only works when applied to DOM objects.

其实 es5-shim 明确说明，这个方法的 polyfill 在 IE8 会失败，因为 IE8 已经有个同名的方法，但只是用于 DOM 对象。

同样的问题还包括 `Object.create`，上述问题可以再引入 es5-sham 解决.

## 0x05 addEventListener

项目中有部分代码直接使用 `addEventListener` 这个 API，但在 IE8 下的事件绑定并不是这个方法。

这个问题很容易解决，也无需去写额外的 polyfill。我们已经把 jQuery 换成 1.x，所以只需把代码中 `addEventListener` 换成 `jQuery` 的写法就 Okay 了。

`jQuery` 其实为我们封装了很多 API，并做了很多兼容性的封装，类似的只要使用封装好的就可以了。

## 0x06 无法获取未定义或 null 引用的属性

这个问题是在特定场景下【转人工】出现的，出现问题的不是 IE8，而是 IE9 和 IE10。

原因是 ocs 实例创建失败，因为没有调用父类的构造函数。

通过安装 `transform-es2015-classes` 和 `transform-proto-to-assign` 解决。

在配置项加上这两个插件的配置：

```js
{
  "plugins": [
      ["transform-es2015-classes", { "loose": true }],
      "transform-proto-to-assign"

  ]
}
```

## 0x07 postMessage

虽然 `postMessage` 是 HTML5 的特性，但 IE8 和 Firefox3 很早就实现了这个 API，当然，跟后来的标准并不一致。这其实也不能怪 IE8。

> The postMessage method is supported in Internet Explorer from version 8, Firefox from version 3 and Opera from version 9.5.

我们可能会这样去使用：

```js
parent.postMessage({success: 'ok', name: ‘mirreal’}, ‘*’);
```

但是为了兼容 IE8，我们得转成字符串：

```js
parent.postMessage(JSON.stringify({success: 'ok', name: "mirreal"}), ‘*’);
```

另外一个需要注意的点是：在 IE8 下 `window.postMessage` 是同步的。

> window.postMessage is syncronouse in IE 8


```js
var syncronouse = true;
window.onmessage = function () {
  console.log(syncronouse); // 在 IE8 下会在控制台打印 true
};
window.postMessage('test', '*');
syncronouse = false;
```

## 0x08 IE8/IE9 的控制台

遇到一个奇怪的问题，在刚开始遇到的时候（其实搞清楚原因，好像也挺正常的），小蜜在 IE8 IE9 无法加载。在 IE8 那个古老浏览器的左下角，好像也是唯一会在页面提示脚本错误的浏览器，提示 `script error`。

第一反应就是应该又是某个函数在 IE 下不支持，准备打开控制台看看到底哪里报错，结果却什么事都没有了，页面竟然顺畅地加载出来了，这下该怎么调试好呢？

开始思考：什么东西是依赖控制台而存在的，到底会是什么呢。。。其实就是控制台本身。

原因就是我们在代码中添加了一些控制信息会打印在控制台，而 IE8/IE9 要开启 IE Dev Tools 才能使用 `console` 对象。

切忌把 IE8/9 想成 Chrome/Firefox，以为永远有 `window.console` 可用.终于，IE10 改邪归正，`console` 不再像段誉的六脉神剑时有时无。

> console.log is there in IE8, but the console object isn't created until you open DevTools. Therefore, a call to console.log may result in an error, for example if it occurs on page load before you have a chance to open the dev tools.


但只要 IE8/9 还在一天，console 检查还是不能少的

事实上，IE8/9 从未死去，所以，得写成这样：

```js

if (window.console) {
  console.log('log here');
}
```

要是有一堆 `console.log`, `console.count`, `console.error`, `console.time`, `console.profile`，... 这样去写，那还不把人写到恶心死。

写个简单的 console polyfill 吧，检测是否存在 `console`，不存在可以创见一个同名的空方法达到不报错的目的。当然，生产环境的代码其实也不会有那么多奇奇怪怪的 `console`。

## 0x09 定义文档兼容性

`X-UA-Compatible` 当初是针对 IE8 新加的一个配置。用于为 IE8 指定不同的页面渲染模式，比如使用 IE7 兼容模式，或者是采用最新的引擎。

现在基本也不需要前者的降级模式，更多的是写入 `IE=edge` 支持最新特性。而 `chrome=1` 则会激活 Google Chrome Frame，前提是你的 IE 安装过这个插件。

有什么用呢，当然有用，有些 API 是作为新特性存在于 IE8 中的，比如 `JSON`，不开启的话就用不了。


### 为什么要用 X-UA-Compatible？

在 IE8 刚推出的时候，很多网页由于重构的问题，无法适应较高级的浏览器，所以使用 `X-UA-Compatible` 强制 IE8 采用低版本方式渲染。

比如：使用下面这段代码后，开发者无需考虑网页是否兼容 IE8 浏览器，只要确保网页在 IE6、IE7 下的表现就可以了。

```html
<meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7" />
```

而这段代码：

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
```

`IE=edge` 告诉 IE 使用最新的引擎渲染网页，`chrome=1` 则可以激活 Chrome Frame[1]。

## 0x0a 条件注释 or 条件编译

最后说说 IE 的条件注释，用法如下：

```text
!	[if !IE]	The NOT operator. This is placed immediately in front of the feature, operator, or subexpression to reverse the Boolean meaning of the expression.

lt	[if lt IE 5.5]	The less-than operator. Returns true if the first argument is less than the second argument.

lte	[if lte IE 6]	The less-than or equal operator. Returns true if the first argument is less than or equal to the second argument.

gt	[if gt IE 5]	The greater-than operator. Returns true if the first argument is greater than the second argument.

gte	[if gte IE 7]	The greater-than or equal operator. Returns true if the first argument is greater than or equal to the second argument.

( )	[if !(IE 7)]	Subexpression operators. Used in conjunction with boolean operators to create more complex expressions.

&	[if (gt IE 5)&(lt IE 7)]	The AND operator. Returns true if all subexpressions evaluate to true

|	[if (IE 6)|(IE 7)]	The OR operator. Returns true if any of the subexpressions evaluates to true.

```

另外一个类似的东西是在 Javascript 中的条件编译（conditional compilation）。我们可以使用这段简单的代码来做浏览器嗅探：

```js
var isIE = /*@cc_on!@*/false
```

在其他浏览器中，false 前的被视为注释，而在 IE 中，`/*@cc_on .... @*/` 之间的部分可以被 IE 识别并作为程序执行，同时启用 IE 的条件编译。

常用变量如下：

```text
* @_win32 如果在 Win32 系统上运行，则为 true。
* @_win16 如果在 Win16 系统上运行，则为 true。
* @_mac 如果在 Apple Macintosh 系统上运行，则为 true。
* @_alpha 如果在 DEC Alpha 处理器上运行，则为 true。
* @_x86 如果在 Intel 处理器上运行，则为 true。
* @_mc680x0 如果在 Motorola 680x0 处理器上运行，则为 true。
* @_PowerPC 如果在 Motorola PowerPC 处理器上运行，则为 true。
* @_jscript 始终为 true。
* @_jscript_build 包含 JavaScript 脚本引擎的生成号。
* @_jscript_version 包含 major.minor 格式的 JavaScript 版本号。
```

> Internet Explorer 11 之前的所有版本的 Internet Explorer 都支持条件编译。  从 Internet Explorer 11 标准模式开始，Windows 8.x 应用商店应用不支持条件编译。  

## 后：

之前一直在做移动端的开发，没想到做 PC 端也会遇到这么多的兼容性问题。不同于移动端设备的繁杂和不确定性，PC 版的兼容更侧重于对特定浏览器的特性的了解，相比而言更为明确，而非因为某一款手机的诡异表现。

## 参考文档：

[Allow reserved words for properties](https://github.com/airbnb/javascript/issues/61)

[IE8 defineProperty/getOwnPropertyDescriptor clash with shim](https://github.com/es-shims/es5-shim/issues/5)

[Runtime transform](http://babeljs.io/docs/plugins/transform-runtime/)

[babel-plugin-transform-runtime definitions](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)

[super() not calling parent's constructor on IE9](https://github.com/babel/babelify/issues/133)

[postMessage method (window) Javascript](http://help.dottoro.com/ljgheukc.php)

[使用 F12 工具控制台查看错误和状态](https://msdn.microsoft.com/library/gg589530(v=vs.85).aspx)

[定义文档兼容性](https://msdn.microsoft.com/zh-cn/library/cc288325(v=vs.85).aspx)

[条件编译 (JavaScript)](https://msdn.microsoft.com/zh-cn/library/121hztk3(v=vs.94).aspx)
