---
layout: post
title: "面向初学者的高阶组件介绍"
date: 2017-07-19 18:59:37 +0800
comments: true
categories: translator
---

> 作者：Brandon Newton
>
> 原文：[Higher-Order Components (HOCs) for Beginners](https://btnwtn.com/articles/higher-order-components-for-beginners)
>
> 谈点：一篇面向初学者的 HOC 介绍。高阶组件听起来挺唬人的，只看名字恐怕不是那么容易明白究竟是何物，而且通常来讲高阶组件并不是组件，而是接受组件作为参数，并且返回组件的函数。早期利用 ES5 的 mixin 语法来做的事，基本都可以使用高阶组件代替，而且能做的还有更多。

## 前言

写这篇文章的起因是其他关于高阶组件（Higher-Order Components）的文章，包含官方文档，都令初学者感到相当困惑。我知道有高阶组件这样一个东西，但不知道它到底有什么用。所以，想通过一篇文章来对高阶组件有一个更好的理解。

在此之前，我们需要先来讲一下 JavaScript 中的函数。

## ES6 箭头函数简介

接下来将提供一些箭头函数的简单示例，如果之前没有使用过，可以认为它们与普通函数基本一致。下面的代码会展示箭头函数与普通函数的区别。

```javascript
function () {
  return 42
}

// same as:
() => 42

// same as:
() => {
  return 42
}

function person(name) {
  return { name: name }
}

// same as:
(name) => {
  return { name: name }
}
```

阅读 [MDN 的箭头函数文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 了解更多信息。

## 作为值的函数与部分调用

就像是数字、字符串、布尔值 一样，函数也是值，意味着可以像传递其他数据一样传递函数，可以将函数作为参数传递给另外一个函数。

```javascript
const execute = (someFunction) => someFunction()

execute(() => alert('Executed'))
```

也可以在在函数中返回一个函数：

```javascript
const getOne = () => () => 1

getOne()()
```

之所以在 `getOne` 后面有两个 `()` ，是因为第一个返回的返回值是一个函数。如下：

```javascript
const getOne = () => () => 1

getOne
//=> () => () => 1

getOne()
//=> () => 1

getOne()()
//=> 1
```

从函数返回函数可以帮助我们追踪初始输入函数。例如，下面的函数接受一个数字作为参数，并返回一个将该参数乘以新参数的函数：

```javascript
const multiply = (x) => (y) => x * y

multiply(5)(20)
```

这个示例跟上述 `getOne` 一样，在下面这个例子，让 x = 5，y = 20。

```javascript
const multiply = (x) => (y) => x * y

multiply
//=> (x) => (y) => x * y

multiply(5)
//=> (y) => 5 * y

multiply(5)(20)
//=> 5 * 20
```

在只传入一个参数调用 `multiply` 函数时，即部分调用该函数。比如，`multiply(5)` 讲得到一个将其输入值乘以 5 的函数，`multiply(7)` 将得到一个将其输入值乘以 7 的函数。依此类推。通过部分调用可以创建一个预定义功能的新函数：

```javascript
const multiply = (x) => (y) => x * y

const multiplyByFive = multiply(5)
const multiplyBy100 = multiply(100)

multiplyByFive(20)
//=> 100
multiply(5)(20)
//=> 100

multiplyBy100(5)
//=> 500
multiply(100)(5)
//=> 500
```

一开始看起来似乎没什么用，但是，通过部分调用这种方式可以编写可读性更高，更易于理解的代码。举个例子，可以用一种更清晰的方式来代替 [style-components](https://www.styled-components.com/docs/basics#adapting-based-on-props) 的函数插入语法。

```javascript
// before
const Button = styled.button`
  background-color: ${({ theme }) => theme.bgColor}
  color: ${({ theme }) => theme.textColor}
`

<Button theme={themes.primary}>Submit</Button>
// after
const fromTheme = (prop) => ({ theme }) => theme[prop]

const Button = styled.button`
  background-color: ${fromTheme("bgColor")}
  color: ${fromTheme("textColor")}
`

<Button theme={themes.primary}>Submit</Button>
```

我们创建一个接受一个字符串作为参数的函数 `fromTheme("textColor")`：它返回一个接受具有 `theme` 属性的对象的函数：`({ theme }) => theme[prop]`，然后再通过初始传入的字符串 `"textColor"` 进行查找。我们可以做得更多，写类似的 `backgroundColor` 和 `textColor` 这种部分调用 `fromTheme` 的函数：

```javascript
const fromTheme = (prop) => ({ theme }) => theme[prop]
const backgroundColor = fromTheme("bgColor")
const textColor = fromTheme("textColor")

const Button = styled.button`
  background-color: ${backgroundColor}
  color: ${textColor}
`

<Button theme={themes.primary}>Submit</Button>
```

## 高阶函数

高阶函数的定义是，接受函数作为参数的函数。如果曾经使用过类似 [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 这样的函数，可能已经很熟悉高阶函数。如果不熟悉 [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)，它是一个数组遍历的方法，接受一个函数作为参数应用到数组中的每个元素。例如，可以像这样对一个数组作平方：

```javascript
const square = (x) => x * x

[1, 2, 3].map(square)
//=> [ 1, 4, 9 ]
```

可以实现一个我们自己的 `map` 版本来说明这个概念：

```javascript
const map = (fn, array) => {
  const mappedArray = []

  for (let i = 0; i < array.length; i++) {
    mappedArray.push(
      // apply fn with the current element of the array
      fn(array[i])
    )
  }

  return mappedArray
}
```

然后再使用我们的 map 版本来对一个数组作平方：

```javascript
const square = (x) => x * x

console.log(map(square, [1, 2, 3, 4, 5]))
//=> [ 1, 4, 9, 16, 25 ]
```

> 译者注：我们也可以将 map 方法从对象中解耦出来：
>
> ```javascript
> const map = (fn, array) => Array.prototype.map.call(array, fn)
> ```
> 这样也可以像上述例子一样调用。
> 或者更函数式的做法，再来点柯里化：
>
> ```javascript
> const map = array => fn => Array.prototype.map.call(array, fn)
> ```

或者是返回一个 `<li>`的 React 元素数组：

```javascript
const HeroList = ({ heroes }) => (
  <ul>
    {map((hero) => (
      <li key={hero}>{hero}</li>
    ), heroes)}
  </ul>
)

<HeroList heroes=[
  "Wonder Woman",
  "Black Widow",
  "Spider Man",
  "Storm",
  "Deadpool"
]/>
/*=> (
  <ul>
    <li>Wonder Woman</li>
    <li>Black Widow</li>
    <li>Spider Man</li>
    <li>Storm</li>
    <li>Deadpool</li>
  </ul>
)*/
```

## 高阶组件

我们知道，高阶函数是接受函数作为参数的函数。在 React 中，任何返回 [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) 的函数都被称为无状态函数组件，简称为函数组件。基本的函数组件如下所示：

```javascript
const Title = (props) => <h1>{props.children}</h1>

<Title>Higher-Order Components(HOCs) for React Newbies</Title>
//=> <h1>Higher-Order Components(HOCs) for React Newbies</h1>
```

高阶组件则是*接受组件作为参数并返回组件的函数*。如何使用传入组件完全取决于你，甚至可以完全忽视它：

```javascript
// Technically an HOC
const ignore = (anything) => (props) => <h1>:)</h1>

const IgnoreHeroList = ignore(HeroList)
<IgnoreHeroList />
//=> <h1>:)</h1>
```

可以编写一个将输入转换成大写的 HOC：

```javascript
const yell = (PassedComponent) =>
  ({ children, ...props }) =>
    <PassedComponent {...props}>
      {children.toUpperCase()}!
    </PassedComponent>

const Title = (props) => <h1>{props.children}</h1>
const AngryTitle = yell(Title)

<AngryTitle>Whatever</AngryTitle>
//=> <h1>WHATEVER!</h1>
```

你也可以返回一个有状态组件，因为 JavaScript 中的类不过是函数的语法糖。这样就可以使用到 React 生命周期的方法，比如 `componentDidMount`。这是 HOCs 真正有用的地方。我们现在可以做一些稍微有趣点的事，比如将 HTTP 请求的结果传递给函数组件。

```javascript
const withGists = (PassedComponent) =>
  class WithGists extends React.Component {
    state = {
      gists: []
    }

    componentDidMount() {
      fetch("https://api.github.com/gists/public")
      .then((r) => r.json())
      .then((gists) => this.setState({
        gists: gists
      }))
    }

    render() {
      return (
        <PassedComponent
          {...this.props}
          gists={this.state.gists}
        />
      )
    }
  }


const Gists = ({ gists }) => (
  <pre>{JSON.stringify(gists, null, 2)}</pre>
)

const GistsList = withGists(Gists)

<GistsList />
//=> Before api request finishes:
// <Gists gists={[]} />
//
//=> After api request finishes:
// <Gists gists={[
//  { /* … */ },
//  { /* … */ },
//  { /* … */ }
// ]} />
```

`withGists` 会传递 gist api 调用的结果，并且你可以在任何组件上使用。[点击这里](https://codesandbox.io/embed/o2YpJnpDj) 可以看到一个更加完整的例子。

## 结论：高阶组件是 🔥🔥🔥

react-redux 也是使用 HOC， [connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) 将应用 store 的值传递到“已连接” 的组件。它还会执行一些错误检查和组件生命周期优化，如果手动完成将导致编写大量重复代码。

如果你发现自己在不同地方编写了大量的代码，那么也可以将代码重构成可重用的 HOC。

HOCs 非常具有表现力，可以使用它们创造很多很酷的东西。

尽可能地保持你的 HOC 简单，*不要编写需要阅读长篇大论才能理解的代码*。

### 附加练习

下面有一些练习，来巩固对 HOC 的理解：

* 写一个反转其输入的 HOC
* 编写一个HOC，将 API 中的数据提供给组件
* 写一个HOC来实现 [`shouldComponentUpdate`](https://facebook.github.io/react/docs/react-component.html#shouldcomponentupdate)，[以避免更新](https://facebook.github.io/react/docs/optimizing-performance.html#avoid-reconciliation)。
- 编写一个 HOC，使用 [`React.Children.toArray`](https://facebook.github.io/react/docs/react-api.html#react.children.toarray) 对传入组件子元素进行排序。
