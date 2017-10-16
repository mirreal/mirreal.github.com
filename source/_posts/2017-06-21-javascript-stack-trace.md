---
layout: post
title:  "JavaScript 异常的防范与监控"
date:   2017-06-21 15:12:00
author: mirreal
categories: team
---

一套完善的前端体系应少不了异常统计与监控，即使有足够的质量保证体系，难免会出现一些意料之外的事，尤其是在复杂的网路环境和运行环境之下。为了保证代码的健壮性以及页面的稳定性，我们从多个方面来做异常的防范和监控。

## 三种思路

### 主动防御

对于我们操作的数据，尤其是由 API 接口返回的，时常会有一个很复杂的深层嵌套的数据结构。为了代码的健壮性，很多时候需要对每一层访问都作空值判断，就像这样：

```javascript
props.user &&
props.user.posts &&
props.user.posts[0] &&
props.user.posts[0].comments &&
props.user.posts[0].comments[0]
```

类似的代码大家可能都写过，没写过大概也见到别人写过。看起来确实相当地不美观，有句话说得很棒：

**The opposite of beautiful is not ugly, but wrong.**

我们得找到一种，更简单、更优雅、更安全的方式来处理这种情形。参考这篇文章：[Safely Accessing Deeply Nested Values In JavaScript](https://medium.com/javascript-inside/safely-accessing-deeply-nested-values-in-javascript-99bf72a0855a)，文章提到借助 Ramda、Lenses、Lodash 以及 Immutable.js 等类库的方式，并提供一个非常简洁明了的原生解决方案：

```js
function getIn(p, o) {
    return p.reduce(function(xs, x) {
        return (xs && xs[x]) ? xs[x] : null;
    }, o);
}
```

接下来我们这样访问就可以了：

```javascript
getIn(['user', 'posts', 0, 'comments'], props)
```

如果正常访问到，则返回对应的值，否则返回 `null`。

这里提供的只是主动防御的一种情形，关于如何编写更安全的代码这里不作深入展开。

### 全局监控

浏览器提供 [`window.onerror`](https://developer.mozilla.org/zh-CN/docs/Web/API/GlobalEventHandlers/onerror) API 来帮助我们进行全局的错误监控：

* 当 JavaScript 运行时错误（包括语法错误）发生时，会执行 `window.onerror()``
* 当一项资源（如 `<img>` 或 `<script>` ）加载失败，能被单一的 `window.addEventListener` 捕获

```javascript
/**
 * @param  {String} message 错误信息
 * @param  {String} source  发生错误的脚本URL
 * @param  {Number} lineno  发生错误的行号
 * @param  {Number} colno   发生错误的列号
 * @param  {Object} error   Error对象
 */
window.onerror = function(message, source, lineno, colno, error) {
  // ...
}
```

其中 error 对象包含详细的错误堆栈信息，在 IE9 以前，没有这个参数。

### 针对性捕获 try..catch

可以通过 try..catch 来主动抓取错误，想要对一段代码 try..catch，我们可以这样：

```javascript
try {
  // ...
} catch (error) {
  handler(error)
}
```

对一个函数做 try..catch 封装:

```javascript
function tryify(func) {
  return function() {
    try {
      return func.apply(this, arguments)
    } catch (error) {
      handleError(error)

      throw error
    }
  }
}
```

## 为什么是 script error

方案已经明确，但还有一些问题。在查看 JavaScript 错误统计时，发现 80% 以上都是 "script error"。原来，当加载自不同域的脚本中发生语法错误时，为避免信息泄露，语法错误的细节将不会报告，而代之简单的 "Script error."

而在大多数情况下，我们的静态资源放在专门的 CDN 服务器上，跟站点并不在一个域，所以如果只是简单的抓取，只会得到一堆意义不大的 `script error`

解决方案：

* 添加 CORS 支持
* 使用 try..catch

### 添加 CORS 支持

需要做两点：

1.在 script 便签添加 crossorigin，默认值 `crossorigin="anonymous"`

```html
<script src="//xxx.com/example.js" crossorigin></script>
```

在 require.js 里提供一个 [onNodeCreated hook](https://github.com/requirejs/requirejs/blob/e97fe4dd894d8a07712156e17cefa28b954a9c3e/require.js#L1946)，供我们提供扩展，要添加 crossorigin 属性，如下所示：

```html
<script>
// 如果在 require.js 加载之前定义了 requirejs，require.js 会将其作为一个对象传入 requirejs.config
var requirejs = {
  onNodeCreated: function(node, config, id, url) {
    node.setAttribute('crossorigin', 'anonymous')
  }
}
</script>
<script src="//xxx.com/require.js" charset="utf-8"></script>
```

在 2.2.0 版本以上可用（很遗憾的是，目前的集成解决方案版本刚好低于这个版本）。

2.同时在 CDN 服务器增加响应头 `access-control-allow-orgin`，配置允许访问 CORS 的域，否则浏览器直接将禁止加载。

### try..catch

这一点上面也有提到，算是一种比较通用，可定制强的方案。当然，在性能上也会有一些损耗。

综合考虑，try..catch 通用性更好，但由于其在性能方面的一些损耗，CORS 优于 try..catch

## 一个监控小工具

随后，介绍一个 JavaScript stack trace 的小工具：https://github.com/CurtisCBS/monitor ，工具由 [Curtis](https://github.com/CurtisCBS) 和 [mirreal](https://github.com/mirreal) 共同完成。

主要是用于捕获页面 JavaScript 异常报错，捕获异常类型包含:

* JavaScript runtime 异常捕捉 √
* 静态资源 load faided 异常捕捉 √
* console.error 的异常捕获 √
* try..catch 错误捕获 √

使用方式也很简单，但使用 script mode 引入文件后，调用 init 函数，进行初始化配置和监听

```html
<script src="//unpkg.com/jstracker@latest/dist/jstracker.js"></script>

<script>
  jstracker.init({
    delay: 1000,
    maxError: 10,
    sampling: 1,
    report: function(errorLogs) {
      console.table(errorLogs)
    }
  })
</script>
```

如果是使用 module mode，如下：

```javascript
// npm install jstracker --save
import jstracker from 'jstracker'

jstracker.init({
  concat: false,
  report: function(errorLogs) {
    // console.log('send')
  }
})
```

如果要使用 try..catch 捕获，jstracker 暴露出一个 `tryJS` 对象，可以处理 try..catch 包装，就像这样：

```javascript
import jstracker from 'jstracker';

this.handleSelect = jstracker.tryJS.wrap(this.handleSelect);
```

所有错误信息统一由 report 函数处理，可以在此之上做数据处理：

```javascript
// ubt.js
import jstracker from 'jstracker';
import utility from 'utility';

jstracker.init({
    concat: false,
    report: function(errorLogs) {
        const errorLog = errorLogs[0];
        errorLog.ua = window.navigator.userAgent;
        ubtTracker.send(errorLog);
    }
});

const ubtTracker = {
    key: {
        UBT_JS_TRACKER: 'xxxx-xxxx-xxxx'
    },

    send(data) {
        if (typeof window['__bfi'] === 'undefined') window['__bfi'] = [];

        const value = utility.deserializeUrl(data);
        window['__bfi'].push(['_tracklog', this.key.UBT_JS_TRACKER, value]);
    }
};

function wrapContext(ctx) {
    for (const func in ctx) {
        ctx[func] = jstracker.tryJS.wrap(ctx[func]);
    }
}

export {
    wrapContext,
    ubtTracker,
    jstracker
};
```

## 概述

作为开发者以及项目维护者的身份，我们应当编写更安全健壮的代码。但由于环境的多样性，无论再完善的测试，code review 都难免都所疏漏，我们需要一套监控系统来完善整个前端体系。

在监控的时候，出于同源安全策略无法拿到准确的错误信息，在此，有两种解决方案：

* 增加 CORS 支持
* 使用 try..catch 进行异常捕获

最后，我们对整个监控工作封装了一个基础的核心，可以监控 JavaScript Runtime 异常，资源加载异常，以及 try..catch 捕获异常等，并给出一个实际工作中的示例。
