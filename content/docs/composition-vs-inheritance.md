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

## 包含

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

## 特例

有些时候我们会将组件视为其它组件的“特例”。例如，我们可以说 `WelcomeDialog` 是 `Dialog` 的特例。

在 React 中，这也可以通过组合来实现，一个更“特定”的组件渲染一个更“通用”的组件，并使用 props 进行配置：

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

组合同样使用于类组件：

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

在 Facebook 我们在数千个组件中使用 React，并且我们还没有发现任何建议创建组件继承层次结构的用例。

Props 和组合提供了所有我们需要的灵活性，通过明确和安全的方式自定义组件的外观和行为。请记住，组件可以接受任意 props，包括原始值、React 元素或者函数。

如果你想在组件之间重用非 UI 功能，我们建议将其分离到单独的 JavaScript 模块中。组件可以将其导入并使用该函数、对象或类，而不对其进行扩展。
