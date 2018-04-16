---
layout: post
title: "React 整洁代码最佳实践"
date: 2017-12-01 08:53:45 +0800
comments: true
categories: translator
---

> 原文：[Clean Code vs. Dirty Code: React Best Practices](http://americanexpress.io/clean-code-dirty-code/)
>
> 作者：Donavon West


本文主要介绍了适用于现代 React 软件开发的整洁代码实践，顺便谈谈 ES6/ES2015 带来的一些好用的“语法糖”。

## 什么是整洁代码，为什么要在乎？

整洁代码代表的是一种一致的编码风格，目的是让代码更易于编写，阅读和维护。通常情况下，开发者在解决问题的时候，一旦问题解决就发起一个 Pull Request（译注：合并请求，在 Gitlab 上叫 Merge Request）。但我认为，这时候工作并没有真正完成，我们不能仅仅满足于代码可以工作。

这时候其实就是整理代码的最好时机，可以通过删除死代码（僵尸代码），重构以及删除注释掉的代码，来保持代码的可维护性。不妨问问自己，“从现在开始再过六个月，其他人还能理解这些代码吗？”简而言之，对于自己编写的代码，你应该保证能很自豪地拿给别人看。

至于为什么要在乎这点？因为我们常说一个优秀的开发者大都比较”懒“。在遇到需要重复做某些事情的情况下，他们会去找到一个自动化（或更好的）解决方案来完成这些任务。

### 整洁代码能够通过“味道测试”

整洁代码应该可以通过“味道测试”。什么意思呢？我们在看代码的时候，包括我们自己写的或或是别人的，会说：“这里不太对劲。”如果感觉不对，那可能就真的是有问题的。如果你觉得你正在试图把一个方形钉子装进一个圆形的洞里，那么就暂停一下，然后休息一下。多次尝试之后，你会找到一个更好的解决方案。

### 整洁代码是符合 DRY 原则的

DRY 是一个缩略词，意思是“不要重复自己”（Don’t Repeat Yourself）。如果发现多个地方在做同样的事情，那么这时候就应该合并重复代码。如果在代码中看到了模式，那么表明需要实行 DRY。

```js
// Dirty
const MyComponent = () => (
  <div>
    <OtherComponent type="a" className="colorful" foo={123} bar={456} />
    <OtherComponent type="b" className="colorful" foo={123} bar={456} />    
  </div>
);
```

```js
// Clean
const MyOtherComponent = ({ type }) => (
  <OtherComponent type={type} className="colorful" foo={123} bar={456} />
);
const MyComponent = () => (
  <div>
    <MyOtherComponent type="a" />
    <MyOtherComponent type="b" />
  </div>
);
```

有时候，比如在上面的例子中，实行 DRY 原则反而可能会增加代码量。但是，DRY 通常也能够提高代码的可维护性。

注意，很容易陷入过分使用 DRY 原则的陷阱，应该学会适可而止。

### 整洁代码是可预测和可测试的

编写单元测试不仅仅只是一个好想法，而且应该是强制性的。不然，怎么能确保新功能不会在其他地方引起 Bug 呢？

许多 React 开发人员选择 [Jest](https://facebook.github.io/jest/) 作为一个零配置测试运行器，然后生成代码覆盖率报告。如果对测试前后对比可视化感兴趣，请查看美国运通的 [Jest Image snanshot](https://github.com/americanexpress/jest-image-snapshot)。

### 整洁代码是自注释的

以前发生过这种情况吗？你写了一些代码，并且包含详细的注释。后来你发现一个 bug，于是回去修改代码。但是，你有没有改变注释来体现新的逻辑？也许会，也许不会。下一个看你代码的人可能因为注意到这些注释而掉进一个陷阱。

注释只是为了解释复杂的想法，也就是说，不要对显而易见的代码进行注释。同时，更少的注释也减少了视觉上的干扰。

```js
// Dirty
const fetchUser = (id) => (
  fetch(buildUri`/users/${id}`) // Get User DTO record from REST API
    .then(convertFormat) // Convert to snakeCase
    .then(validateUser) // Make sure the the user is valid
);
```

在整洁代码的版本中，我们对一些函数进行重命名，以便更好地描述它们的功能，从而消除注释的必要性，减少视觉干扰。并且避免后续因代码与注释不匹配导致的混淆。

```js
// Clean
const fetchUser = (id) => (
  fetch(buildUri`/users/${id}`)
    .then(snakeToCamelCase)
    .then(validateUser)
);
```

### 命名

在我之前的文章 [将函数作为子组件是一种反模式](http://americanexpress.io/faccs-are-an-antipattern)，强调了命名的重要性。每个开发者都应该认真考虑变量名，函数名，甚至是文件名。

这里列举一下命名原则：

- 布尔变量或返回布尔值的函数应该以“is”，“has”或“should”开头。

  ```js
  // Dirty
  const done = current >= goal;
  ```

  ```js
  // Clean
  const isComplete = current >= goal;
  ```

- 函数命名应该体现做了什么，而不是是怎样做的。换言之，不要在命名中体现出实现细节。假如有天出现变化，就不需要因此而重构引用该函数的代码。比如，今天可能会从 REST API 加载配置，但是可能明天就会将其直接写入到 JavaScript 中。

  ```js
  // Dirty
  const loadConfigFromServer = () => {
    ...
  };
  ```

  ```js
  // Clean
  const loadConfig = () => {
    ...
  };
  ```

### 整洁代码遵循成熟的设计模式和最佳实践

计算机已经存在很长一段时间了。多年以来，程序员通过解决某些特定问题，发现了一些固有套路，被称为设计模式。换言之，有些算法已经被证明是可以工作的，所以应该站在前人的肩膀上，避免犯同样的错误。

那么，什么是最佳实践，与设计模式类似，但是适用范围更广，不仅仅针对编码算法。比如，“应该对代码进行静态检查”或者“当编写一个库时，应该将 React 作为 `peerDependency`”，这些都可以称为最佳实践。

构建 React 应用程序时，应该遵循以下最佳实践：

- 使用小函数，每个函数具备单一功能，即所谓的单一职责原则（Single responsibility principle）。确保每个函数都能完成一项工作，并做得很好。这样就能将复杂的组件分解成许多较小的组件。同时，将具备更好的可测试性。
- 小心抽象泄露（leaky abstractions）。换言之，不要强迫消费方去了解内部代码实现细节。
- 遵循严格的代码检查规则。这将有助于编写整洁，一致的代码。

### 整洁代码不需要花长时间来编写

总会听到这样的说法：编写整洁代码会降低生产力。简直是在胡说八道。是的，可能刚开始需要放慢速度，但最终会随着编写更少的代码而节奏加快。

而且，不要小看代码评审导致的重写重构，以及修复问题花费的时间。如果把代码分解成小的模块，每个模块都是单一职责，那么很可能以后再也不用去碰大多数模块了。时间就省下来了，也就是说 “write it and forget it”。

## 槽糕代码与整洁代码的实例

### 使用 DRY 原则

看看下面的代码示例。如上所述，从你的显示器退后一步，发现什么模式了吗？注意 `Thingie` 组件与 `ThingieWithTitle` 组件除了 `Title` 组件几乎完全相同，这是实行 DRY 原则的最佳情形。

```js
// Dirty
import Title from './Title';

export const Thingie = ({ description }) => (
  <div class="thingie">
    <div class="description-wrapper">
      <Description value={description} />
    </div>
  </div>
);

export const ThingieWithTitle = ({ title, description }) => (
  <div>
    <Title value={title} />
    <div class="description-wrapper">
      <Description value={description} />
    </div>
  </div>
);
```

在这里，我们将 `children` 传递给 `Thingie`。然后创建 `ThingieWithTitle`，这个组件包含 `Thingie`，并将 `Title` 作为其子组件传给 `Thingie`。

```js
// Clean
import Title from './Title';

export const Thingie = ({ description, children }) => (
  <div class="thingie">
    {children}
    <div class="description-wrapper">
      <Description value={description} />
    </div>
  </div>
);

export const ThingieWithTitle = ({ title, ...others }) => (
  <Thingie {...others}>
    <Title value={title} />
  </Thingie>
);
```

### 默认值

看看下面的代码。使用逻辑或将 `className` 的默认值设置成 “icon-large”，看起来像是上个世纪的人才会写的代码。

```js
// Dirty
const Icon = ({ className, onClick }) => {
  const additionalClasses = className || 'icon-large';

  return (
    <span
      className={`icon-hover ${additionalClasses}`}
      onClick={onClick}>
    </span>
  );
};
```

这里我们使用 ES6 的默认语法来替换 `undefined` 时的值，而且还能使用 ES6 的箭头函数表达式写成单一语句形式，从而去除对 `return` 的依赖。

```js
// Clean
const Icon = ({ className = 'icon-large', onClick }) => (
  <span className={`icon-hover ${className}`} onClick={onClick} />
);
```

在下面这个更整洁的版本中，使用 React 中的 API 来设置默认值。

```js
// Cleaner
const Icon = ({ className, onClick }) => (
  <span className={`icon-hover ${className}`} onClick={onClick} />
);

Icon.defaultProps = {
  className: 'icon-large',
};
```

为什么这样显得更加整洁？而且它真的会更好吗？三个版本不是都在做同样的事情吗？某种意义上来说，是对的。让 React 设置 prop 默认值的好处是，可以产生更高效的代码，而且在基于 `Class` 的生命周期组件中允许通过 `propTypes` 检查默认值。还有一个优点是：将默认逻辑从组件本身抽离出来。

例如，你可以执行以下操作，将所有默认属性放到一个地方。当然，并不是建议你这样做，只是说具有这样的灵活性。

```js
import defaultProps from './defaultProps';
// ...
Icon.defaultProps = defaultProps.Icon;
```

### 从渲染分离有状态的部分

将有状态的数据加载逻辑与渲染逻辑混合可能增加组件复杂性。更好的方式是，写一个负责完成数据加载的有状态的容器组件，然后编写另一个负责显示数据的组件。这被称为 [容器模式](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)。

在下面的示例中，用户数据加载和显示功能放在一个组件中。

```js
// Dirty
class User extends Component {
  state = { loading: true };

  render() {
    const { loading, user } = this.state;
    return loading
      ? <div>Loading...</div>
      : <div>
          <div>
            First name: {user.firstName}
          </div>
          <div>
            First name: {user.lastName}
          </div>
          ...
        </div>;
  }

  componentDidMount() {
    fetchUser(this.props.id)
      .then((user) => { this.setState({ loading: false, user })})
  }
}
```

在整洁版本中，加载数据和显示数据已经分离。这不仅使代码更容易理解，而且能减少测试的工作量，因为可以独立测试每个部分。而且由于 `RenderUser` 是一个无状态组件，所以结果是可预测的。

```js
// Clean
import RenderUser from './RenderUser';

class User extends Component {
  state = { loading: true };

  render() {
    const { loading, user } = this.state;
    return loading ? <Loading /> : <RenderUser user={user} />;
  }

  componentDidMount() {
    fetchUser(this.props.id)
      .then(user => { this.setState({ loading: false, user })})
  }
}
```

### 使用无状态组件

React v0.14.0 中引入了无状态函数组件（SFC），被简化成纯渲染组件，但有些开发者还在使用过去的方式。例如，以下组件就应该转换为 SFC。

```js
// Dirty
class TableRowWrapper extends Component {
  render() {
    return (
      <tr>
        {this.props.children}
      </tr>
    );
  }
}
```

整洁版本清除了很多可能导致干扰的信息。通过 React 核心的优化，使用无状态组件将占用更少的内存，因为没有创建 Component 实例。

```js
// Clean
const TableRowWrapper = ({ children }) => (
  <tr>
    {children}
  </tr>
);

```

### 剩余/扩展属性（rest/spread）

大约在一年前，我还推荐大家多用 `Object.assign`。但时代变化很快，在 ES2016/ES7 中引入新特性 [rest/spread](https://github.com/tc39/proposal-object-rest-spread)。

比如这样一种场景，当传递给一些 props 给一个组件，只希望在组件本身使用 `className`，但是需要将其他所有 props 传递到子组件。这时，你可能会这样做：

```js
// Dirty
const MyComponent = (props) => {
  const others = Object.assign({}, props);
  delete others.className;

  return (
    <div className={props.className}>
      {React.createElement(MyOtherComponent, others)}
    </div>
  );
};

```

这不是一个非常优雅的解决方案。但是使用 rest/spread，就能轻而易举地实现，

```js
// Clean
const MyComponent = ({ className, ...others }) => (
  <div className={className}>
    <MyOtherComponent {...others} />
  </div>
);
```

我们将剩余属性展开并作为新的 props 传递给 `MyOtherComponent` 组件。

### 合理使用解构

ES6 引入 [解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)（destructuring） 的概念，这是一个非常棒的特性，用类似对象或数组字面量的语法获取一个对象的属性或一个数组的元素。

#### 对象解构

在这个例子中，`componentWillReceiveProps` 组件接收 `newProps` 参数，然后将其 `active` 属性设置为新的 `state.active`。

```js
// Dirty
componentWillReceiveProps(newProps) {
  this.setState({
    active: newProps.active
  });
}
```

在整洁版本中，我们解构 `newProps `成 `active`。这样我们不仅不需要引用 `newProps.active`，而且也可以使用 ES6 的简短属性特性来调用 `setState`。

```
// Clean
componentWillReceiveProps({ active }) {
  this.setState({ active });
}

```

#### 数组解构

一个经常被忽视的 ES6 特性是数组解构。以下面的代码为例，它获取 `locale` 的值，比如“en-US”，并将其分成 `language`（en）和 `country`（US）。

```js
// Dirty
const splitLocale = locale.split('-');
const language = splitLocale[0];
const country = splitLocale[1];

```

在整洁版本，使用 ES6 的数组解构特性可以自动完成上述过程：

```js
// Clean
const [language, country] = locale.split('-');

```

## 所以结论是

希望这篇文章能有助于你看到编写整洁代码的好处，甚至可以直接使用这里介绍的一些代码示例。一旦你习惯编写整洁代码，将很快就会体会到 “write it and forget it” 的生活方式。
