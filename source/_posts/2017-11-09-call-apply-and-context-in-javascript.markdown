---
layout: post
title: "魔幻语言 JavaScript 系列之 call、bind 以及上下文"
date: 2017-11-09 14:24:57 +0800
comments: true
categories: translator
---

> 原文：[The Most Clever Line of JavaScript](http://blog.bloomca.me/2017/11/08/the-most-clever-line-of-javascript.html)
>
> 作者：[Seva Zaikov](https://twitter.com/blooomca)

## 原文

最近 [一个朋友](https://twitter.com/vedroarbuzov) 发给我一段非常有趣的 JavaScript 代码，是他在某个 [开源库中](https://github.com/pelias/openstreetmap/blob/313f208ea323232919e42bf88871d8e19ddacec3/stream/address_extractor.js#L54) 看到的：

```javascript
addressParts.map(Function.prototype.call, String.prototype.trim);
```

一开始，我觉得这是一个“不错的尝试”。但是，印象中 `map` 好像只接受一个参数，这里却出现第二个参数，所以去查看了 [MDN文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)，才知道可以传一个上下文（context）作为第二个参数。在这时候，我还无法解释这段代码，运行完之后感到更加困惑了，因为它竟然能如预期那样工作。

我花了至少半个小时来尝试理解这段代码，这是一个很有趣的例子，可以用来说明 JavaScript 是一门多么魔幻的语言，即使已经写了好几年的 JS。当然，你可以选择自己去弄清楚，如果想看看我的理解，请继续阅读。

那么，它到底是如何工作的呢？让我们从一种更简单的实现开始（实际上这种实现代码更短，并且更易读:)）：

```javascript
addressParts.map(str => str.trim());
```

[Function.prototype.call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 是 JavaScript 函数原型中的一个函数，它调用函数，使用第一个参数作为 `this` 参数，并传递剩余参数作为被调用函数的参数。举个例子：

```javascript
// this function has `Function` in prototype chain
// so `call` is available
function multiply(x, y) {
  return x * y;
}

multiply.call(null, 3, 5); // 15
multiply(3, 5); // same, 15
```

`map` 第二个参数的典型用法如下所示，假设有一个基于类的 React 组件，其功能是渲染一个按钮列表：

```javascript
class ExampleComponent extends Component {
  renderButton({ title, name }) {
    // without proper `this` it will fail
    const { isActive } = this.props;
    return (
      <Button key={title} title={title}>
        {name}
      </Button>
    );
  }

  render() {
    const { buttons } = this.props;

    // without second param our function won't be able
    // to access `this` inside
    const buttonsMarkup = buttons.map(this.renderButton, this);
  }
}
```

但是，以我的经验来看，这种使用第二个参数的做法并不常见，更常见的做法是使用类属性或装饰器来避免绑定。

> 译者注：`map` 第二个参数的用法等同于
> ```javascript
> const buttonsMarkup = buttons.map(this.renderButton.bind(this);
> ```

还有一个类似的方法 -- [Function.prototype.apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)，工作原理与 `call` 相同，只是第二个参数应该是一个数组（译者注：或者是一个类数组），它将被转换成一个参数列表，用逗号分隔。所以，让我们看看如何使用它来计算最大值：

```javascript
Math.max(1, 2, 3); // if we know all numbers upfront
// we can call it like that

Math.max([1, 2, 3]); // won't work!

Math.max.apply(null, [1, 2, 3]); // will work!

// however, ES2015 array destructuring works as well:
Math.max(...[1, 2, 3]);
```

现在，我们重新创建一个可以解决问题的函数调用方式。我们想删除字符串两端的空白字符，这个方法位于 `String.prototype` ，所以我们使用 `.` 操作符来调用它（虽然，字符串是原始值（primitive），但是当我们进行方法调用时，会在内部被转换成对象）。我们继续：

```javascript
// let's try to imagine how trim method is implemented
// on String.prototype
String.prototype.trim = function() {
  // string itself is contained inside `this`!
  const str = this;
  // this is a very naive implementation
  return str.replace(/(^\s+)|(\s+$)/g, '');
};

// let's try to use `.call` method to invoke `trim`
" aa ".trim.call(thisArg);

// but `this` is our string itself!
// so, next two calls are equivalent:
" aa ".trim.call(" aa ");
String.prototype.trim.call(" aa ");
```

我们现在距离答案更近一步，但是仍然没有解释清楚最初那段代码：

```javascript
addressParts.map(Function.prototype.call, String.prototype.trim);
```

让我们自己来实现 `Function.prototype.call`：

```javascript
Function.prototype.call = function(thisArg, ...args) {
  // `this` in our case is actually our function!
  const fn = this;

  // also, pretty naive implementation
  return fn.bind(thisArg)(...args);
};
```

现在，我们可以来理一理所有的东西。当我们在 `.map` 里面声明函数的时候，我们给 `Function.prototype.call` 绑定`String.prototype.trim` 作为 `this` 上下文，然后我们在数组中的每个元素上调用这个函数，把每个字符串作为 `thisArg` 参数的值传递给 `call`。这意味着，`String.prototype.trim` 将使用字符串作为 `this` 上下文来调用。我们已经知道这样做是有效的，看看下面的例子：

```javascript
String.prototype.trim.call(" aa "); // "aa"
```

问题解决了！但是，我认为这并不是一个好的做法，至于应该如何使用一种好的方式来完成这件事， 很简单，只需传递一个匿名函数就能搞定：

```javascript
addressParts.map(str => str.trim()); // same effect
```

## 也谈谈 JavaScript 中的 call、apply 和 bind

作者在最后这一段可能讲得有些简略，尤其是对于 `bind` 的用法，谈谈我的理解思路：

```javascript
// 我们从常用的 slice 说起
// 相信很多人都写过这样的代码
// 我们称之为方法借用
Array.prototype.slice.call([1, 2, 3], 1) // [ 2, 3]

// 也会有人这样写
[].slice.call([1, 2, 3], 1) // [2, 3]

// 但上面的例子其实不是其真实的使用场景，因为 [1, 2, 3] 本身就是一个 array，可以直接调用 slice
[1, 2, 3].slice(1) // [2, 3]

// 之前比较常见的场景是处理 argumnents，通过这种方式将这种类数组转换成真正的数组
Array.prototype.slice.call(arguments)

// 回到最上面的例子，我们已经知道使用 call 可以让你在某个特定上下文(context)调用函数(fn)
// fn.call(context [, ...args])
// 而对 call 来说，它的上下文就是 fn
// 所以 call 本身也是有上下文的，那我们为什么不可以直接给 call 指定一个上下文，就像这样：
Function.prototype.call.call(Array.prototype.slice, [1, 2, 3], 1) // [2, 3]

// 或者是这样，apply 接受一个数组
Function.prototype.call.apply(Array.prototype.slice, [[1, 2, 3], 1]) // [2, 3]

// 当然，也可以使用一下 bind，这样会返回一个新的函数
// 我们直接将 slice 绑定到 call 的上下文
var slice = Function.prototype.call.bind(Array.prototype.slice)
slice([1, 2, 3], 1) // [2, 3]

// 我们来稍微改动一下，跟上述 slice 的例子一致
var trim = Function.prototype.call.bind(String.prototype.trim)

// 上述 slice 等同于 Array.prototype.slice.call
// 所以这里的 trim，等同于 String.prototype.trim.call
// 那么
trim(' node') // 'node'

// 现在，在 map 里使用 trim
addressParts.map(Function.prototype.call.bind(String.prototype.trim))

// 回到最初的那段代码，这里面包含一个隐式的 bind 操作，与上面的代码等效
// 问题到这里就已经解决
addressParts.map(Function.prototype.call, String.prototype.trim)


// 如作者所言，这样的代码确实不容易阅读，不过对于我们理解 call、bind 以及 context 的概念仍是个很好的例子
// 我们还可以写得更复杂
// 不用担心，下面这段代码什么新东西都没有，不过是给 map 绑定到 call 而已
Function.prototype.call.bind(Array.prototype.map)(addressParts, Function.prototype.call, String.prototype.trim)
```
