---
layout: post
title: "编写扁平化的代码"
date: 2017-11-02 11:20:03 +0800
comments: true
categories: translator
---

> 原文：[Writing flat & declarative code](https://peeke.nl/writing-flat-code)
>
> 作者：[Peeke Kuepers](https://twitter.com/peeke__)

-- *给你的代码增加一点点函数式编程的特性*

**最近我对函数式编程非常感兴趣。这个概念让我着迷：应用数学来增强抽象性和强制纯粹性，以避免副作用，并实现代码的良好可复用性。同时，函数式编程非常复杂。**

函数式编程有一个非常陡峭的学习曲线，因为它来源于数学中的[范畴论](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/ch5.md#category-theory)。接触不久之后，就将遇到诸如组合（composition）、恒等（identity），函子（functor）、单子（monad），以及逆变（contravariant）等术语。我根本不太了解这些概念，可能这也是我从来没有在实践中运用函数式编程的原因。

我开始思考：在常规的命令式编程和完全的函数式编程之间是否可能会有一些中间形式？既允许在代码库引入函数式编程的一些很好的特性，同时暂时保留已有的旧代码。

对我而言，函数式编程最大的作用就是强制你编写声明性代码：代码描述你做什么，而不是在描述如何做。这样就可以轻松了解特定代码块的功能，而无需了解其真正的运行原理。事实证明，编写声明式代码是函数式编程中最简单的部分之一。

## 循环

> ...一个循环就是一个命令式控制结构，难以重用，并且难以插入到其他操作中。此外，它还得不断变化代码来响应新的迭代需求。
>
> -- Luis Atencio

所以，让我们先看一下循环，循环是命令式编程的一个很好的例子。循环涉及很多语法，都是描述它们的行为是如何工作，而不是它们在做什么。例如，看看这段代码：

```javascript
function helloworld(arr) {
    for (let i = 1; i < arr.length; i++) {
        arr[i] *= 2
        if (arr[i] % 2 === 0) {
            doSomething(arr[i])
        }
    }
}
```

这段代码在做什么呢？它将数组内除第一个数字 （`let i = 1`）的其他所有数字乘以 2，如果是偶数的话（`if (arr % 2 === 0)`），就进行某些操作。在此过程中，原始数组的值会被改变。但这通常是不必要的，因为数组可能还会在代码库中的其他地方用到，所以刚才所做的改变可能会导致意外的结果。

但最主要的原因是，这段代码看起来很难一目了然。它是命令式的，for 循环告诉我们如何遍历数组，在里面，使用一个 if 语句有条件地调用一个函数。

我们可以通过使用数组方法以声明式的方式重写这段代码。数组方法直接表达所做的事，比较常见的方法包括：`forEach`，`map`，`filter`，`reduce` 和 `slice`。

结果就像下面这样：

```javascript
function helloworld(arr) {
    const evenNumbers = n => n % 2 === 0

    arr
        .slice(1)
        .map(v => v * 2)
        .filter(evenNumbers)
        .forEach(v => doSomething(v))    
}
```

在这个例子中，我们使用一种很好的，扁平的链式结构去描述我们在做什么，明确表明意图。此外，我们避免了改变原始数组，从而避免不必要的副作用，因为大多数数组方法会返回一个新数组。当箭头函数开始变得越来越复杂时，可以地将其提取到一个特定的函数中，比如 `evenNumbers`， 从而尽量保持结构简单易读。

在上面的例子，链式调用并没有返回值，而是以 forEach 结束。然而，我们可以轻松地剥离最后一部分，并返回结果，以便我们可以在其他地方处理它。如果还需要返回除数组以外的任何东西，可以使用 `reduce` 函数。

对于接下来的一个例子，假设我们有一组 JSON 数据，其中包含在一个虚构歌唱比赛中不同国家获得的积分：

```json
[
    {
        "country": "NL",
        "points": 12
    },
    {
        "country": "BE",
        "points": 3
    },
    {
        "country": "NL",
        "points": 0
    },
    ...
]
```

我们想计算荷兰（NL）获得的总积分，根据印象中其强大的音乐能力，我们可以认为这是一个非常高的分数，但我们想要更精确地确认这一点。

使用循环可能会是这样：

```javascript
function countVotes(votes) {
    let score = 0;

    for (let i = 0; i < votes.length; i++) {
        if (votes[i].country === 'NL') {
            score += votes[i].points;
        }
    }

    return score;
}
```

使用数组方法重构，我们得到一个更干净的代码片段：

```javascript
function countVotes(votes) {
    const sum = (a, b) => a + b;

    return votes
        .filter(vote => vote.country === 'NL')
        .map(vote => vote.points)
        .reduce(sum);
}
```

有时候 `reduce` 可能有点难以阅读，将 `reduce` 函数提取出来会在理解上有帮助。在上面的代码片段中，我们定义了一个 `sum` 函数来描述函数的作用，因此方法链仍然保持很好的可读性。

## if else 语句

接下来，我们来聊聊大家都很喜欢的 if else 语句，if else 语句也是命令式代码里一个很好的例子。为了使我们的代码更具声明式，我们将使用三元表达式。

一个三元表达式是 if else 语句的替代语法。以下两个代码块具有相同的效果：

```javascript
// Block 1
if (condition) {
    doThis();
} else {
    doThat();
}

// Block 2
const value = condition ? doThis() : doThat();
```

当在定义（或返回）一个常量时，三元表达式非常有用。使用 if else 语句会将该变量的使用范围限制在语句内，通过使用三元语句，我们可以避免这个问题：

```javascript
if (condition) {
    const a = 'foo';
} else {
    const a = 'bar';
}

const b = condition ? 'foo' : 'bar';

console.log(a); // Uncaught ReferenceError: a is not defined
console.log(b); // 'bar'
```

现在，我们来看看如何应用这一点来重构一些更重要的代码：

```javascript
const box = element.getBoundingClientRect();

if (box.top - document.body.scrollTop > 0 && box.bottom - document.body.scrollTop < window.innerHeight) {
    reveal();
} else {
    hide();
}
```

那么，上面的代码发生了什么呢？if 语句检查元素当前是否在页面的可见部分内，这个信息在代码的任何地方都没有表达出来。基于此布尔值，再调用 `reveal()` 或者 `hide()` 函数。

将这个 if 语句转换成三元表达式迫使我们将条件移动到它自己的变量中。这样我们可以将三元表达式组合在一行上，现在通过变量的名称来传达布尔值表示的内容，这样还不错。

```javascript
const box = element.getBoundingClientRect();
const isInViewport =
    box.top - document.body.scrollTop > 0 &&
    box.bottom - document.body.scrollTop < window.innerHeight;

isInViewport ? reveal() : hide();
```

通过这个例子，重构带来的好处可能看起来不大。接下来会有一个相比更复杂的例子：

```javascript
elements
    .forEach(element => {
        const box = element.getBoundingClientRect();

        if (box.top - document.body.scrollTop > 0 && box.bottom - document.body.scrollTop < window.innerHeight) {
            reveal();
        } else {
            hide();
        }

    });
```

这很不好，打破了我们优雅的扁平的调用链，从而使代码更难读。我们再次使用三元操作符，而在使用它的时候，使用 `isInViewport` 检查，并跟它自己的动态函数分开。

```javascript
const isInViewport = element => {
    const box = element.getBoundingClientRect();
    const topInViewport = box.top - document.body.scrollTop > 0;
    const bottomInViewport = box.bottom - document.body.scrollTop < window.innerHeight;
    return topInViewport && bottomInViewport;
};

elements
    .forEach(elem => isInViewport(elem) ? reveal() : hide());
```

此外，现在我们将 `isInViewport` 移动到一个独立函数，可以很容易地把它放在它自己的 helper 类/对象之内：

```javascript
import { isInViewport } from 'helpers';

elements
    .forEach(elem => isInViewport(elem) ? reveal() : hide());
```

虽然上面的例子依赖于所处理的是数组，但是在不明确是在数组的情况下，也可以采用这种编码风格。

例如，看看下面的函数，它通过三条规则来验证密码的有效性。

```javascript
import { passwordRegex as requiredChars } from 'regexes'
import { getJson } from 'helpers'

const validatePassword = async value => {
  if (value.length < 6) return false
  if (!requiredChars.test(value)) return false

  const forbidden = await getJson('/forbidden-passwords')
  if (forbidden.includes(value)) return false

  return value
}

validatePassword(someValue).then(persist)
```

如果我们使用数组包装初始值，就可以使用在上面的例子中里面所用到的所有数组方法。此外，我们已经将验证函数打包成 `validationRules` 使其可重用。

```javascript
import { minLength, matchesRegex, notBlacklisted } from 'validationRules'
import { passwordRegex as requiredChars } from 'regexes'
import { getJson } from 'helpers'

const validatePassword = async value => {
  const result = Array.from(value)
    .filter(minLength(6))
    .filter(matchesRegex(requiredChars))
    .filter(await notBlacklisted('/forbidden-passwords'))
    .shift()

  if (result) return result
  throw new Error('something went wrong...')
}

validatePassword(someValue).then(persist)
```

目前在 JavaScript 中有一个 [管道操作符](https://github.com/tc39/proposal-pipeline-operator) 的提案。使用这个操作符，就不用再把原始值换成数组了。可以直接在前面的值调用管道操作符之后的函数，有点像 `Array` 的 `map` 功能。修改之后的代码大概就像这样：

```javascript
import { minLength, matchesRegex, notBlacklisted } from 'validationRules'
import { passwordRegex as requiredChars } from 'regexes'
import { getJson } from 'helpers'

const validatePassword = async value =>
  value
    |> minLength(6)
    |> matchesRegex(requiredChars)
    |> await notBlacklisted('/forbidden-passwords')

try { someValue |> await validatePassword |> persist }
catch(e) {
  // handle specific error, thrown in validation rule
}
```

但需要注意的是，这仍然是一个非常早期的提案，不过可以稍微期待一下。

## 事件

最后，我们来看看事件处理。一直以来，事件处理很难以扁平化的方式编写代码。可以 `Promise` 化来保持一种链式的，扁平化的编程风格，但 `Promise` 只能 `resolve` 一次，而事件绝对会多次触发。

在下面的示例中，我们创建一个类，它对用户的每个输入值进行检索，结果是一个自动补全的数组。首先检查字符串是否长于给定的阈值长度。如果满足条件，将从服务器检索自动补全的结果，并将其渲染成一系列标签。

注意代码的不“纯”，频繁地使用 `this` 关键字。几乎每个函数都在访问 `this` 这个关键字：

> 译注：作者在这里使用 "this keyword"，有一种双关的意味

```javascript
import { apiCall } from 'helpers'

class AutoComplete {

  constructor (options) {

    this._endpoint = options.endpoint
    this._threshold = options.threshold
    this._inputElement = options.inputElement
    this._containerElement = options.list

    this._inputElement.addEventListener('input', () =>
      this._onInput())

  }

  _onInput () {

    const value = this._inputElement.value

    if (value > this._options.threshold) {
      this._updateList(value)
    }

  }

  _updateList (value) {

    apiCall(this._endpoint, { value })
      .then(items => this._render(items))
      .then(html => this._containerElement = html)

  }

  _render (items) {

    let html = ''

    items.forEach(item => {
      html += `<a href="${ item.href }">${ item.label }</a>`
    })

    return html

  }

}
```

通过使用 `Observable`，我们将用一种更好的方式对这段代码进行重写。可以简单将 `Observable` 理解成一个能够多次 `resolve` 的 `Promise`。

> Observable 类型可用于基于推送模型的数据源，如 DOM 事件，定时器和套接字

`Observable` 提案目前处于 Stage-1。在下面 `listen` 函数的实现是从 GitHub 上的[提案](https://github.com/tc39/proposal-observable)中直接复制的，主要是将事件监听器转换成 `Observable`。可以看到，我们可以将整个 `AutoComplete` 类重写为单个方法的函数链。

```javascript
import { apiCall, listen } from 'helpers';
import { renderItems } from 'templates';

function AutoComplete ({ endpoint, threshold, input, container }) {

  listen(input, 'input')
    .map(e => e.target.value)
    .filter(value => value.length >= threshold)
    .forEach(value => apiCall(endpoint, { value }))
    .then(items => renderItems(items))
    .then(html => container.innerHTML = html)

}
```

由于大多数 `Observable` 库的实现过于庞大，我很期待 ES 原生的实现。`map`，`filter` 和 `forEach`方法还不是规范的一部分，但是在 [zen-observable](https://github.com/zenparsing/zen-observable) 已经在扩展 API 实现，而 zen-observable 本身是 ES Observables 的一种实现 。

--

我希望你会对这些“扁平化”模式感兴趣。就个人而言，我很喜欢以这种方式重写我的程序。你接触到的每一段代码都可以更易读。使用这种技术获得的经验越多，就越来越能认识到这一点。记住这个简单的法则：

**The flatter the better!**
