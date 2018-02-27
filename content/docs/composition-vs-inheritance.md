---
id: composition-vs-inheritance
title: 组合和继承
permalink: docs/composition-vs-inheritance.html
redirect_from: "docs/multiple-components.html"
prev: lifting-state-up.html
next: thinking-in-react.html
---

React 有一个强大的组合模型，我们推荐使用组合而不是继承来重用组件之间的代码。

在本章节中，我们将考虑 React 开发者新手经常遇到的一些问题，并展示如何使用组合来解决它们。

## Containment

一些组件不会预先知道它们的子节点是什么。尤其是像 `Sidebar` 或者 `Dialog` 这样代表“盒子”的通用组件。

我们推荐这样的组件使用特殊的 `children` prop 来直接传递子节点元素到它们的输出：

```js{4}
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

这让其他组件可以通过 JSX 嵌套传递任意子节点给它们：

```js{4-9}
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

[在 CodePen 上尝试](https://codepen.io/gaearon/pen/ozqNOV?editors=0010)

`<FancyBorder>` JSX 标签中的任何内容都会作为 `children` prop 传递到 `FancyBorder` 组件中。由于 `FancyBorder` 在一个 `<div>` 中渲染 `{props.children}`，那么传递的元素会在最终的输出中出现。

虽然这种情况不太常见，但有时你可在组件中需要多个“插入空间”。在这种情况下，你可以使用自定义的 prop 而不是使用 `children`：

```js{5,8,18,21}
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

[在 CodePen 上尝试](https://codepen.io/gaearon/pen/gwZOJp?editors=0010)

像 `<Contacts />` 和 `<Chat />` 这样的 React 元素只是一个对象，所以你可以像其他数据一样将它们作为 props 传递。这种方法可能会让你想起其他库中的 “slots”，但在 React 中，可以作为 props 传递的内容是没有限制的。

## Specialization

Sometimes we think about components as being "special cases" of other components. For example, we might say that a `WelcomeDialog` is a special case of `Dialog`.

In React, this is also achieved by composition, where a more "specific" component renders a more "generic" one and configures it with props:

```js{5,8,16-18}
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

[在 CodePen 上尝试](https://codepen.io/gaearon/pen/kkEaOZ?editors=0010)

Composition works equally well for components defined as classes:

```js{10,27-31}
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

[在 CodePen 上尝试](https://codepen.io/gaearon/pen/gwZbYa?editors=0010)

## 那什么是继承呢？

At Facebook, we use React in thousands of components, and we haven't found any use cases where we would recommend creating component inheritance hierarchies.

Props and composition give you all the flexibility you need to customize a component's look and behavior in an explicit and safe way. Remember that components may accept arbitrary props, including primitive values, React elements, or functions.

If you want to reuse non-UI functionality between components, we suggest extracting it into a separate JavaScript module. The components may import it and use that function, object, or a class, without extending it.
