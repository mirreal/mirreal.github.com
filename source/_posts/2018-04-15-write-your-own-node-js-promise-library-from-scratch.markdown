---
layout: post
title: "从零开始写一个 Promise 库"
date: 2018-04-15 12:15:14 +0800
comments: true
categories: translator
---

> 原文：[Write Your Own Node.js Promise Library from Scratch](http://thecodebarbarian.com/write-your-own-node-js-promise-library-from-scratch.html)
>
> 作者：[code_barbarian](https://twitter.com/code_barbarian)


[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 已经是 JavaScript 中异步处理的基石，[回调](http://thecodebarbarian.com/2015/03/20/callback-hell-is-a-myth)的场景将会越来越少，而且现在[可以直接在 Node.js 使用 async/await](http://thecodebarbarian.com/common-async-await-design-patterns-in-node.js.html)。async/await 基于 Promise，因此需要了解 Promise 来掌握 async/await。这篇文章，将介绍如何编写一个 Promise 库，并演示如何使用 async/await。

## Promise 是什么？

在 ES6 规范中，[Promise 是一个类](http://www.ecma-international.org/ecma-262/6.0/#sec-promise-executor)，它的构造函数接受一个 `executor` 函数。Promise 类的实例有一个 [`then()` 方法](http://www.ecma-international.org/ecma-262/6.0/#sec-promise.prototype.then)。根据规范，Promise 还有其他的一些属性，但在这里可以暂时忽略，因为我们要实现的是一个精简版的库。下面是一个 `MyPromise` 类的脚手架：

```
class MyPromise {
    // `executor` 函数接受两个参数，`resolve()` 和 `reject()`
    // 负责在异步操作成功(resolved)或者失败(rejected)的时候调用 `resolve()` 或者 `reject()`
    constructor(executor) {}

    // 当 promise 的状态是 fulfilled（完成）时调用 `onFulfilled` 方法，
    // 当 promise 的状态是 rejected（失败）时调用 `onRejected` 方法
    // 到目前为止，可以认为 'fulfilled' 和 'resolved' 是一样的
    then(onFulfilled, onRejected) {}
}
```

`executor` 函数需要两个参数，`resolve()` 和 `reject()`。promise 是一个状态机，包含三个状态：

* pending：初始状态，既不是成功，也不是失败状态
* fulfilled：意味着操作成功完成，返回结果值
* rejected：意味着操作失败，返回错误信息

这样很容易就能实现 `MyPromise` 构造函数的初始版本：

```
constructor(executor) {
    if (typeof executor !== 'function') {
        throw new Error('Executor must be a function')
    }

    // 初始状态，$state 表示 promise 的当前状态
    // $chained 是当 promise 处在 settled 状态时需要调用的函数数组
    this.$state = 'PENDING'
    this.$chained = []

    // 为处理器函数实现 `resolve()` 和 `reject()`
    const resolve = res => {
        // 只要当 `resolve()` 或 `reject()` 被调用
        // 这个 promise 对象就不再处于 pending 状态，被称为 settled 状态
        // 调用 `resolve()` 或 `reject()` 两次，以及在 `resolve()` 之后调用 `reject()` 是无效的
        if (this.$state !== 'PENDING') {
            return
        }
        // 后面将会谈到 fulfilled 和 resolved 之间存在细微差别
        this.$state = 'FULFILLED'
        this.$internalValue = res
        // If somebody called `.then()` while this promise was pending, need
        // to call their `onFulfilled()` function
        for (const { onFulfilled } of this.$chained) {
            onFulfilled(res)
        }
    }
    const reject = err => {
        if (this.$state !== 'PENDING') {
            return
        }
        this.$state = 'REJECTED'
        this.$internalValue = err
        for (const { onRejected } of this.$chained) {
            onRejected(err)
        }
    }

    // 如规范所言，调用处理器函数中的 `resolve()` 和 `reject()`
    try {
        // 如果处理器函数抛出一个同步错误，我们认为这是一个失败状态
        // 需要注意的是，`resolve()` 和 `reject()` 只能被调用一次
        executor(resolve, reject)
    } catch (err) {
        reject(err)
    }
}
```

`then()` 函数的实现更简单，它接受两个参数，`onFulfilled()` 和 `onRejected()`。`then()` 函数必须确保 promise 在 fulfilled 时调用 `onFulfilled()`，在 rejected 时调用 `onRejected()`。如果 promise 已经 resolved 或 rejected，`then()` 函数会立即调用 `onFulfilled()` 或 `onRejected()`。如果 promise 仍处于 pending 状态，就将函数推入 `$chained` 数组，因此后续 `resolve()` 和 `reject()` 函数仍然可以调用它们。

```
then(onFulfilled, onRejected) {
    if (this.$state === 'FULFILLED') {
        onFulfilled(this.$internalValue)
    } else if (this.$state === 'REJECTED') {
        onRejected(this.$internalValue)
    } else {
        this.$chained.push({ onFulfilled, onRejected })
    }
}
```

**除此之外：ES6 规范表示，如果在已经 resolved 或 rejected 的 promise 调用 `.then()`, 那么 `onFulfilled()` 或 `onRejected()` 将在下一个时序被调用。由于本文代码只是一个教学示例而不是规范的精确实现，因此实现会忽略这些细节。*

## Promise 调用链

上面的例子特意忽略了 promise 中最复杂也是最有用的部分：[链式调用](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises#Chaining)。如果 `onFulfilled()` 或者 `onRejected()` 函数返回一个 promise，则 `then()` 应该返回一个 [“locked in”](http://www.ecma-international.org/ecma-262/6.0/#sec-promise-objects) 的新 promise 以匹配这个 promise 的状态。例如：

```
p = new MyPromise(resolve => {
    setTimeout(() => resolve('World'), 100)
})

p
    .then(res => new MyPromise(resolve => resolve(`Hello, ${res}`)))
    // 在 100 ms 后打印 'Hello, World'
    .then(res => console.log(res))
```

下面是可以返回 promise 的 `.then()` 函数实现，这样就可以进行链式调用。

```
then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
        // 确保在 `onFulfilled()` 和 `onRejected()` 的错误将导致返回的 promise 失败（reject)
        const _onFulfilled = res => {
            try {
                // 如果 `onFulfilled()` 返回一个 promise, 确保 `resolve()` 能正确处理
                resolve(onFulfilled(res))
            } catch (err) {
                reject(err)
            }
        }
        const _onRejected = err => {
            try {
                reject(onRejected(err))
            } catch (_err) {
                reject(_err)
            }
        }
        if (this.$state === 'FULFILLED') {
            _onFulfilled(this.$internalValue)
        } else if (this.$state === 'REJECTED') {
            _onRejected(this.$internalValue)
        } else {
            this.$chained.push({ onFulfilled: _onFulfilled, onRejected: _onRejected })
        }
    })
}
```

现在 `then()` 返回一个 promise，但是还需要完成一些工作：如果 `onFulfilled()` 返回一个 promise，`resolve()` 要能够正确处理。所以 `resolve()` 函数需要在 `then()` 递归调用，下面是更新后的 `resolve()` 函数：

```
const resolve = res => {
    // 只要当 `resolve()` 或 `reject()` 被调用
    // 这个 promise 对象就不再处于 pending 状态，被称为 settled 状态
    // 调用 `resolve()` 或 `reject()` 两次，以及在 `resolve()` 之后调用 `reject()` 是无效的
    if (this.$state !== 'PENDING') {
        return
    }

    // 如果 `res` 是 thenable（带有then方法的对象）
    // 将锁定 promise 来保持跟 thenable 的状态一致
    if (res !== null && typeof res.then === 'function') {
        // 在这种情况下，这个 promise 是 resolved，但是仍处于 'PENDING' 状态
        // 这就是 ES6 规范中说的"一个 resolved 的 promise"，可能处在 pending, fulfilled 或者 rejected 状态
        // http://www.ecma-international.org/ecma-262/6.0/#sec-promise-objects
        return res.then(resolve, reject)
    }

    this.$state = 'FULFILLED'
    this.$internalValue = res
    // If somebody called `.then()` while this promise was pending, need
    // to call their `onFulfilled()` function
    for (const { onFulfilled } of this.$chained) {
        onFulfilled(res)
    }

    return res
}
```

*为了简单起见，上面的例子省略了一旦 promise 被锁定用以匹配另一个 promise 时，调用 resolve() 或者 reject() 是无效的关键细节。在上面的例子中，你可以 resolve() 一个 pending 的 promise ，然后抛出一个错误，然后 res.then(resolve, reject) 将会无效。这仅仅是一个例子，而不是 ES6 promise 规范的完全实现。*

上面的代码说明了 resolved 的 promise 和 fulfilled 的 promise 之间的区别。这种区别是微妙的，并且与 promise 链式调用有关。resolved 不是一种真正的 promise 状态，但它是[ES6规范中定义](http://www.ecma-international.org/ecma-262/6.0/#sec-promise-objects)的[术语](http://www.ecma-international.org/ecma-262/6.0/#sec-promise-objects)。当对一个已经 resolved 的 promise 调用 `resolve()`，可能会发生以下两件事之一：

* 在调用 `resolve(v)`时，如果 `v` 不是一个 promise ，那么 promise 立即成为 fulfilled。在这种简单的情况下，resolved 和 fulfilled 就是一样的。
* 在调用 `resolve(v)`时，如果 `v` 是另一个 promise，那么这个 promise 一直处于 pending 直到 `v` 调用 resolve 或者 reject。在这种情况下， promise 是 resolved 但处于 pending 状态。

## 与 Async/Await 一起使用

关键字 `await` 会暂停执行一个 `async` 函数，直到等待的 promise 变成 settled 状态。现在我们已经有了一个简单的自制 promise 库，看看结合使用 async/await 中时会发生什么。向 `then()` 函数添加一个 `console.log()` 语句：

```
then(onFulfilled, onRejected) {
    console.log('Then', onFulfilled, onRejected, new Error().stack)
    return new MyPromise((resolve, reject) => {
        /* ... */
    })
}
```

现在，我们来 `await` 一个 `MyPromise` 的实例，看看会发生什么。

```
run().catch(error => console.error(error.stack))

async function run() {
    const start = Date.now()
    await new MyPromise(resolve => setTimeout(() => resolve(), 100))
    console.log('Elapsed time', Date.now() - start)
}
```

注意上面的 `.catch()` 调用。[`catch()` 函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)是 ES6 promise 规范的核心部分。本文不会详细讲述它，因为 `.catch(f)` 相当于 `.then(null, f)`，没有什么特别的内容。

以下是输出内容，注意 await 隐式调用 `.then()` 中的 `onFulfilled()` 和 `onRejected()` 函数，这是 V8 底层的 C++ 代码（native code）。此外，`await` 会一直等待调用 `.then()` 直到下一个时序。

```
Then function () { [native code] } function () { [native code] } Error
    at MyPromise.then (/home/val/test/promise.js:63:50)
    at process._tickCallback (internal/process/next_tick.js:188:7)
    at Function.Module.runMain (module.js:686:11)
    at startup (bootstrap_node.js:187:16)
    at bootstrap_node.js:608:3
Elapsed time 102
```

## 更多

[async/await](http://thecodebarbarian.com/common-async-await-design-patterns-in-node.js.html) 是非常强大的特性，但掌握起来稍微有点困难，因为需要使用者了解 promise 的基本原则。 promise 有很多细节，例如捕获处理器函数中的同步错误，以及 promise 一旦解决就无法改变状态，这使得 async/await 成为可能。一旦对 promise 有了充分的理解，async/await 就会变得容易得多。
