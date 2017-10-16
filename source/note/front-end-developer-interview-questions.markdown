---
layout: page
title: "front-end-developer-interview-questions"
date: 2016-03-23 17:05
comments: true
sharing: true
footer: true
---

## Front-end-Developer-Interview-Questions

### 常见问题

#### 你在昨天/本周学到了什么？

了解细节，深入细节，跟优秀的人学习

#### 编写代码的哪些方面能够使你兴奋或感兴趣？

实现的过程，尤其是进入到从未涉猎的领域，并且一步步看到自己写的东西发挥它的威力

优雅而简洁的方式

#### 你最近遇到过什么技术挑战？你是如何解决的？

比如如何去实现 hack 一把历史栈。

查资料，上 MDN，

#### 在制作一个网页应用或网站的过程中，你是如何考虑其 UI、安全性、高性能、SEO、可维护性以及技术因素的？


#### 请谈谈你喜欢的开发环境。

webstorm

#### 你最熟悉哪一套版本控制系统？

git

* 你能描述当你制作一个网页的工作流程吗？

* 假若你有 5 个不同的样式文件 (stylesheets), 整合进网站的最好方式是?

压缩合并成一个

#### 你能描述渐进增强 (progressive enhancement) 和优雅降级 (graceful degradation) 之间的不同吗?
你如何对网站的文件和资源进行优化？
浏览器同一时间可以从一个域名下载多少资源？
有什么例外吗？
请说出三种减少页面加载时间的方法。(加载时间指感知的时间或者实际加载时间)
如果你参与到一个项目中，发现他们使用 Tab 来缩进代码，但是你喜欢空格，你会怎么做？
请写一个简单的幻灯效果页面。
如果今年你打算熟练掌握一项新技术，那会是什么？
请谈谈你对网页标准和标准制定机构重要性的理解。
什么是 FOUC (无样式内容闪烁)？你如何来避免 FOUC？
请解释什么是 ARIA 和屏幕阅读器 (screenreaders)，以及如何使网站实现无障碍访问 (accessible)。
请解释 CSS 动画和 JavaScript 动画的优缺点。
什么是 CORS，以及其要解决的问题？

### HTML 相关问题：

#### doctype(文档类型) 的作用是什么？

声明文档类型，提示浏览器应当按照什么规范来渲染出当前页面。比如，现在使用的 `<!DOCTYPE html>`，浏览器会按照最新的版本来渲染，至于为什么取消版本号，是以后声明方式不会变了，标准只需要一套，但标准会不断进化。

#### 浏览器标准模式 (standards mode) 和怪异模式 (quirks mode) 之间的区别是什么？

TBC 怪异模式会兼容一些旧的标准，以及一些不规范的用法

#### HTML 和 XHTML 有什么区别？

XHTML 有着更为严格的语法规范要求

#### 如果页面使用 'application/xhtml+xml' 会有什么问题吗？

#### 如果网页内容需要支持多语言，你会怎么做？

#### 在设计和开发多语言网站时，有哪些问题你必须要考虑？

#### data-属性的作用是什么？

使用自定义属性，可以在特定元素上绑定自定义数据

#### 如果把 HTML5 看作做一个开放平台，那它的构建模块有哪些？

#### 请描述 cookies、sessionStorage 和 localStorage 的区别。

cookies 是一种很古老的浏览器数据存储方案，是作为 HTTP 协议的组成部分，主要是用作回传给服务器的额外信息，同时服务器端可以通过 http 协议读写客户端的 cookies，大小通常限制到每个域名下 4k

localStorage 和 sessionStorage 都是作为 webStorage 的一部分，作为对 cookies 的补充。提供大容量的存储空间，不同浏览器会有差异。跟 cookies 不同，数据只存储在客户端。

sessionStorage 和 localStorage 基本一致，不同在于当回话消失数据消失。

#### 请解释 `<script>` 、 `<script async>` 和 `<script defer>` 的区别。

`<script async>` 表示应该立即加载下载当前脚本，但不应该妨碍页面中其他操作，比如下载其他资源。

`<script defer>` 表示脚本可以延迟到文档完全被解析和显示之后再执行。

#### 为什么通常推荐将 CSS `<link>` 放置在 `<head></head>` 之间，而将 JS `<script>` 放置在 `</body>` 之前？你知道相关解释吗？

CSS 应当写在 head 中，让浏览者能尽早的看到网站的完整样式，以避免页面元素由于样式缺失造成瞬间的白页或者给用户闪烁感

JS 放在后面主要是尽快呈现页面，不要阻塞页面渲染，使 JS 在加载过程中不影响内容表现

#### 什么是渐进式渲染 (progressive rendering)？

#### 你用过哪些不同的 HTML 模板语言？

Jade、ejs

### CSS 相关问题：

#### CSS 中类 (classes) 和 ID 的区别。

class 可以用作一组元素的样式、ID 只用作一个

#### 请问 "resetting" 和 "normalizing" CSS 之间的区别？你会如何选择，为什么？



#### 请解释浮动 (Floats) 及其工作原理。

#### 描述 `z-index` 和叠加上下文是如何形成的。


#### 请描述 BFC(Block Formatting Context) 及其如何工作。

直译过来就是块级格式化上下文，是指页面中的一块渲染区域，有一套规则来决定其子元素将如何定位，以及和其他元素的关系和相互作用。大致就是内部元素一个接一个垂直排列，相邻元素 margin 会重合，左侧与包含元素的左侧相接触。对一个 BFC 区域而言，它是一个独立容器，内部子元素不影响外部，它也不会与 float box 重叠，计算高度时，浮动元素也会参与。

生成 BFC 最常用的方式就是 `overflow: hidden;`，事实上规则是 overflow 的值不为 visible 都可以生成 BFC，还有就是 float 不为 none，position 为 absolute 或者 fixed，display 为 flex、table、table-cell、table-caption、inline-block、inline-flex、inline-table

#### 列举不同的清除浮动的技巧，并指出它们各自适用的使用场景。

`clear: both;`

#### 请解释 CSS sprites，以及你要如何在页面或网站中实现它。

将多个图标类合并成一张图片，在通过改变 `background-position` 的方式使用某一个特定的图标。

问题主要在于维护成本和可访问性（背景图片是不可访问的）。

使用的时候应该进行图片分类，把内容固定、不会作太多变动的图标归入一个 sprites 中。

#### 你最喜欢的图片替换方法是什么，你如何选择使用。


#### 你会如何解决特定浏览器的样式问题？



#### 如何为有功能限制的浏览器提供网页？

#### 你会使用哪些技术和处理方法？

#### 有哪些的隐藏内容的方法 (如果同时还要保证屏幕阅读器可用呢)？
你用过栅格系统 (grid system) 吗？如果使用过，你最喜欢哪种？
你用过媒体查询，或针对移动端的布局/CSS 吗？
你熟悉 SVG 样式的书写吗？
如何优化网页的打印样式？
在书写高效 CSS 时会有哪些问题需要考虑？
使用 CSS 预处理器的优缺点有哪些？
请描述你曾经使用过的 CSS 预处理器的优缺点。
如果设计中使用了非标准的字体，你该如何去实现？
请解释浏览器是如何判断元素是否匹配某个 CSS 选择器？
请描述伪元素 (pseudo-elements) 及其用途。
请解释你对盒模型的理解，以及如何在 CSS 中告诉浏览器使用不同的盒模型来渲染你的布局。
请解释 * { box-sizing: border-box; } 的作用, 并且说明使用它有什么好处？
请罗列出你所知道的 display 属性的全部值
请解释 inline 和 inline-block 的区别？
请解释 relative、fixed、absolute 和 static 元素的区别
CSS 中字母 'C' 的意思是叠层 (Cascading)。请问在确定样式的过程中优先级是如何决定的 (请举例)？如何有效使用此系统？
你在开发或生产环境中使用过哪些 CSS 框架？你觉得应该如何改善他们？
请问你有尝试过 CSS Flexbox 或者 Grid 标准规格吗？
为什么响应式设计 (responsive design) 和自适应设计 (adaptive design) 不同？
你有兼容 retina 屏幕的经历吗？如果有，在什么地方使用了何种技术？
请问为何要使用 translate() 而非 absolute positioning，或反之的理由？为什么？

### JS 相关问题：

#### 请解释事件代理 (event delegation)。

基于事件冒泡原理，当出发事件时，会从子元素一层层向外传递，使用 `event.stopPropagation()` 可以阻止当前事件的进一步冒泡行为。

先捕获再冒泡

#### 请解释 JavaScript 中 this 是如何工作的。

当前调用作用域对象，比如全局就是 global

当使用 new 实例化时，指向创建的对象

#### 请解释原型继承 (prototypal inheritance) 的原理。

原型链的概念。当访问一个对象的属性时，会先查找当前对象的属性，如果没有，则查找原型对象 `__proto__` 上的属性，这样一步步，就像链条一样查找。

```js
Doctor.prototype = new Man();
```

```js
Object.create = function (parent) {
    function F() {}
    F.prototype = parent;
    return new F();
};
```

__proto__ 和 prototype

对象都有原型对象，隐式 __proto__ ，隐式原型指向创建这个对象的构造函数(constructor)的原型对象（prototype)

方法是特殊对象，有一个属性 prototype，指向函数的原型对象，用来实现基于原型的继承与属性的共享

####  你怎么看 AMD vs. CommonJS？



#### 请解释为什么接下来这段代码不是 IIFE (立即调用的函数表达式)：function foo(){ }();.
要做哪些改动使它变成 IIFE?
描述以下变量的区别：null，undefined 或 undeclared？
该如何检测它们？
什么是闭包 (closure)，如何使用它，为什么要使用它？
请举出一个匿名函数的典型用例？
你是如何组织自己的代码？是使用模块模式，还是使用经典继承的方法？
请指出 JavaScript 宿主对象 (host objects) 和原生对象 (native objects) 的区别？

#### 请指出以下代码的区别：function Person(){}、var person = Person()、var person = new Person()？

####.call 和 .apply 的区别是什么？

#### 请解释 Function.prototype.bind？

创建一个绑定特定执行环境（上下文 context）的新函数。函数与原函数一致，但执行环境是传入的第一个参数。

在什么时候你会使用 document.write()？
请指出浏览器特性检测，特性推断和浏览器 UA 字符串嗅探的区别？
请尽可能详尽的解释 AJAX 的工作原理。
请解释 JSONP 的工作原理，以及它为什么不是真正的 AJAX。
你使用过 JavaScript 模板系统吗？
如有使用过，请谈谈你都使用过哪些库？
#### 请解释变量声明提升 (hoisting)。

```js
var i = 0;
```

执行时实际上是：

```js
var i;

i = 0;
```

请描述事件冒泡机制 (event bubbling)。
"attribute" 和 "property" 的区别是什么？
为什么扩展 JavaScript 内置对象不是好的做法？
#### 请指出 document load 和 document ready 两个事件的区别。

load 是指所有资源都加载完成，ready 是指文档加载完成

####== 和 === 有什么不同？
####请解释 JavaScript 的同源策略 (same-origin policy)。
#### 如何实现下列代码：
[1,2,3,4,5].duplicator(); // [1,2,3,4,5,1,2,3,4,5]

#### 什么是三元表达式 (Ternary expression)？“三元 (Ternary)” 表示什么意思？

#### 什么是 "use strict"; ? 使用它的好处和坏处分别是什么？

使用严格模式

#### 请实现一个遍历至 100 的 for loop 循环，在能被 3 整除时输出 "fizz"，在能被 5 整除时输出 "buzz"，在能同时被 3 和 5 整除时输出 "fizzbuzz"。

```js
// 你想到的
function loop() {
  for (let i = 0; i < 100; i++) {
    if (i % 3 === 0 && i % 5 === 0) {
      console.log(i + 'fizzbuzz');
    } else if (i % 3 === 0) {
      console.log(i + 'fizz');
    } else if (i % 5 === 0) {
      console.log(i + 'buzz');
    }
  }
}

// 来点其他
function loop() {
  let three = 'fizz';
  let five = 'buzz';
  for (let i; i< 100; i++) {
    if (i % 3 === 0 || i % 5 === 0) {
      console.log(i)
      if (i % 3 === 0) console.log(three)
      if (i % 5 === 0) console.log(five)
    }
  }
}
```

#### 为何通常会认为保留网站现有的全局作用域 (global scope) 不去改变它，是较好的选择？

维持现有的全局作用域意味着是可控的。我们应尽量减少使用全局量，防止混乱和污染。

#### 为何你会使用 load 之类的事件 (event)？此事件有缺点吗？你是否知道其他替代品，以及为何使用它们？
请解释什么是单页应用 (single page app), 以及如何使其对搜索引擎友好 (SEO-friendly)。
What is the extent of your experience with Promises and/or their polyfills?
#### 使用 Promises 而非回调 (callbacks) 优缺点是什么？



使用一种可以编译成 JavaScript 的语言来写 JavaScript 代码有哪些优缺点？
你使用哪些工具和技术来调试 JavaScript 代码？
你会使用怎样的语言结构来遍历对象属性 (object properties) 和数组内容？
请解释可变 (mutable) 和不变 (immutable) 对象的区别。
请举出 JavaScript 中一个不变性对象 (immutable object) 的例子？
不变性 (immutability) 有哪些优缺点？
如何用你自己的代码来实现不变性 (immutability)？
请解释同步 (synchronous) 和异步 (asynchronous) 函数的区别。
#### 什么是事件循环 (event loop)？
#### 请问调用栈 (call stack) 和任务队列 (task queue) 的区别是什么？

### 测试相关问题：

对代码进行测试的有什么优缺点？
你会用什么工具测试你的代码功能？
单元测试与功能/集成测试的区别是什么？
代码风格 linting 工具的作用是什么？

### 效能相关问题：

你会用什么工具来查找代码中的性能问题？
你会用什么方式来增强网站的页面滚动效能？
请解释 layout、painting 和 compositing 的区别。

### 网络相关问题：

为什么传统上利用多个域名来提供网站资源会更有效？
请尽可能完整得描述从输入 URL 到整个网页加载完毕及显示在屏幕上的整个流程。
Long-Polling、Websockets 和 Server-Sent Event 之间有什么区别？
请描述以下 request 和 response headers：
Diff. between Expires, Date, Age and If-Modified-...
Do Not Track
Cache-Control
Transfer-Encoding
ETag
X-Frame-Options
什么是 HTTP action？请罗列出你所知道的所有 HTTP action，并给出解释。

### 代码相关的问题：

#### 问题：foo的值是什么？

```js
var foo = 10 + '20';
```

* 加号运算符的作用
* 自动转化

answer:

```
"1020"
```

#### 问题：如何实现以下函数？

```js
add(2, 5); // 7
add(2)(5); // 7
```

answer:

```js
var add = funtion() {

}
```

#### 问题：下面的语句的返回值是什么？

```js
"i'm a lasagna hog".split("").reverse().join("");
```

answer

```
goh angasal a m'i
```

#### 问题：window.foo的值是什么？

```js
( window.foo || ( window.foo = "bar" ) );
```

answer

```
bar
```

#### 问题：下面两个 alert 的结果是什么？

```js
var foo = "Hello";
(function() {
  var bar = " World";
  alert(foo + bar);
})();
alert(foo + bar);
```

answer

```
Hello World
```

#### 问题：foo.length 的值是什么？

```js
var foo = [];
foo.push(1);
foo.push(2);
```

2


#### 问题：foo.x 的值是什么？

```js
var foo = {n: 1};
var bar = foo;
foo.x = foo = {n: 2};
```

answer

undefined

解析：

先看这个例子：

```js
var foo = {n: 1};
var bar = foo;
foo.x = foo;
foo = {n: 2};
```

* 首先创建了一个对象1 `{n: 1}`，foo 是它的引用
* 然后 bar 也是它的引用
* 然后引用 foo.x 指向 foo，则对象1的属性 x 指向其自身
* 最后，引用 foo 被重新赋值，指向一个对象2 `{n: 2}`

结果就是对象1有两个属性 n: 1, 属性 x 指向其自身，同时 引用 bar 指向该对象。而 foo 指向对象2.


再看看这个例子：

```js
var foo = {n: 1};
var bar = foo;
foo = {n: 2};
foo.x = foo;
```

前面都一样，接下来：

* 引用 foo 被重新赋值，指向一个对象2 `{n: 2}`
* 引用 foo.x 指向 foo

结果就是引用 bar 扔指向对象1，引用 foo 指向对象2，有两个属性 n: 2，属性 x 指向其自身。


回到原初问题：

```js
foo.x = foo = {n: 2};
```

相当于：

```js
foo.x = (foo = {n: 2});
```

那这个要怎么看呢，引用 foo.x 指向一个新的对象 XXX，可以理解成对象1，新增一个属性 x，然后指向 XXX。然后，这个 XXX 到底是什么？其实就是对象2 `{n: 2}`，同时引用 foo 被重新赋值，指向对象2。而引用 bar 自始至终不变，仍然指向对象1。





#### 问题：下面代码的输出是什么？

```js
console.log('one');
setTimeout(function() {
  console.log('two');
}, 0);
console.log('three');
```

answer:

one
three
two

### 趣味问题：

你最近写过什么的很酷的项目吗？
在你使用的开发工具中，最喜欢哪些方面？
你有什么业余项目吗？是哪种类型的？
你最爱的 IE 特性是什么？
你对咖啡有没有什么喜好？



### 注释

* TBC(to be confirmed)


### 面试题

#### reduce 求和

```js
function sum(arr) {
	return arr.reduce((prev, curr) => prev + curr, 0);
}

Array.prototype.sum = function() {
	return this.reduce((prev, curr) => prev + curr, 0);
}
```

#### 求数组不同数的和

```js
sum([3, 2, 7, 6, 9, 4, 6, 2, 5, 6, 3]) //25
sum([7, 9, 4, 5]) //25
sum([4, 7, 5, 3]) //19
```

```js
// 遍历数组，在 map 中储存数字出现次数
// 然后从 map 里取出出现次数为一的数进行求和
// 最常规的做法，但繁杂
function sum(list) {
  let map = {};
  let sum = 0;

  list.forEach(num => {
    if (map[num] === undefined) {
      map[num] = 1;
    } else {
      map[num] += 1;
    }
  });

  for (num in map) {
    if (map[num] === 1) {
      sum += ~~num;
    }
  }

  return sum;
}
```

```js
// 另一种问题
function sum(list) {
  let sum = 0;
  let set = new Set();
  list.map(num => set.add(num));

  for (let num of set) {
    sum += num;
  }

  return sum;
}

// 排序
// 重复数求一次
function sum(list) {
  list.sort();

  let prev;
  let sum = 0;
  list.forEach(num => {
    if (num !== prev) {
      sum += num;
    }
    prev = num;
  });

  return sum;
}
// 降序排序
list.sort((prev, curr) => {
  if (prev > curr) return 1;
  if (prev < curr) return -1;
  return 0
})
```

#### 问题 f(a)(b)(c)....(x)()        a+b+c+...+x，其中 f() = 0

test

```js
f(6)(7)(3)() //16
f() //0
```

简单方法

```js
function f(a) {
  if (a === undefined) return 0;
  return function(b) {
    if (b === undefined) {
      return a;
    } else {
      return f(a + b)
    }
  }
}

// use arrow function in es6
const f = a =>a === undefined ? 0 : b => b === undefined ? a : f(a + b)
```

#### 另一种问题，f(a)(b)(c) ...f(x)  a+b+c+...+x
```js
function sum(...args) {
  // let args = [].slice.call(arguments, 1);

  return [...args].reduce((prev, curr) => prev + curr );
}
// 假设已经有一个 curry 函数
let f = _.curry(sum);


// test
function sum(a, b, c) {
  return a + b + c;
}
let f = _.curry(sum);
f(8)(4)(7);

function sum() {
  let args = [].slice.call(arguments, 1);

  return args.reduce((prev, curr) => prev + curr );
}
let f = _.curry(sum);
f(8)(4)(7);


function sub_curry(fn /*, variable number of args */) {
    var args = [].slice.call(arguments, 1);
    return function () {
        return fn.apply(this, args.concat([].slice.call(arguments, 1)));
    };
}
// 柯里化函数
function curry(fn, length) {
    length = length || fn.length;
    return function () {
        if (arguments.length < length) {
            // not all arguments have been specified. Curry once more.
            var combined = [fn].concat([].slice.call(arguments, 1));
            return length - arguments.length > 0
                ? curry(sub_curry.apply(this, combined), length - arguments.length)
                : sub_curry.call(this, combined );
        } else {
            // all arguments have been specified, actually call function
            return fn.apply(this, arguments);
        }
    };
}

// function f(num) {
//   if (num === undefined) return 0;
//
//   return function(num) {
//     return sum.apply(this, [].slice.call(arguments, 1).push(num));
//   }
// }


var f = curry(sum);




var fn = function(a, b, c) { return [a, b, c]; };


// these are all equivalent
fn("a", "b", "c");
sub_curry(fn, "a")("b", "c");
sub_curry(fn, "a", "b")("c");
sub_curry(fn, "a", "b", "c")();
```


#### 关于事件轮训的原理

```js
var len = 4;
while (len--) {
  setTimeout(function() {
    console.log(len);
  }, 0);
  console.log(len);
}
```


#### 字符串的用法

```js
'hello' => 'h-e-l-l-o'
```


```js
// 借用数组方法
Array.prototype.join.call(str, '-')

// 转数组
[...str].join('-')

// 另一种转达
str.split('').join('-')
```


#### promise 合并，生成器

display

flex-box

文本溢出: text-ellipse

fixed 失效：transform

边界值

事件循环原理

promise 原理

webpack

算法：二分查找、top n、斐波那契、碰撞检测

301 和 302

> article: 关于 length, 类数组，以及 call 和 apply 的区别

深拷贝 浅拷贝


关于请求的先后顺序的问题，以及重复，这里举了一个 promise 的例子
考察信号量（解决竞争的常见方法，主要侧重于对基础知识的了解）
类似时间戳，但其实不需要比较先后，只需要比对通信双方是否具有一致性

随机数均匀分布

* 还是得掌握一种框架才行
* npm 至少得自己去写个包
* 参与开源项目
* node 的原理你都懂么
* 性能优化
* HTTP 协议
* 浏览器渲染原理
* 安全 ？？？貌似没人特别关注
* 原理，还是原理，构建工具原理，插件机制，真正了解其原理，用，谁不会啊
