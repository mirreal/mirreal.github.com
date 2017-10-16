---
layout: post
title: "JavaScript 中的匿名递归"
date: 2017-08-09 14:54:22 +0800
comments: true
categories: translator
---

> 原文：[Anonymous Recursion in JavaScript](https://dev.to/simov/anonymous-recursion-in-javascript)
>
> 作者：Simeon Velichkov

```javascript
(
  (
    (f) => f(f)
  )
  (
    (f) =>
      (l) => {
        console.log(l)
        if (l.length) f(f)(l.slice(1))
        console.log(l)
      }
  )
)
(
  [1, 2, 3]
)
```

是的，这就是想要分享给大家的一个有趣的示例。这个例子包含以下特性：[闭包](https://en.wikipedia.org/wiki/Closure_(computer_programming))，[自执行函数](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)，[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，[函数式编程](https://en.wikipedia.org/wiki/Functional_programming) 和 [匿名递归](https://en.wikipedia.org/wiki/Anonymous_recursion)。

你可以复制粘贴上述代码到浏览器控制台，会看到打印结果如下：

```
[ 1, 2, 3 ]
[ 2, 3 ]
[ 3 ]
[]
[]
[ 3 ]
[ 2, 3 ]
[ 1, 2, 3 ]
```

说到函数式编程，这里有一个使用 [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)) (JavaScript 借鉴过的其中一门语言)编写的类似例子：

```scheme
(
  (
    (lambda (f) (f f))
    (lambda (f)
      (lambda (l)
        (print l)
        (if (not (null? l)) ((f f) (cdr l)))
        (print l)
      )
    )
  )
  '(1 2 3)
)
```

## Unwind

像其他很多编程语言一样，函数调用是通过在函数名称后添加括号 `()` 来完成的：

```javascript
function foo () { return 'hey' }
foo()
```

在 JavaScript 中我们可以使用括号包裹任意数量的表达式：

```javascript
('hey', 2+5, 'dev.to')
```

上面代码返回结果是 `'dev.to'`，原因是 JavaScript 返回最后一个表达式作为结果。

使用括号 `()` 包裹一个匿名函数表示其结果就是 [匿名函数](https://en.wikipedia.org/wiki/Anonymous_function) 本身。

```javascript
(function () { return 'hey' })
```

这本身并没有用处，因为匿名函数没有命名，无法被引用，除非在初始化的时候立即调用它。

就像是普通函数一样，我们可以在其后面添加括号 `()` 来进行调用。

```javascript
(function () { return 'hey' })()
```

也可以使用箭头函数：

```javascript
(() => 'hey')()
```

同样地，在匿名函数后添加括号 `()` 来执行函数，这被称为 [自执行函数](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)。

## 闭包

[闭包](https://en.wikipedia.org/wiki/Closure_(computer_programming)) 指的是函数和该函数声明词法环境的组合。结合 [箭头功能](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，我们可以定义如下：

```javascript
var foo = (hi) => (dev) => hi + ' ' + dev
```

在控制台调用上述函数会打印 `hey dev.to`:

```javascript
foo('hey')('dev.to')
```

注意，我们可以在内部函数作用域访问外部函数的参数 `hi`。

以下代码跟上述代码一样：

```javascript
function foo (hi) {
  return function (dev) { return hi + ' ' + dev }
}
```

自执行的版本如下：

```javascript
(
  (hi) =>
    (
      (dev) => `${hi} ${dev}`
    )
    ('dev.to')
)
('hey')
```

首先，将 `hey` 作为参数 `hi` 的值传给最外层作用域的函数，然后这个函数返回另一个自执行函数。`dev.to` 作为参数 `dev` 的值传给内部函数，最后这个函数返回最终值：`'hey dev.to'`。

## 再深入一点

这个一个上述自执行函数的修改版本：

```javascript
(
  (
    (dev) =>
      (hi) => `${hi} ${dev}`
  )
  ('dev.to')
)
('hey')
```

需要注意的是，[自执行函数](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression) 和 [闭包](https://en.wikipedia.org/wiki/Closure_(computer_programming)) 用作初始化和封装状态，接下来我们来看另外一个例子。

## 匿名递归

回到我们最初的例子，这次加点注释：

```javascript
(
  (
    (f) => f(f) // 3.
  )
  (
    (f) => // 2.
      (l) => { // 4.
        console.log(l)
        if (l.length) f(f)(l.slice(1))
        console.log(l)
      }
  )
)
(
  [1, 2, 3] // 1.
)
```

1. 输入函数 `[1, 2, 3]` 传给最外层作用域
2. 整个函数作为参数传给上面函数
3. 这个函数接收下面函数作为参数 `f` 的值，然后自身调用
4. `2.`将被调用被作为 `3.`的结果然后返回函数 `4.` ，该函数是满足最外层作用域的函数，因此接收输入数组作为 `l` 参数的值

至于结果为什么是那样子，原因是在递归内部有一个对函数 `f` 的引用来接收输入数组 `l`。所以能那样调用：

```javascript
f(f)(l.slice(1))
```

注意，`f` 是一个闭包，所以我们只需要调用它就可以访问到操作输入数组的最里面的函数。

为了说明目的，第一个 `console.log(l)` 语句表示递归自上而下，第二个语句表示递归自下而上。

## 结论

希望你喜欢这篇文章，并从中学到了新的东西。闭包、自执行函数、函数式编程模式不是黑魔法。它们遵循一套易于理解和玩乐的简单原则。

话虽如此，你必须培养自己何时使用它们，何时不用的这样一种感觉。如果你的代码变得难以维护，那这可能会成为重构中一些好点子。

然而，理解这些基本技术对于创建清晰优雅的解决方案以及提升自我是至关重要的。

Happy Coding！
