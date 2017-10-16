---
layout: post
title: "ES6 Promise：模式与反模式"
date: 2017-09-29 16:50:11 +0800
comments: true
categories: translator
---

> 原文：[ES6 Promises: Patterns and Anti-Patterns](https://medium.com/datafire-io/es6-promises-patterns-and-anti-patterns-bbb21a5d0918)
> 作者：Bobby Brennan

当几年前，第一次使用 NodeJS 的时候，对现在被称为“ [回调地狱](http://callbackhell.com/) ”的写法感到很困扰。幸运的是，现在是 2017 年了，NodeJS 已经采用大量 JavaScript 的最新特性，从 [v4](http://node.green/) 开始已经支持 Promise。

尽管 Promise 可以让代码更加简洁易读，但对于只熟悉回调函数的人来说，可能对此还是会有所怀疑。在这里，将列出我在使用Promise 时学到的一些基本模式，以及踩的一些坑。

*注意：**在本文中**将使用[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) ，如果你还不是很熟悉，其实很简单，建议[先读一下使用它们的好处](http://exploringjs.com/es6/ch_arrow-functions.html)*

## 模式与最佳实践

#### 使用 Promise

如果使用的是已经支持 Promise 的第三方库，那么使用起来非常简单。只需关心两个函数：`then()` 和 `catch()`。例如，有一个客户端 API 包含三个方法，`getItem()`，`updateItem()`，和`deleteItem()`，每一个方法都返回一个 Promise：

```javascript
Promise.resolve()
  .then(_ => {
    return api.getItem(1)
  })
  .then(item => {
    item.amount++
    return api.updateItem(1, item);
  })
  .then(update => {
    return api.deleteItem(1);
  })
  .catch(e => {
    console.log('error while working on item 1');
  })
```

每次调用 `then()` 会在 Promise 链中创建一个新的步骤，如果链中的任何一个地方出现错误，就会触发接下来的 `catch()` 。`then()` 和 `catch()` 都可以返回一个值或者一个新的 Promise，结果将被传递到 Promise 链的下一个`then()`。

为了比较，这里使用回调函数来实现相同逻辑：

```javascript
api.getItem(1, (err, data) => {
  if (err) throw err;
  item.amount++;
  api.updateItem(1, item, (err, update) => {
    if (err) throw err;
    api.deleteItem(1, (err) => {
      if (err) throw err;
    })
  })
})
```

要注意的第一个区别是，使用回调函数，我们必须在过程的**每个**步骤中进行错误处理，而不是用单个的 catch-all 来处理。回调函数的第二个问题更直观，每个步骤都要水平缩进，而使用 Promise 的代码则有显而易见的顺序关系。

#### 回调函数 Promise 化

需要学习的第一个技巧是如何将回调函数转换为 Promise。你可能正在使用仍然基于回调的库，或是自己的旧代码，不过不用担心，因为只需要几行代码就可以将其包装成一个 Promise。这是将 Node 中的一个回调方法 `fs.readFile` 转换为 Promise的示例：

```javascript
function readFilePromise(filename) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    })
  })
}

readFilePromise('index.html')
  .then(data => console.log(data))
  .catch(e => console.log(e))
```

关键部分是 Promise 构造函数，它接收一个函数作为参数，这个函数有两个函数参数：`resolve` 和 `reject`。在这个函数里完成所有工作，完成之后，在成功时调用 `resolve`，如果有错误则调用 `reject`。

需要注意的是只有**一个**`resolve` 或者 `reject` 被调用，即应该只被调用一次。在我们的示例中，如果 `fs.readFile` 返回错误，我们将错误传递给 `reject`，否则将文件数据传递给`resolve`。

#### Promise 的值

ES6 有两个很方便的辅助函数，用于通过普通值创建 Promise：`Promise.resolve()` 和 `Promise.reject()`。例如，可能需要在同步处理某些情况时一个返回 Promise 的函数：

```javascript
function readFilePromise(filename) {
  if (!filename) {
    return Promise.reject(new Error("Filename not specified"));
  }
  if (filename === 'index.html') {
    return Promise.resolve('<h1>Hello!</h1>');
  }
  return new Promise((resolve, reject) => {/*...*/})
}
```

注意，虽然可以传递任何东西（或者不传递任何值）给 `Promise.reject()`，但是好的做法是传递一个`Error`。

#### 并行运行

`Promise.all`是一个并行运行 Promise 数组的方法，也就是说是同时运行。例如，我们有一个要从磁盘读取文件的列表。使用上面创建的 `readFilePromise` 函数，将如下所示：

```javascript
let filenames = ['index.html', 'blog.html', 'terms.html'];

Promise.all(filenames.map(readFilePromise))
  .then(files => {
    console.log('index:', files[0]);
    console.log('blog:', files[1]);
    console.log('terms:', files[2]);
  })
```

我甚至不会使用传统的回调函数来尝试编写与之等效的代码，那样会很凌乱，而且也容易出错。

#### 串行运行

有时同时运行一堆 Promise 可能会出现问题。比如，如果尝试使用 `Promise.all` 的 API ​​去检索一堆资源，则可能会在达到速率限制时开始响应[429错误](https://httpstatuses.com/429)。

一种解决方案是串行运行 Promise，或一个接一个地运行。但是在 ES6 中没有提供类似 `Promise.all` 这样的方法（为什么？），但我们可以使用 [`Array.reduce`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce?v=b) 来实现：

```javascript
let itemIDs = [1, 2, 3, 4, 5];

itemIDs.reduce((promise, itemID) => {
  return promise.then(_ => api.deleteItem(itemID));
}, Promise.resolve());
```

在这种情况下，我们需要等待每次调用 `api.deleteItem()` 完成之后才能进行下一次调用。这种方法，比为每个 itemID 写 `.then()` 更简洁更通用：

```javascript
Promise.resolve()
  .then(_ => api.deleteItem(1))
  .then(_ => api.deleteItem(2))
  .then(_ => api.deleteItem(3))
  .then(_ => api.deleteItem(4))
  .then(_ => api.deleteItem(5));
```

#### Race

ES6 提供的另一个很方便的函数是 `Promise.race`。跟 `Promise.all` 一样，接收一个 Promise 数组，并同时运行它们，但不同的是，会在一旦**任何** Promise 完成或失败的情况下返回，并放弃所有其他的结果。

例如，我们可以创建一个在几秒钟之后超时的 Promise：

```javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(reject, ms);
  })
}

Promise.race([readFilePromise('index.html'), timeout(1000)])
  .then(data => console.log(data))
  .catch(e => console.log("Timed out after 1 second"))
```

需要注意的是，其他 Promise 仍将继续运行 ，只是看不到结果而已。

#### 捕获错误

捕获错误最常见的方式是添加一个  `.catch()` 代码块，这将捕获前面所有 `.then()` 代码块中的错误  ：

```javascript
Promise.resolve()
  .then(_ => api.getItem(1))
  .then(item => {
    item.amount++;
    return api.updateItem(1, item);
  })
  .catch(e => {
    console.log('failed to get or update item');
  })
```

在这里，只要有 `getItem` 或者 `updateItem` 失败，`catch()`就会被触发。但是如果我们想分开处理 `getItem` 的错误怎么办？只需再插入一个`catch()` 就可以，它也可以返回另一个 Promise。

```javascript
Promise.resolve()
  .then(_ => api.getItem(1))
  .catch(e => api.createItem(1, {amount: 0}))
  .then(item => {
    item.amount++;
    return api.updateItem(1, item);
  })
  .catch(e => {
    console.log('failed to update item');
  })
```

现在，如果`getItem()`失败，我们通过第一个 `catch` 介入并创建一条新的记录。

#### 抛出错误

应该将 `then()` 语句中的所有代码视为 `try` 块内的所有代码。`return Promise.reject()` 和 `throw new Error()` 都会导致下一个 `catch()` 代码块的运行。

这意味着运行时错误也会触发 `catch()`，所以不要去假设错误的来源。例如，在下面的代码中，我们可能希望该 `catch()` 只能获得 `getItem` 抛出的错误，但是如示例所示，它还会在我们的 `then()` 语句中捕获运行时错误。

```javascript
api.getItem(1)
  .then(item => {
    delete item.owner;
    console.log(item.owner.name);
  })
  .catch(e => {
    console.log(e); // Cannot read property 'name' of undefined
  })
```

#### 动态链

有时，我们想要动态地构建 Promise 链，例如，在满足特定条件时，插入一个额外的步骤。在下面的示例中，在读取给定文件之前，我们可以选择创建一个锁定文件：

```javascript
function readFileAndMaybeLock(filename, createLockFile) {
  let promise = Promise.resolve();

  if (createLockFile) {
    promise = promise.then(_ => writeFilePromise(filename + '.lock', ''))
  }

  return promise.then(_ => readFilePromise(filename));
}
```

一定要通过重写 `promise = promise.then(/*...*/)` 来更新 `Promise` 的值。参看接下来反模式中会提到的 **多次调用 then()**。

## 反模式

Promise 是一个整洁的抽象，但很容易陷入某些陷阱。以下是我遇到的一些最常见的问题。

#### 重回回调地狱

当我第一次从回调函数转到 Promise 时，发现很难摆脱一些旧习惯，仍像使用回调函数一样嵌套 Promise：

```javascript
api.getItem(1)
  .then(item => {
    item.amount++;
    api.updateItem(1, item)
      .then(update => {
        api.deleteItem(1)
          .then(deletion => {
            console.log('done!');
          })
      })
  })
```

这种嵌套是完全没有必要的。有时一两层嵌套可以帮助组合相关任务，但是最好总是使用 `.then()` 重写成 Promise 垂直链  。

#### 没有返回

我遇到的一个经常会犯的错误是在一个 Promise 链中忘记 `return` 语句。你能发现下面的 bug 吗？

```javascript
api.getItem(1)
  .then(item => {
    item.amount++;
    api.updateItem(1, item);
  })
  .then(update => {
    return api.deleteItem(1);
  })
  .then(deletion => {
    console.log('done!');
  })
```

因为我们没有在第4行的 `api.updateItem()` 前面写 `return`，所以 `then()` 代码块会立即 resolove，导致 `api.deleteItem()` 可能在` api.updateItem()` 完成之前就被调用。

在我看来，这是 ES6 Promise 的一个大问题，往往会引发意想不到的行为。问题是，  `.then()` 可以返回一个值，也可以返回一个新的 Promise，`undefined` 完全是一个有效的返回值。就个人而言，如果我负责 Promise API，我会在 `.then()` 返回   `undefined` 时抛出运行时错误，但现在我们需要特别注意 `return` 创建的 Promise。

#### 多次调用 `.then()`

根据规范，在同一个 Promise 上多次调用 `then()` 是完全有效的，并且回调将按照其注册顺序被调用。但是，我并未见过需要这样做的场景，并且在使用返回值和错误处理时可能会产生一些意外行为：

```javascript
let p = Promise.resolve('a');
p.then(_ => 'b');
p.then(result => {
  console.log(result) // 'a'
})

let q = Promise.resolve('a');
q = q.then(_ => 'b');
q = q.then(result => {
  console.log(result) // 'b'
})
```

在这个例子中，因为我们在每次调用 `then()` 不更新 `p` 的值，所以我们看不到 `'b'` 返回。但是每次调用 `then()` 时更新 `q`，所以其行为更可预测。

这也适用于错误处理：

```javascript
let p = Promise.resolve();
p.then(_ => {throw new Error("whoops!")})
p.then(_ => {
  console.log('hello!'); // 'hello!'
})

let q = Promise.resolve();
q = q.then(_ => {throw new Error("whoops!")})
q = q.then(_ => {
  console.log('hello'); // We never reach here
})
```

在这里，我们期望的是抛出一个错误来打破 Promise 链，但由于没有更新 `p` 的值，所以第二个 `then()` 仍会被调用。

有可能在一个 Promise 上多次调用 `.then()` 有很多理由  ，因为它允许将 Promise 分配到几个新的独立的 Promise 中，但是还没发现真实的使用场景。

#### 混合使用回调和 Promise

很容易进入一种陷阱，在使用基于 Promise 库的同时，仍在基于回调的项目中工作。始终避免在 `then()` 或 `catch()` 使用回调函数 ，否则 Promise 会吞噬任何后续的错误，将其作为 Promise 链的一部分。例如，以下内容看起来是一个挺合理的方式，使用回调函数来包装一个 Promise：

```javascript
function getThing(callback) {
  api.getItem(1)
    .then(item => callback(null, item))
    .catch(e => callback(e));
}

getThing(function(err, thing) {
  if (err) throw err;
  console.log(thing);
})
```

这里的问题是，如果有错误，我们会收到关于“Unhandled promise rejection”的警告，即使我们添加了一个 `catch()` 代码块。这是因为，`callback()` 在 `then()` 和 `catch()` 都会被调用，使之成为 Promise 链的一部分。

如果必须使用回调来包装 Promise，可以使用  `setTimeout` （或者是 NodeJS 中的 `process.nextTick`）来打破 Promise：

```javascript
function getThing(callback) {
  api.getItem(1)
    .then(item => setTimeout(_ => callback(null, item)))
    .catch(e => setTimeout(_ => callback(e)));
}

getThing(function(err, thing) {
  if (err) throw err;
  console.log(thing);
})
```

#### 不捕获错误

JavaScript 中的错误处理有点奇怪。虽然支持熟悉的 `try/catch` 范例，但是没有办法强制调用者以 Java 的方式处理错误。然而，使用回调函数，使用所谓的“errbacks”，即第一个参数是一个错误回调变得很常见。这迫使调用者至少承认错误的可能性。例如，`fs` 库：

```javascript
fs.readFile('index.html', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
})
```

使用 Promise，又将很容易忘记需要进行错误处理，特别是对于敏感操作（如文件系统和数据库访问）。目前，如果没有捕获到 reject 的 Promise，将在 NodeJS 中看到非常丑的警告：

```text
(node:29916) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: whoops!
(node:29916) DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

确保在主要的事件循环中任何 Promise 链的末尾添加 `catch()` 以避免这种情况。

## 总结

希望这是一篇有用的关于常见 Promise 模式和反模式的概述。如果你想了解更多，这里有一些有用的资源：

* [Mozilla 的 ES6 Promise 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* [来自 Google 的 Promise 介绍](https://developers.google.com/web/fundamentals/getting-started/primers/promises)
* [Dave Atchley 的 ES6 Promise 概述](http://www.datchley.name/es6-promises/)

更多的 Promise [模式](http://www.datchley.name/promise-patterns-anti-patterns/)和[反模式](https://hackernoon.com/javascript-promises-best-practices-anti-patterns-b32309f65551)

[或者阅读来自 DataFire 团队的内容](https://medium.com/datafire-io)
