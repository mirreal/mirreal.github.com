---
layout: post
title: "魔幻语言 JavaScript 系列之类型转换、宽松相等以及原始值"
date: 2018-04-12 10:18:01 +0800
comments: true
categories: translator
---

> 编译自：[`[1] + [2] – [3] === 9`!? Looking into assembly code of coercion.](https://wanago.io/2018/04/02/1-2-3-9-looking-into-assembly-code-of-coercion/)
>
> 全文从两个题目来介绍类型转换、宽松相等以及原始值的概念:
>
> [1] + [2] – [3] === 9
>
> 如果让 a == true && a == false 的值为 true
>
> 第二道题目是译者加的，因为这其实是个很好的例子，体现出 JavaScript 的魔幻之处

变量值都具有类型，但仍然可以将一种类型的值赋值给另一种类型，如果是由开发者进行这些操作，就是**类型转换**（显式转换）。如果是发生在后台，比如在尝试对不一致的类型执行操作时，就是**隐式转换**（强制转换）。

## 类型转换（Type casting）

### 基本包装类型（Primitive types wrappers）

在 JavaScript 中除了 `null` 和 `undefined` 之外的所有基本类型都有一个对应的基本包装类型。通过使用其构造函数，可以将一个值的类型转换为另一种类型。

```js
String(123); // '123'
Boolean(123); // true
Number('123'); // 123
Number(true); // 1
```

> 基本类型的包装器不会保存很长时间，一旦完成相应工作，就会消失

需要注意的是，如果在构造函数前使用 `new` 关键字，结果就完全不同，比如下面的例子：

```js
const bool = new Boolean(false);
bool.propertyName = 'propertyValue';
bool.valueOf(); // false

if (bool) {
  console.log(bool.propertyName); // 'propertyValue'
}
```

由于 `bool` 在这里是一个新的对象，已经不再是基本类型值，它的计算结果为 `true`。

上述例子，因为在 if 语句中，括号间的表达式将会装换成布尔值，比如

```js
if (1) {
    console.log(true);
}
```

其实，上面这段代码跟下面一样：

```js
if ( Boolean(1) ) {
    console.log(true);
}
```

### parseFloat

`parseFloat` 函数的功能跟 `Number` 构造函数类似，但对于传参并没有那么严格。当它遇到不能转换成数字的字符，将返回一个到该点的值并忽略其余字符。

```js
Number('123a45'); // NaN
parseFloat('123a45'); // 123
```

### parseInt

`parseInt` 函数在解析时将会对数字进行向下取整，并且可以使用不同的进制。

```js
parseInt('1111', 2); // 15
parseInt('0xF'); // 15

parseFloat('0xF'); // 0
```

`parseInt` 函数可以猜测进制，或着你可以显式地通过第二个参数传入进制，参考 [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt)。

而且不能正常处理大数，所以不应该成为 [**Math.floor**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor) 的替代品，是的，`Math.floor` 也会进行类型转换：

```js
parseInt('1.261e7'); // 1
Number('1.261e7'); // 12610000
Math.floor('1.261e7') // 12610000

Math.floor(true) // 1
```

### toString

可以使用 **toString** 函数将值转换为字符串，但是在不同原型之间的实现有所不同。

**String.prototype.toString**

返回字符串的值

```js
const dogName = 'Fluffy';

dogName.toString() // 'Fluffy'
String.prototype.toString.call('Fluffy') // 'Fluffy'

String.prototype.toString.call({}) // Uncaught TypeError: String.prototype.toString requires that 'this' be a String
```

**Number.prototype.toString**

返回将数字的字符串表示形式，可以指定进制作为第一个参数传入

```js
(15).toString(); // "15"
(15).toString(2); // "1111"
(-15).toString(2); // "-1111"
```

**Symbol .prototype.toString**

返回  `Symbol（${description}）`

**Boolean.prototype.toString**

返回 `“true”` 或 `“false”`

**Object.prototype.toString**

返回一个字符串 `[ object $ { tag } ] ` ，其中 tag 可以是内置类型比如 “Array”，“String”，“Object”，“Date”，也可以是自定义 tag。

```js
const dogName = 'Fluffy';

dogName.toString(); // 'Fluffy' (String.prototype.toString called here)
Object.prototype.toString.call(dogName); // '[object String]'
```

随着 ES6 的推出，还可以使用 [**Symbol**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 进行自定义 tag。

```js
const dog = { name: 'Fluffy' }
console.log( dog.toString() ) // '[object Object]'

dog[Symbol.toStringTag] = 'Dog';
console.log( dog.toString() ) // '[object Dog]'
```

或者

```js
const Dog = function(name) {
  this.name = name;
}
Dog.prototype[Symbol.toStringTag] = 'Dog';

const dog = new Dog('Fluffy');
dog.toString(); // '[object Dog]'
```

还可以结合使用 ES6 class 和 getter：

```js
class Dog {
  constructor(name) {
    this.name = name;
  }
  get [Symbol.toStringTag]() {
    return 'Dog';
  }
}

const dog = new Dog('Fluffy');
dog.toString(); // '[object Dog]'
```

**Array.prototype.toString**

在每个元素上调用 `toString`，并返回一个字符串，并且以逗号分隔。

```js
const arr = [
  {},
  2,
  3
]

arr.toString() // "[object Object],2,3"
```

## 强制转换

如果了解类型转换的工作原理，那么理解强制转换就会容易很多。

### 数学运算符

**加号运算符**

在作为二元运算符的 `+` 如果两边的表达式存在字符串，最后将会返回一个字符串。

```js
'2' + 2 // '22'
15 + '' // '15'
```

可以使用一元运算符将其转换为数字：

```js
+'12' // 12
```

**其他数学运算符**

其他数学运算符（如 `-`或 `/`）将始终转换为数字。

```js
new Date('04-02-2018') - '1' // 1522619999999
'12' / '6' // 2
-'1' // -1
```

上述例子中，Date 类型将转换为数字，即 [Unix 时间戳](https://en.wikipedia.org/wiki/Unix_time)。

### 逻辑非

如果原始值是 *假*，则使用逻辑非将输出 *真*，如果 *真*，则输出为 *假*。 如果使用两次，可用于将该值转换为相应的布尔值。

```js
!1 // false
!!({}) // true
```

### 位或

值得一提的是，即使 ToInt32 实际上是一个抽象操作（仅限内部，不可调用），将一个值转换为一个[有符号的 32 位整数](https://en.wikipedia.org/wiki/32-bit)。

```js
0 | true          // 1
0 | '123'         // 123
0 | '2147483647'  // 2147483647
0 | '2147483648'  // -2147483648 (too big)
0 | '-2147483648' // -2147483648
0 | '-2147483649' // 2147483647 (too small)
0 | Infinity      // 0
```

当其中一个操作数为 0 时执行按位或操作将不改变另一个操作数的值。

### 其他情况下的强制转换

在编码时，可能会遇到更多强制转换的情况，比如这个例子：

```js
const foo = {};
const bar = {};
const x = {};

x[foo] = 'foo';
x[bar] = 'bar';

console.log(x[foo]); // "bar"
```

发生这种情况是因为 `foo` 和 `bar` 在转换为字符串的结果均为 `“[object Object]”`。就像这样：

```js
x[bar.toString()] = 'bar';
x["[object Object]"]; // "bar"
```

使用[模板字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)的时候也会发生强制转换，在下面例子中重写 `toString` 函数：

```js
const Dog = function(name) {
  this.name = name;
}
Dog.prototype.toString = function() {
  return this.name;
}

const dog = new Dog('Fluffy');
console.log(`${dog} is a good dog!`); // "Fluffy is a good dog!"
```

正因为如此，**宽松相等**（==）被认为是一种不好的做法，如果两边类型不一致，就会试图进行强制隐式转换。

看下面这个有趣的例子：

```js
const foo = new String('foo');
const foo2 = new String('foo');

foo === foo2 // false
foo >= foo2 // true
```

在这里我们使用了 `new` 关键字，所以 `foo` 和 `foo2` 都是字符串包装类型，原始值都是 `foo` 。但是，它们现在引用了两个不同的对象，所以 `foo === foo2` 将返回 `false`。这里的关系运算符 `>=` 会在两个操作数上调用 `valueOf` 函数，因此比较的是它们的原始值，`'foo' > = 'foo'` 的结果为 `true`。

## [1] + [2] - [3] === 9

希望这些知识都能帮助揭开这个题目的神秘面纱

1. `[1] + [2]` 将调用 `Array.prototype.toString` 转换为字符串，然后进行字符串拼接。结果将是 `“12”`
   - `[1,2] + [3,4]` 的值讲师 `“1,23,4”`
2. `12 - [3]`，减号运算符会将值转换为 Number 类型，所以等于 `12-3`，结果为 `9`
   - 12 - [3,4] 的值是 `NaN`，因为`"3,4"` 不能被转换为 Number

## 总结

尽管很多人会建议尽量避免强制隐式转换，但了解它的工作原理非常重要，在调试代码和避免错误方面大有帮助。

【译文完】


## 再谈点，关于宽松相等和原始值

这里看另一道题目，在 JavaScript 环境下，能否让表达式 `a == true && a == false` 为 `true`。

就像下面这样，在控制台打印出 `’yeah'`:

```js
// code here
if (a == true && a == false) {
    console.log('yeah');
}
```

关于宽松相等（==），先看看 ECMA 5.1 的规范，包含 `toPrimitive`:

* [11.9.3](http://www.ecma-international.org/ecma-262/5.1/#sec-11.9.3) The Abstract Equality Comparison Algorithm
* [9.1](http://www.ecma-international.org/ecma-262/5.1/#sec-9.1) ToPrimitive

### 稍作总结

规范很长很详细，简单总结就是，对于下述表达式：

```js
x == y
```

* 类型相同，判断的就是 x === y
* 类型不同
    * 如果 x，y 其中一个是布尔值，将这个布尔值进行 ToNumber 操作
    * 如果 x，y 其中一个是字符串，将这个字符串进行 ToNumber 操作
    * 若果 x，y 一方为对象，将这个对象进行 ToPrimitive 操作

至于 `ToPrimitive`，即求原始值，可以简单理解为进行 `valueOf()` 和 `toString()` 操作。

稍后我们再详细剖析，接下来先看一个问题。

### Question：是否存在这样一个变量，满足 x == !x

就像这样：

```js
// code here
if (x == !x) {
    console.log('yeah');
}
```

可能很多人会想到下面这个，毕竟我们也曾热衷于各种奇技淫巧：

```js
[] == ![] // true
```

但答案绝不仅仅局限于此，比如：

```js
var x = new Boolean(false);

if (x == !x) {
    console.log('yeah');
}
// x.valueOf() -> false
// x is a object, so: !x -> false


var y = new Number(0);
y == !y // true
// y.valueOf() -> 0
// !y -> false
// 0 === Number(false) // true
// 0 == false // true
```

理解这个问题，那下面的这些例子都不是问题了：

```js
[] == ![]
[] == {}
[] == !{}
{} == ![]
{} == !{}
```

在来看看什么是 `ToPrimitive`

### ToPrimitive

看规范：[8.12.8](http://www.ecma-international.org/ecma-262/5.1/#sec-8.12.8) `[[DefaultValue]] (hint)`

如果是 `Date` 求原始值，则 hint 是 `String`，其他均为 `Number`，即先调用 `valueOf()` 再调用 `toString()`。

如果 hint 为 `Number`，具体过程如下：

1. 调用对象的 `valueOf()` 方法，如果值是原值则返回
2. 否则，调用对象的 `toString()` 方法，如果值是原值则返回
3. 否则，抛出 TypeError 错误


```js
// valueOf 和 toString 的调用顺序
var a = {
    valueOf() {
        console.log('valueof')
        return []
    },
    toString() {
        console.log('toString')
        return {}
    }
}

a == 0
// valueof
// toString
// Uncaught TypeError: Cannot convert object to primitive value


// Date 类型先 toString，后 valueOf
var t = new Date('2018/04/01');
t.valueOf = function() {
    console.log('valueof')
    return []
}
t.toString = function() {
    console.log('toString')
    return {}
}
t == 0
// toString
// valueof
// Uncaught TypeError: Cannot convert object to primitive value
```

到目前为止，上面的都是 ES5 的规范，那么在 ES6 中，有什么变化呢

### ES6 中 ToPrimitive

[7.1.1](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-toprimitive)ToPrimitive ( input [, PreferredType] )

在 ES6 中吗，是可以自定义 `@@toPrimitive` 方法的，这是 Well-Known Symbols([§6.1.5.1](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-well-known-symbols))中的一个。JavaScript 内建了一些在 ECMAScript 5 之前没有暴露给开发者的 symbol，它们代表了内部语言行为。

来自 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive) 的例子：

```js
// 没有 Symbol.toPrimitive 属性的对象
var obj1 = {};
console.log(+obj1); // NaN
console.log(`${obj1}`); // '[object Object]'
console.log(obj1 + ''); // '[object Object]'

// 拥有 Symbol.toPrimitive 属性的对象
var obj2 = {
    [Symbol.toPrimitive](hint) {
        if (hint == 'number') {
            return 10;
        }
        if (hint == 'string') {
            return 'hello';
        }
        return true;
    }
};
console.log(+obj2); // 10 -- hint is 'number'
console.log(`${obj2}`); // 'hello' -- hint is 'string'
console.log(obj2 + ''); // 'true' -- hint is 'default'
```

有了上述铺垫，答案就呼之欲出了

### `a == true && a == false` 为 `true` 的答案

```js
var a = {
    flag: false,
    toString() {
        return this.flag = !this.flag;
    }
}
```

或者使用 `valueOf()`：

```js
var a = {
    flag: false,
    valueOf() {
        return this.flag = !this.flag;
    }
}
```

或者是直接改变 ToPrimitive 行为：

```js
// 其实只需设置 default 即可
var a = {
    flag: false,
    [Symbol.toPrimitive](hint) {
        if (hint === 'number') {
            return 10
        }
        if (hint === 'string') {
            return 'hello'
        }
        return this.flag = !this.flag
    }
}
```

### 如果是严格相等呢

这个问题在严格相等的情况下，也是能够成立的，这又是另外的知识点了，使用 `defineProperty` 就能实现：

```js
let flag = false
Object.defineProperty(window, 'a', {
    get() {
        return (flag = !flag)
    }
})

if (a === true && a === false) {
    console.log('yeah');
}
```

### 阅读更多

* [Can (a== 1 && a ==2 && a==3) ever evaluate to true?](https://stackoverflow.com/questions/48270127/can-a-1-a-2-a-3-ever-evaluate-to-true)
