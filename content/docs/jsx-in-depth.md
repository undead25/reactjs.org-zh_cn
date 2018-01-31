---
id: jsx-in-depth
title: 深入 JSX
permalink: docs/jsx-in-depth.html
redirect_from:
  - "docs/jsx-spread.html"
  - "docs/jsx-gotchas.html"
  - "tips/if-else-in-JSX.html"
  - "tips/self-closing-tag.html"
  - "tips/maximum-number-of-jsx-root-nodes.html"
  - "tips/children-props-type.html"
  - "docs/jsx-in-depth-zh-CN.html"
  - "docs/jsx-in-depth-ko-KR.html"
---

基本上，JSX 只是给 `React.createElement(component, props, ...children)` 函数提供语法糖。JSX 代码：

```js
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

编译为：

```js
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

如果没有子节点，你可以使用自闭合标签：

```js
<div className="sidebar" />
```

编译为：

```js
React.createElement(
  'div',
  {className: 'sidebar'},
  null
)
```

如果你想测试某些指定的 JSX 是如何转换成 JavaScript 的，可以尝试下 [Babel 在线编译器](babel://jsx-simple-example)

## 指定 React 元素类型

JSX 标签的第一个部分决定了 React 元素的类型。

首字母大写表示 JSX 标签代表一个 React 组件。这些标签会被便以为指定变量的直接引用，所以，如果你使用 JSX `<Foo />` 表达式，那 `Foo` 必须在作用域内。

### React 必须在作用域内

由于 JSX 编译为 `React.createElement` 的调用，所以 `React` 库也必须始终在 JSX 代码的作用域中。

例如，在下面的代码中，尽管 `React` 和 `CustomButton` 都没有直接使用 JavaScript 引用，但这两个导入都是必须的：

```js{1,2,5}
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // 返回 React.createElement(CustomButton, {color: 'red'}, null);
  return <CustomButton color="red" />;
}
```

如果你没有使用 JavaScript 打包器，而是通过 `<script>` 标签来加载的，那 `React` 已经作用于全局了。

### 使用点号

你也可以在 JSX 中使用点号来引用 React 组件。这可以方便你从一个模块中返回多个 React 组件。例如，如果 `MyComponents.DatePicker` 是一个组件，那你可以在 JSX 中直接使用它：

```js{10}
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

### 用户定义的组件必须被大写

当元素类型以小写字母开头时，它指的是一个像 `<div>` 或者 `<span>` 这样的内置组件，并会生成一个字符串 `'div'` 或者 `'span'` 传递给 `React.createElement`。像 `<Foo />` 这样以大写字母开头的类型会被编译为 `React.createElement(Foo)`，并对应你在 JavaScript 文件中定义或导入的组件。

我们建议用大写字母开头来命名组件。如果你确实有一个以小写字母开头的组件，那么在 JSX 中使用它之前，将其赋值给一个大写的变量。

例如，下面的代码将不会按预期运行：

```js{3,4,10,11}
import React from 'react';

// 错误！这是一个组件，首字母必须大写：
function hello(props) {
  // 正确！这种使用 <div> 是合法的，因为 div 是一个有效的 HTML 标签：
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // 错误！React 认为 <hello /> 是一个 HTML 标签，因为它不是首字母大写的：
  return <hello toWhat="World" />;
}
```

为了解决这个问题，我们会将 `hello` 重命名为 `Hello`，并在引用它时使用 `<Hello />`：

```js{3,4,10,11}
import React from 'react';

// 正确！组件名应该首字母大写:
function Hello(props) {
  // 正确！这种使用 <div> 是合法的，因为 div 是一个有效的 HTML 标签：
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // 正确！React 知道 <Hello /> 是一个 组件，因为它是首字母大写的：
  return <Hello toWhat="World" />;
}
```

### 在运行时选择类型

你不能使用通用表达式来作为 React 元素类型。如果你确实想使用通用表达式来指明元素的类型，只需首先将它赋值给首字母大写的变量。这在你想根据 prop 渲染不同的组件时很常见。

```js{10,11}
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 错误！元素类型不能是一个表达式。
  return <components[props.storyType] story={props.story} />;
}
```

要解决这个问题，我们首先需要将其赋值给首字母大写的变量：

```js{10-12}
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 正确！元素类型可以是一个首字母大写的变量。
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

## JSX 中的 Props

在 JSX 中有几种不同的方式来指定 prop。

### JavaScript Expressions as Props

你可以传递通过 `{}` 包裹的 JavaScript 表达式来作为 prop。例如下面的 JSX：

```js
<MyComponent foo={1 + 2 + 3 + 4} />
```

对于 `MyComponent`，由于表达式 `1 + 2 + 3 + 4` 的求值，所以 `props.foo` 的值会是 `10`。

`if` 语句和 `for` 循环不是 JavaScript 表达式，所以它们不能在 JSX 中直接使用。你可以把它们放在周围的代码中。例如：

```js{3-7}
function NumberDescriber(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
```

你可在相应的章节中了解到关于[条件渲染](/docs/conditional-rendering.html)和[循环](/docs/lists-and-keys.html)的更多信息。

### 字符串字面量

你可以传递字符串字面量来作为属性。下面的两个 JSX 表达式是相同的：

```js
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

当你传递字符串字面量时，它不会被 HTML 转义。所以下面两个 JSX 表达式是相同的：

```js
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

这种行为通常都不相关。这里提到是为了完整性。

### Props 默认为 `true`

如果 prop 没有传值，那它的值默认是 `true`。下面两个 JSX 表达式是相同的：

```js
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

一般情况下，我们不推荐这么使用，因为这会和 [ES6 的对象简写](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015)混淆，比如 `{foo}` 是 `{foo: foo}` 的简写，而不是 `{foo: true}`。这种行为只是为匹配 HTML 的行为。

### 属性拓展

如果你已经有了一个 `props` 对象，并且想在 JSX 中传递它，你可以使用 `...` 作为“拓展”操作符来传递整个 props 对象。下面这两个组件是相同的：

```js{7}
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

你也可以选择组件会消费的确切 props，同时使用拓展操作符来传递所有其他的 props。

```js{2}
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
};
```

在上面的例子中，`kind` prop 被安全的消费了，并且**不会**传递给 DOM 中的 `<button>` 元素。
所有其他的 props 都通过 `...other` 对象传递，这使得这个组件非常灵活。你可以看到它传递了 `onClick` 和 `children` props。

属性拓展很有用，但它们也会很容易地向组件传递不必要的 props，或者向 DOM 传递无效的 HTML 属性。我们建议少用这个语法。

## JSX 子节点

在同时包含了开始标签和结束标签的 JSX 表达式中，这些标签中间的内容被作为特殊的 prop 传递：`props.children`。有几种不同的方式来传递子节点：

### 字符串字面量

你可以在开始和结束标签中放置字符串，`props.children` 就是那个字符串。这对很多内置的 HTML 元素很有用。例如：

```js
<MyComponent>Hello world!</MyComponent>
```

这是一个有效的 JSX，在 `MyComponent` 中的 `props.children` 是字符串 `"Hello world!"`。HTML 没有转义，所以你可以像写 HTML 一样来写 JSX。

```html
<div>This is valid HTML &amp; JSX at the same time.</div>
```

JSX removes whitespace at the beginning and ending of a line. It also removes blank lines. New lines adjacent to tags are removed; new lines that occur in the middle of string literals are condensed into a single space. So these all render to the same thing:

```js
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### JSX Children

You can provide more JSX elements as the children. This is useful for displaying nested components:

```js
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

You can mix together different types of children, so you can use string literals together with JSX children. This is another way in which JSX is like HTML, so that this is both valid JSX and valid HTML:

```html
<div>
  Here is a list:
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
```

A React component can also return an array of elements:

```js
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

### JavaScript Expressions as Children

You can pass any JavaScript expression as children, by enclosing it within `{}`. For example, these expressions are equivalent:

```js
<MyComponent>foo</MyComponent>

<MyComponent>{'foo'}</MyComponent>
```

This is often useful for rendering a list of JSX expressions of arbitrary length. For example, this renders an HTML list:

```js{2,9}
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

JavaScript expressions can be mixed with other types of children. This is often useful in lieu of string templates:

```js{2}
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

### Functions as Children

Normally, JavaScript expressions inserted in JSX will evaluate to a string, a React element, or a list of those things. However, `props.children` works just like any other prop in that it can pass any sort of data, not just the sorts that React knows how to render. For example, if you have a custom component, you could have it take a callback as `props.children`:

```js{4,13}
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

Children passed to a custom component can be anything, as long as that component transforms them into something React can understand before rendering. This usage is not common, but it works if you want to stretch what JSX is capable of.

### Booleans, Null, and Undefined Are Ignored

`false`, `null`, `undefined`, and `true` are valid children. They simply don't render. These JSX expressions will all render to the same thing:

```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

This can be useful to conditionally render React elements. This JSX only renders a `<Header />` if `showHeader` is `true`:

```js{2}
<div>
  {showHeader && <Header />}
  <Content />
</div>
```

One caveat is that some ["falsy" values](https://developer.mozilla.org/en-US/docs/Glossary/Falsy), such as the `0` number, are still rendered by React. For example, this code will not behave as you might expect because `0` will be printed when `props.messages` is an empty array:

```js{2}
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>
```

To fix this, make sure that the expression before `&&` is always boolean:

```js{2}
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```

Conversely, if you want a value like `false`, `true`, `null`, or `undefined` to appear in the output, you have to [convert it to a string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#String_conversion) first:

```js{2}
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
```
