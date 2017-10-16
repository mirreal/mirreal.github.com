---
layout: post
title: "在 Node.js 中使用原生 ES 模块"
date: 2017-09-13 16:19:59 +0800
comments: true
categories: translator
---

> 原文：[Using ES modules natively in Node.js](http://2ality.com/2017/09/native-esm-node.html)
>
> 作者：[Axel Rauschmayer](https://twitter.com/rauschma)


从版本 8.5.0 开始，Node.js 开始支持原生 ES 模块，可以通过命令行选项打开该功能。新功能很大程度上得归功于 [Bradley Farias](https://twitter.com/bradleymeck)。

## 1.演示   

这个示例的代码目录结构如下：

```
esm-demo/
    lib.mjs
    main.mjs
```

lib.mjs：

```javascript
export function add(x, y) {
    return x + y;
}
```

main.mjs：

```javascript
import {add} from './lib.mjs';

console.log('Result: '+add(2, 3));
```

运行演示：

```sh
$ node --experimental-modules main.mjs
Result: 5
```

## 2.清单：需要注意的事情   

### ES 模块：

* 不能动态导入模块。但是 [动态 import()](http://2ality.com/2017/01/import-operator.html) 的相关工作正在进行中，应该很快就能提供支持。

* 没有元变量，如 `__dirname` 和 `__filename`。但是，有一个的类似功能的提案：“[import.meta](https://github.com/tc39/proposal-import-meta)”。看起来可能是这样：

```javascript
console.log(import.meta.url);
```

* 现在所有模块标识符都是 URL（这部分在 Node.js 是新增的）：
    * 文件 - 带文件扩展名的相对路径： `../util/tools.mjs`
    * 库 - 没有文件扩展名，也没有路径 `lodash`
    * 如何更好地使 npm 库在浏览器中也可用（不使用 bundler）仍有待观察。一种可能性是引入 RequireJS 风格的配置数据，将路径映射到实际路径。目前，在浏览器中使用 bare path 的模块标识符是非法的。

### 与 CJS 模块的互操作性

* 你可以导入 CJS 模块，但它们总是只有默认的导出 - 即 `module.exports` 的值。让 CJS 模块支持命名导出已经在做了，但可能需要一段时间。如果你能帮忙，[可以来做](https://twitter.com/bradleymeck/status/906210545145184257)。

```javascript
import fs1 from 'fs';
console.log(Object.keys(fs1).length); // 86

import * as fs2 from 'fs';
console.log(Object.keys(fs2)); // ['default']
```

* 不能在 ES 模块中使用 require()。主要原因是：
    * 路径解析工作稍有不同：ESM 不支持 `NODE_PATH` 和 `require.extensions`。而且，它的标识符始终是 URL 也会导致一些细微差异。
    * ES 模块始终以异步方式加载，这确保了与 Web 的最大兼容性。这种加载风格并不能通过 require() 混合使用同步加载 CJS 模块。
    * 禁止同步模块加载也可以为 Top-level await 导入 ES 模块保留后路（一个当前正在考虑的功能）。


## 3.早期版本的 Node.js 上的 ES 模块   

如果要在 8.5.0 之前的 Node.js 版本上使用 ES 模块，请参阅 John-David Dalton 的 [@std/esm](https://github.com/standard-things/esm)。

提示：如果不启用任何可解锁的额外功能，将在 Node.js 保持 100％ 兼容原生 ES 模块.

## FAQ

### 什么时候可以不带命令行选项使用ES 模块？

目前的计划是在 Node.js 10 LTS 中默认可使用 ES 模块。

## 进一步阅读  

有关 Node.js 和浏览器中 ES 模块的更多信息：

* “[Making transpiled ES modules more spec-compliant](http://2ality.com/2017/01/babel-esm-spec-mode.html)” [using ES modules natively vs. transpiling them via Babel]
* “[Module specifiers: what’s new with ES modules?](http://2ality.com/2017/05/es-module-specifiers.html)” [Why `.mjs`? How are module specifiers resolved? Etc.]
* “[Modules](http://exploringjs.com/es6/ch_modules.html)” [in-depth chapter on ES modules in “Exploring ES6”]

即将到来的 ECMAScript 提案：

* 博客: “[ES proposal: `import()` – dynamically importing ES modules](http://2ality.com/2017/01/import-operator.html)”
* 提案: “[`import.meta`](https://github.com/tc39/proposal-import-meta)”
