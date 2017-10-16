---
layout: post
title: "CSS Animations vs Web Animations API"
date: 2017-06-26 20:47:08 +0800
author: mirreal
categories: translator
---

> 作者：Ollie Williams
>
> 原文：[CSS Animations vs Web Animations API](https://css-tricks.com/css-animations-vs-web-animations-api/)

在  JavaScript 有一个原生动画 API 叫 Web Animations API，在这篇文章中简称为 WAAPI。MDN 上已经有 [很好的文档](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)，而且，Dan Wilson 为此写了 [一个很棒的文章系列](http://danielcwilson.com/blog/2015/07/animations-part-1/)。

在本文中，我们一起来对比一下 WAAPI 和 CSS 动画。


## 关于浏览器支持

尽管浏览器原生支持仍然有限，但 WAAPI 有一个全面和强大的 [polyfill](https://github.com/web-animations/web-animations-js)，使得现在就能在生产环境使用。

同样地，可以在 [Can I Use](https://github.com/web-animations/web-animations-js) 查看浏览器兼容性数据。然而，这并没有提供很好的信息来支持 WAAPI 的所有子功能。这里有一个检查工具：

See the Pen [WAAPI Browser Support Test](https://codepen.io/danwilson/pen/xGBKVq/) by Dan Wilson ([@danwilson](https://codepen.io/danwilson)) on [CodePen](https://codepen.io/).

要想再没有 polyfill 的情况下体验所有功能，请使用 Firefox Nightly。

##  WAAPI 的基础知识

如果你曾经使用 jQuery  的 `.animate()`，那么应该会觉得 WAAPI 的基本语法看起来很熟悉。

```javascript
var element = document.querySelector('.animate-me');
element.animate(keyframes, 1000);
```

`animate` 方法接受两个参数：关键帧和持续时间。与 jQuery 相比的优势是，不仅是浏览器内置，而且性能也更好。

第一个参数，关键帧，是一个对象数组，每个对象都是动画中的一个关键帧。看这个简单的例子：

```javascript
var keyframes = [
  { opacity: 0 },
  { opacity: 1 }
];
```

第二个参数，持续时间，指的是想要动画持续多久，在上面的例子中是 1000 毫秒。接下来看一个更令人兴奋的例子。

## 用 WAAPI 重新创建一个 animista 的 CSS 动画

这里有一些从 [animista](http://animista.net/) 拉取的 CSS 代码，被称为 slide-in-blurred-top 的入场动画。看起来很漂亮


在 [实际PERF](http://animista.net/play/entrances/slide-in-blurred/slide-in-blurred-top) 比这个 GIF 效果好很多。

以下是 CSS 中的关键帧：

```css
0% {
  transform: translateY(-1000px) scaleY(2.5) scaleX(.2);
  transform-origin: 50% 0;
  filter: blur(40px);
  opacity: 0;
}
100% {
  transform: translateY(0) scaleY(1) scaleX(1);
  transform-origin: 50% 50%;
  filter: blur(0);
  opacity: 1;
}
```

在 WAAPI 中代码基本相同：

```javascript
var keyframes = [
  {
    transform: 'translateY(-1000px) scaleY(2.5) scaleX(.2)',
    transformOrigin: '50% 0',
    filter: 'blur(40px)',
    opacity: 0
  },
  {
    transform: 'translateY(0) scaleY(1) scaleX(1)',
    transformOrigin: '50% 50%',
    filter: 'blur(0)',
    opacity: 1
  }
];
```

可以看出，将关键帧应用到需要动画的元素上是多么容易：

```javascript
element.animate(keyframes, 700);
```

为了简单起见，只指定了持续时间。但是，我们可以使用这个第二个参数来传递更多的选项，至少也应该指定一个缓动效果。以下是所有可用选项的完整列表，其中包含一些示例值：

```javascript
var options = {
  iterations: Infinity,
  iterationStart: 0,
  delay: 0,
  endDelay: 0,
  direction: 'alternate',
  duration: 700,
  fill: 'forwards',
  easing: 'ease-out',
}
element.animate(keyframes, options);
```

加上这些选项，我们的动画将从头开始，没有任何延迟，在动画完成后往返循环播放。

不爽的是，对于熟悉 CSS 动画的人来说，一些术语跟我们习惯的有所不同。好处是打字会更快一些！

* 使用 `easing` 而不是 `animation-timing-function`
* 不是 `animation-iteration-count`，而是 `iterations`。如果我们希望动画永远重复，使用 `Infinity` 而不是 `infinite`。有点混乱， `Infinity` 不带引号。`Infinity` 是一个 JavaScript 关键字，而其他值都是字符串。
* 我们使用毫秒而不是秒，对于之前写过许多 JavaScript 的人来说，这应该是一样的。（你也可以在 CSS 动画中使用毫秒数，但很少有人使用。）

我们来仔细看看一个选项：`iterationStart`。

当我第一次碰到 `iterationStart` 有点困惑。为什么要从指定的迭代开始，而不是只要减少迭代次数？当使用十进制数时，此选项非常有用。例如，可以将其设置为  `.5`，动画将开始一半。要做一个完整的动画需要两个一半，所以如果迭代次数设置为 1，并且将 `iterationStart` 设置为  `.5`，动画将从一半到动画结束播放，然后从动画开头开始，结束于中间！

值得注意的是，也可以将迭代次数设置为小于 1。例如：

```javascript
var option = {
  iterations: .5,
  iterationStart: .5
}
```

这样，动画会从中间开始，一直播放到最后。

endDelay：如果要将多个动画串在一起，但是希望在一个动画的结尾和后续动画的开始之间存在差距，这时 endDelay 就很有用。这是一个有用的视频，由 Patrick Brosset 来解释。

[一个 YouTube 的视频](https://www.youtube.com/embed/hWe-qukNrN8)


## 缓动（easing）

在任何动画中，缓动都是最重要的元素之一。WAAPI 为我们提供了两种不同的方式设置缓动 - 在我们的关键帧数组或我们的选项对象内。

在 CSS 中，如果使用 `animation-timing-function: ease-in-out` 你可能会认为动画会缓慢开始，然后缓慢结束。实际上，这些缓动应用在关键帧之间，而不是整个动画。这可以对动画的感觉进行细粒度的控制。WAAPI 也提供这种功能。

```javascript
var keyframes = [
  { opacity: 0, easing: 'ease-in' },
  { opacity: 0.5, easing: 'ease-out' },
  { opacity: 1 }
]
```

值得注意的是，在 CSS 和 WAAPI 中，不应该传入最后一帧的缓动值，因为这将不起作用。可是很多人会犯这种错误。

有时候，在整个动画中添加缓动效果更为直观。这在 CSS 是不可能的，但现在可以在 WAAPI 中实现。

```javascript
var options = {
  duration: 1000,
  easing: 'ease-in-out',
}
```

可以看到这两种缓动在 CodePen 上的区别：

[点我查看](https://codepen.io/cssgrid/pen/OmrVeQ)

## 缓动 vs 线性

值得注意的是 CSS 动画和 WAAPI 之间的另一个区别：**在 CSS 中 默认值是 ease，而在 WAAPI 默认是 linear。** ease 实际上是 `ease-in-out` 的一个版本，当你想偷懒时这是一个非常好的选择。同时，线性代表致命的沉闷和无生命 - 一致的速度看起来机械和不自然。它被选为默认值，可能因为它是最中立的选项。然而，在使用 WAAPI 时，更好是使用缓动，以免动画看起来很乏味和机械。

## 性能

WAAPI 提供与 CSS 动画相同的性能改进，尽管这并不意味着一定就是平滑的动画。

希望这个 API 的性能优化做到，使我们可以避免使用 `will-change`和 `translateZ` 成为可能。但是，至少在目前的浏览器实现中，这些属性在处理性能问题方面仍然是有帮助，有必要的。

但是，如果你的动画有延迟，则无需担心使用 `will-change`。web animations 规范的主要作者对 [Animation for Work Slack community](https://damp-lake-50659.herokuapp.com/) 提出了一些有趣的建议，希望他不介意我在这里重复：

> 如果有一个正向的延迟，不需要使用 `will-change`，因为浏览器将在延迟开始时进行分层，当动画启动时，它将准备就绪。

## WAAPI 对战 CSS 动画？

WAAPI 为我们提供了一套已经在 CSS 中实现的 JavaScript 语法。然而，它们不应该被视为对手。如果我们坚持使用 CSS 完成动画和转换，那么我们可以在 WAAPI 进行动画交互。

## 动画对象

`.animate()` 方法不仅处理元素的动画，它也返回一些东西。

```javascript
var myAnimation = element.animate(keyframes, options);
```

在控制台中查看的动画对象

如果我们在控制台中查看返回值，会发现这是一个动画对象。这为我们提供了各种各样的功能，其中一些是不言自明，比如 `myAnimation.pause()`。通过更改 `animation-play-state` 属性，我们可以通过 CSS 动画实现类似的结果，但 WAAPI 语法比 `element.style.animationPlayState = "paused"` 更简洁。我们也可以通过 `myAnimation.reverse()` 轻松反转动画，同样地，跟我们使用脚本更改 CSS 的 `animation-direction` 属性相比，稍微有点进步。

然而，到目前为止，使用 JavaScript 操作 `@keyframe` 并不是件容易的的事。即使是重新启动动画这样简单的事，也是需要一些技巧的，就像 Chris Coyier [先前写过的那样](https://css-tricks.com/restart-css-animation/)。使用 WAAPI，我们可以简单地使用 `myAnimation.play()` ，如果动画已经完成，将从一开始就重播动画，或者如果我们暂停播放，则从中间迭代播放动画。

我们甚至可以轻松地改变动画的速度。

```javascript
myAnimation.playbackRate = 2; // speed it up
myAnimation.playbackRate = .4; // use a number less than one to slow it down
```

## getAnimations()

这个方法将返回所有动画对象的数组，包含使用 WAAPI 定义的动画和 CSS 转换或动画。

```javascript
element.getAnimations() // returns any animations or transitions applied to our element using  CSS or WAAPI
```

如果你喜欢使用 CSS 来定义和使用动画，`getAnimations()`  允许 API​​ 与 `@keyframes` 结合使用。你可以继续使用 CSS 进行大部分动画工作，然后在需要 API 时获得使用 API 的优势。

即使一个 DOM 元素只使用到一个动画，`getAnimations()` 也将始终返回一个数组。我们使用那个单一的动画对象来处理。

```javascript
var h2 = document.querySelector("h2");
var myCSSAnimation = h2.getAnimations()[0];
```

我们也可以在 CSS 动画中使用 web animation API :)

```javascript
myCSSAnimation.playbackRate = 4;
myCSSAnimation.reverse();
```

## Promise 和 Event

很多通过 CSS 触发的事件，现在我们已经可以使用 JavaScript 代码来完成：   `animationstart`，`animationend`，`animationiteration` 和 `transitionend`。之前经常需要监听动画或转换的结束，以便从 DOM 中删除应用的元素。

在动画对象可以使用 WAAPI 来完成 `animationend` 或 `transitionend` 做的事情：

```javascript
myAnimation.onfinish = function() {
  element.remove();
}
```

WAAPI 为我们提供了两个选择：event 和 promise。动画对象的 `.finished` 方法会返回一个在动画结束时的 promise。下面这段代码是上面例子的 promise 版本：

```javascript
myAnimation.finished.then(() =>
  element.remove())
```

我们来看看来自 Mozilla 开发者网络中的一个稍微复杂点的例子。`Promise.all` 接受一个 promise 的数组，一旦所有 promise 完成才会运行回调函数。可以看出，`element.getAnimations()` 返回的是一个动画对象数组。我们可以将数组中的所有动画对象 map 到每个动画对象的 `.finished`上，这样就获得需要的 promise 数组。

在这个例子中，只有在页面上的所有动画完成后，我们的函数才能运行。

```javascript
Promise.all(document.getAnimations().map(animation =>
  animation.finished)).then(function() {           
    // do something cool
  })
```

## 未来

本文中提到的功能只是一个开始。从目前的规范和实施来看，未来会有一个很强大动画 API。
