---
layout: post
title: "为什么是 JSONP"
date: 2016-04-01 21:06:58 +0800
comments: true
categories:
---

<!-- # JSONP，跨域，安全策略，以及 WHY

服务器端、浏览器端

原生实现，包装 Api

同步请求？异步请求

文件上传

安全，跨域问题 -->

## AJAX、JSON、JSONP

在 WEB 开发中，经常见到诸如 AJAX、JSON、JSONP 这些词，但这三种东西到底是什么，又有什么关系和区别。

### AJAX （Asynchronous JavaScript + XML）

{% blockquote Jesse James Garrett, http://adaptivepath.org/ideas/ajax-new-approach-web-applications/ Ajax: A New Approach to Web Applications%}
Ajax isn’t a technology. It’s really several technologies, each flourishing in its own right, coming together in powerful new ways. Ajax incorporates:

* standards-based presentation using XHTML and CSS;
* dynamic display and interaction using the Document Object Model;
* data interchange and manipulation using XML and XSLT;
* asynchronous data retrieval using XMLHttpRequest;
* and JavaScript binding everything together.
{% endblockquote %}


异步 JavaScript + XML, 本身不是一种技术, 是在2005年由 Jesse James Garrett 提出的一个术语, 描述了一种结合使用大量已有技术的方式, 包括: HTML 或 XHTML, CSS, JavaScript, DOM, XML, XSLT, 还有最重要的 XMLHttpRequest 对象.

尽管在 AJAX 中 X 代表 XML, 但现在 JSON 使用的更多，因为 JSON 具有很多优势，比如更轻量并且是 JavaScript 的一部分. 在 AJAX 模型中 JSON 和 XML 都用于承载信息.

### JSON（Javascript Object Notation）

JSON 是一种轻量级的数据交换格式。由道格拉斯·克罗克福特（Douglas Crockford）在 2012 年发明，并逐渐取代 XML 成为事实上的数据交换格式标准。

JSON 基于 JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999的一个子集。但采用完全独立于语言的文本格式，并使用了类似于 C 语言家族的习惯。

在 JSON 中，一共 6 种数据类型：

* number：跟 Javascript 的数值一致，除去未曾使用的八进制与十六进制格式，和一些编码细节
* boolean：`true` 和 `false`
* string：是由双引号包围的任意数量Unicode字符的集合，使用反斜线转义
* null：`null`
* array：数组是值（value）的有序集合。一个数组以“[”（左中括号）开始，“]”（右中括号）结束，值之间使用“,”（逗号）分隔
* object：对象是一个无序的“‘名称/值’对”集合。一个对象以“{”（左括号）开始，“}”（右括号）结束，每个“名称”后跟一个“:”（冒号）；“‘名称/值’ 对”之间使用“,”（逗号）分隔

以及上面的任意组合。

在 javascript 中有一个全局的对象 JSON，包含两个方法 `JSON.stringify()` 和 `JSON.parse()`，用于序列化和解析 JSON。

### JSONP（JSON with Padding）

最初是开发者为了解决跨域问题搞出来的一个颇为奇怪的东西，产生原因和名字一样古怪，光听名字恐怕没几个人知道说的是个什么东西。

因为 ajax 请求收到同源策略的限制不允许跨域访问，而在实际开发中又常常会有类似的需求。

刚好 `<script>` 标签可以引用其他站点的静态资源，想想我们有时候在站点引入的数据统计类的 js。

但我们要的是数据，而不是一段静态代码，怎么办？

这还不简单吗，让服务器动态生成 js ，再把数据放进去不就可以吗。为了区分不数据，还需要针对返回的数据做一个标识，其实就是在数据外面包裹一个函数名。

然后需要浏览器端预先设置好这样一个函数，返回的数据就相当于一次执行过程，对获取数据的处理。

### 总结

1. AJAX 是一类技术的集合，其中最重要的是 `XMLHttpRequest`
2. JSON 是一个数据交换格式
3. JSONP 是为了解决跨域问题搞出来的一种获取数据的方式


## 下面举个栗子

### 服务器

这里使用 node 返回一段简单的数据。

```js
/**
 * 一个简单的 http 服务器,返回 json 数据
 * 跟 node 主页上的那个经典例子没太大差别
 */

var http = require('http');
var urllib = require('url');

var host = '127.0.0.1';
var port = 9999;

var data = {'name': 'Mirreal', 'age': '24'};

http.createServer(function(req, res) {
  var params = urllib.parse(req.url, true);

  if (params.query && params.query.callback) {

    var str =  params.query.callback + '(' + JSON.stringify(data) + ')'; // jsonp
    res.writeHead(200, { 'Content-Type': 'application/javascript' });
    res.end(str);
  } else {
    res.end(JSON.stringify(data)); // 普通的json
  }

}).listen(port, host, function() {
  console.log('server is listening on port ' + port);

});
```

### 浏览器

```js
// zepto 的写法
$.ajax({
  type: 'GET',
  url: 'http://127.0.0.1:9999',
  data: { _input_charset: 'utf-8' },
  dataType: 'jsonp',
  timeout: 300,
  context: $('body'),
  success: function(data){
    console.log(data)
  },
  error: function(xhr, type) {
    console.log('Ajax error!')
  }
});
```

这样就很轻松的通过 JSONP 的方式获取到数据，我们也不需要关心里面究竟是怎么一回事，但经常会有人问起像“为什么 jsonp 不能使用 POST 方法”的问题，其实稍微了解一下 JSONP 的原理，这种问题完全就不存在了。

虽然像 jQuery 这类库将 jsonp 封装到 Ajax 上，但准确来讲是不对的。因为 jsonp 只是通过动态地通过 `<script>` 标签去请求一段 js 代码（或者叫数据），而非使用 XMLHttpRequest ，原理就像下面这样：

```js
/**
 * 对 jsonp 的一种简单封装
 * @param {Object} options
 * @returns null
 */
function getJsonp(options) {

  var callbackName = options.callbackName;
  var url = options.url;


  var scriptElem = document.createElement('script');
  scriptElem.setAttribute('src', url + '?callback=' + callbackName);

  scriptElem.onload = function(e) {
    delete window[callbackName];
    this.parentNode.removeChild(this);
  };

  scriptElem.onerror = function(e) {
    console.log(e, 'load error');

    delete window[callbackName];
    this.parentNode.removeChild(this);
  };

  window[callbackName] = options.success;

  // 调用
  document.querySelector('head').appendChild(scriptElem);
}
```

这段代码对 JSONP 进行一层简单包装，调用也很简单：

```js
getJsonp({
  'url': 'http://127.0.0.1:9999/',
  'callbackName': 'log',
  'success': function (data) {
    console.log('我是回调函数,我拿到数据了', data);
  }
});
```

看上去代码还挺长的，实际上核心代码不多，分三步：

#### 1.创建一个 `<script>` 标签，并设置其 url

```js
var scriptElem = document.createElement('script');
scriptElem.setAttribute('src', url + '?callback=' + callbackName);
```

#### 2.设置回调函数

```js
window[callbackName] = options.success;
```

这里简单处理，直接把传入的回调函数设置成全局的

#### 3.调用

```js
document.querySelector('head').appendChild(scriptElem);
```

实际上就是把 `<script>` 加到 html 文档中，这样就会去加载标签的内容，也就是一个 js 文件。


但通常现实中跑的代码内容会更多，包含一些错误控制、参数拼接、超时处理、性能安全等方面的，但它仍然清楚地描述 JSONP 的原理。

## 安全

早期的浏览器处于安全层面的考量，使用同源策略，限制了一个源（origin）中加载文本或脚本与来自其它源（origin）中资源的交互方式。

但是随着互联网的发展催生了跨域访问进行数据交互的需求，于是 JSONP 就产生了，以及后来的 CORS 机制，允许 XMLHttpRequest 对象发起跨域的请求。

但是另一方面，也增加了安全风险，我们在使用的时候应当更加谨慎小心，防止 XSS、CSRF 等攻击。

<!-- JSONP 的安全问题通常属于 CSRF（ Cross-site request forgery 跨站请求伪造）攻击范畴。 -->

## 其他

### 数据预览

之前碰到一个问题，为什么调用一些接口返回的数据无法使用 Chrome 预览，我自己写测试接口的时候也碰到过。其实这里完全没有什么技术可言，只是因为没有在 response 头部加上 `Content-Type: application/javascript`，仅此而已。

## 参考文档

* [AJAX | MDN](https://developer.mozilla.org/zh-CN/docs/AJAX)
* Jesse James Garrett 的文章：[Ajax: A New Approach to Web Applications](http://adaptivepath.org/ideas/ajax-new-approach-web-applications/)
* [JSON](http://www.json.org/json-zh.html)
