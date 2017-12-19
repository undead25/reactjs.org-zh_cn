---
id: introducing-jsx
title: Introducing JSX
permalink: docs/introducing-jsx.html
prev: hello-world.html
next: rendering-elements.html
---

看下这个变量：

```js
const element = <h1>Hello, world!</h1>;
```

这个有趣的标签语法既不是一个字符串也不是 HTML。

它叫做 JSX，是 JavaScript 的语法拓展。我们建议 React 中使用它来描述用户界面。JSX 可能让你想起了模板余元，但它具有 JavaScript 的全部功能。

JSX 生成 React “元素”。我们将在[后面的章节](/docs/rendering-elements.html)中探索如何将它们渲染到 DOM 中。接下来你会了解到 JSX 的基本使用方法。

### 为什么是 JSX？

React 信奉渲染逻辑与其他 UI 逻辑耦合的事实：事件是如何处理，状态如何随时间改变以及数据如何准备显示。

React 通过松散耦合的单元 —— “组件”，来做到[关注点**分离**](https://en.wikipedia.org/wiki/Separation_of_concerns)，而不是使用人为的分离**技术** —— 将标记（HTML）和逻辑放在不同的文件中。我们将在[后面的章节](/docs/components-and-props.html)中讨论组件，但是如果你不习惯在 JS 中添加标记，[这个演讲](https://www.youtube.com/watch?v=x7cQ3mrcKaY)可能会说服你。

React 可以[不需要](/docs/react-without-jsx.html) 使用 JSX，但是大多数人发现在 JavaScript 代码中编写 UI 时，它可以作为视觉辅助工具。它还允许 React 显示更多有用的错误和警告消息。

就这样，让我们开始吧！

### 在 JSX 中使用表达式

你可以通过大括号在 JSX 中嵌入任何 [JavaScript 表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)

例如 `2 + 2`、`user.firstName` 和 `formatName(user)` 都是有效的表达式：

```js{12}
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

[](codepen://introducing-jsx).

为了便于阅读，我们将 JSX 分成了多行。虽然这不是必需的，但当这样做的时候，我们建议把它们放在圆括号中以避免[自动分号插入](http://stackoverflow.com/q/2846283)的缺陷。

### JSX 自身也是一个表达式

经过编译后，JSX 会被转换成普通的 JavaScript 对象。

这意味着你可以在 `if` 或者 `for` 循环中使用 JSX、将它赋值给变量、作为参数传入、以及从函数中返回。

```js{3,5}
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

### 在 JSX 中指定属性

你可以使用引号将字符串文字指定为属性：

```js
const element = <div tabIndex="0"></div>;
```

你也可以使用大括号将 JavaScript 表达式嵌入到属性中：

```js
const element = <img src={user.avatarUrl}></img>;
```

在属性中嵌入 JavaScript 表达式时，不要用引号包住大括号。对于同样的属性，应该使用引号（用于字符串值）或者大括号（用于表达式），但不能同时使用。

>**警告:**
>
> 由于 JSX 更接近 JavaScript 而不是 HTML，所以 React DOM 使用 `camelCase` 属性名称，而不是 HTML 属性名称。
>
> 例如在 JSX 中 `class` 变成了 [`className`](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)，`tabindex` 变成了 [`tabIndex`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/tabIndex)。

### Specifying Children with JSX

If a tag is empty, you may close it immediately with `/>`, like XML:

```js
const element = <img src={user.avatarUrl} />;
```

JSX tags may contain children:

```js
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### JSX Prevents Injection Attacks

It is safe to embed user input in JSX:

```js
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

By default, React DOM [escapes](http://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html) any values embedded in JSX before rendering them. Thus it ensures that you can never inject anything that's not explicitly written in your application. Everything is converted to a string before being rendered. This helps prevent [XSS (cross-site-scripting)](https://en.wikipedia.org/wiki/Cross-site_scripting) attacks.

### JSX Represents Objects

Babel compiles JSX down to `React.createElement()` calls.

These two examples are identical:

```js
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```js
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

`React.createElement()` performs a few checks to help you write bug-free code but essentially it creates an object like this:

```js
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world'
  }
};
```

These objects are called "React elements". You can think of them as descriptions of what you want to see on the screen. React reads these objects and uses them to construct the DOM and keep it up to date.

We will explore rendering React elements to the DOM in the next section.

>**建议：**
>
>We recommend using the ["Babel" language definition](http://babeljs.io/docs/editors) for your editor of choice so that both ES6 and JSX code is properly highlighted. This website uses the [Oceanic Next](https://labs.voronianski.com/oceanic-next-color-scheme/) color scheme which is compatible with it.
