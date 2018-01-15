---
id: error-boundaries
title: 错误边界
permalink: docs/error-boundaries.html
---

在过去，组件内部的 JavaScript 错误会破坏 React 的内部状态，并在下一次渲染时[发出](https://github.com/facebook/react/issues/4026)[隐晦的](https://github.com/facebook/react/issues/6895)[错误信息](https://github.com/facebook/react/issues/8579)。这些错误总是由于应用代码中早期的错误造成的，但 React 没有提供一种在组件内部优雅地处理它们的方式，也不能从错误中恢复。

## 错误边界介绍

部分 UI 的 JavaScript 错误不应该破坏整个应用。为了解决这个问题，React 16 引入一个叫做“错误边界”的新概念。

错误边界是用于一个 React 组件，它**捕获其子组件树中任何地方的 JavaScript 错误，记录这些错误并展示一个回退的 UI**，而不是组件树的崩溃。错误边界可以在渲染期间、生命周期方法内以及整棵树下的构造函数中捕获错误。

> 注意
> 
> 错误边界**不会**捕获如下错误：
>
> * 事件处理（[了解更多](#how-about-event-handlers)）
> * 异步代码（例如 `setTimeout` 或者 `requestAnimationFrame` 回调）
> * 服务端渲染
> * 错误边界自身抛出的异常（而不是其子组件）

如果一个类组件定义了一个新的 `componentDidCatch(error, info)` 生命周期方法，那么它就成了一个错误边界：

```js{7-12,15-18}
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

然后你可以把它作为一个普通组件来使用：

```js
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

`componentDidCatch()` 方法像 JavaScript 的 `catch {}` 一样，但它只针对组件。只有类组件可以成为错误边界。实际上，大多数时候你只需要声明一个错误边界组件，然后在整个应用中使用它。

需要注意的是**错误边界仅捕获其子组件树中的错误**。错误边界不能捕获其自身的错误。如果错误边界渲染错误信息失败，那错误会冒泡到其最近的错误边界。这也类似于 JavaScript 中 `catch {}`。

### componentDidCatch 参数

`error` 是被抛出的错误。

`info` 是一个包含 `componentStack` 键的对象。这个属性包含了抛出错误时关于组件的堆栈信息。

```js
//...
componentDidCatch(error, info) {
  
  /* 堆栈信息示例：
     in ComponentThatThrows (created by App)
     in ErrorBoundary (created by App)
     in div (created by App)
     in App
  */
  logComponentStackToMyService(info.componentStack);
}

//...
```

## 在线示例

查看这个使用  [React 16 beta](https://github.com/facebook/react/issues/10294) 来[声明和使用错误边界的示例](https://codepen.io/gaearon/pen/wqvxGa?editors=0010)。


## 何处放置错误边界

错误边界的粒度取决于你。你可以包裹顶层的路由组件来向用户展示一个“出错”信息，就像服务端框架通常处理崩溃一样。你也可以将独立插件包裹在错误边界中来保护应用不受其崩溃而影响。

## 未捕获错误的新行为

这一变化具有重要意义。**从 React 16 开始，任何没有被错误边界捕获的错误会导致整个 React 组件树的卸载。**

这个决定饱受争议，但根据我们的经验，保留错误的 UI 会比完全移除它更糟糕。例如，在像 Messenger 的产品中，保留错误的 UI 可见会导致有人向错误的人发送信息。同样地，在支付应用中展示一个错误的金额会比什么都不渲染更糟糕。

这个变化意味着，随着你迁移到 React 16，你可能会发现一些你以前在你的应用中没有注意到的崩溃。添加错误边界可以在发生错误时提供更好的用户体验。

例如，Facebook Messenger 将侧边栏、信息面板、聊天记录和信息输入框包裹在各自的错误边界中。如果这些 UI 中的某个组件崩溃了，其余部分仍然可用。

我们也鼓励你使用 JS 错误报告服务（或者自行构建）以便你可以可以发现生产环境产生的异常并修复它们。


## 组件栈追踪

在开发模式下，React 16 会在渲染过程中打印所有发生的错误到控制台，即使应用掩盖了它们。除了错误信息和 JavaScript 栈信息外，它还提供组件堆栈追踪。现在你可以看到错误发生在组件树中的确切位置：

<img src="../images/docs/error-boundaries-stack-trace.png" style="max-width:100%" alt="Error caught by Error Boundary component">

你也可以在组件的堆栈追踪中看到文件名和行数。这在 [Create React App](https://github.com/facebookincubator/create-react-app) 创建的项目中是默认开启的。

<img src="../images/docs/error-boundaries-stack-trace-line-numbers.png" style="max-width:100%" alt="Error caught by Error Boundary component with line numbers">

如果你没有使用 Create React App，你可以手动添加这个[插件](https://www.npmjs.com/package/babel-plugin-transform-react-jsx-source)到你的 Babel 配置中。需要注意的是这只适用于开发环境，**禁止在生产环境使用**。

> 注意
> 
> 在堆栈追踪中显示的组件名称依赖于 [`Function.name`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name) 属性。如果要支持的旧版本浏览器和设备没有原生提供此属性（例如 IE 11），请考虑添加 `Function.name` 的 polyfill 到你的应用中，例如 [`function.name-polyfill`](https://github.com/JamesMGreene/Function.name)。或者你可以在你所有的组件中显式设置 [`displayName`](/docs/react-component.html#displayname) 属性。


## 为什么不使用 try/catch？

`try` / `catch` 很好，但只适用于命令式代码：

```js
try {
  showButton();
} catch (error) {
  // ...
}
```

然而，React 组件是声明式的，需要指明**什么**需要被渲染：

```js
<Button />
```

错误边界保留了 React 声明式的特性，并按照你的预期工作。例如，在组件树中某处由 `setState` 引起的、发生在 `componentDidUpdate` 生命周期钩子中的错误，它仍然会正确地冒泡到最近的错误边界。

## 怎么处理事件？

错误边界**无法**捕捉到事件处理器内部的错误。

React 不需要错误边界从事件处理器的错误中来恢复。不像 render 方法和生命周期钩子，事件处理器不会在渲染过程中触发。所以如果它们抛出错误，React 仍然知道在屏幕上展示什么。

如果你需要在事件处理器中捕获错误，请使用普通的 `try` / `catch` 语句。

```js{8-12,16-19}
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
  }
  
  handleClick = () => {
    try {
      // 做一些可能抛出错误的操作
    } catch (error) {
      this.setState({ error });
    }
  }

  render() {
    if (this.state.error) {
      return <h1>Caught an error.</h1>
    }
    return <div onClick={this.handleClick}>Click Me</div>
  }
}
```

需要注意的是，上面的例子没有使用错误边界，只是演示普通的 JavaScript 行为。

## React 15 后名称的变更

React 15 在一个不同的方法名下 —— `unstable_handleError`，包含了对错误边界有限的支持。自 React 16 beta 发布，这个方法已不再工作，你需要在你的代码中将它改为 `componentDidCatch`。

对于这个改变，我们提供了一个 [codemod](https://github.com/reactjs/react-codemod#error-boundaries) 来自动迁移你的代码。
