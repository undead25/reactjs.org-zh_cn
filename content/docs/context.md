---
id: context
title: Context
permalink: docs/context.html
---

> 注意：
>
> `React.PropTypes` 在 React v15.5 之后已经被移到了其它的包中，请使用 [`prop-types`](https://www.npmjs.com/package/prop-types) 来定义 `contextTypes`.
>
> 我们提供了 [codemod 脚本](/blog/2017/04/07/react-v15.5.0.html#migrating-from-react.proptypes) 来进行自动转换。

通过 React 组件可以很容易地追踪数据流。当你仔细观察一个组件时，你可以看到哪些 props 被传递了，这使得你的应用很容易理解。

在某些情况下，你想通过组件树来传递数据，而不想在每一层上手动传递 props。
你可以直接使用 React 强大的 “context” API 来做到。

## 为什么不要使用 Context

绝大多数的应用都不需要使用 context。

如果你想要你的应用稳定，请不要使用 context。它是一个实验性的 API，在未来的 React 版本中可能会被更改。

如果你不熟悉类似 [Redux](https://github.com/reactjs/redux) 或者 [MobX](https://github.com/mobxjs/mobx) 的 state 管理库，请不要使用 context。对于许多实际应用，这些库和它们的 React 绑定是管理许多组件相关 state 的不错选择。Redux 可能是你需要的最佳解决方案，而不是 context。

如果你不是一个很有经验的 React 开发者，请不要使用 context。通常只使用 props 和 state 来实现功能是一个更好的方式。

如果你还是坚持要使用 context，那么请尽量在一小部分地方使用它，并且尽量避免直接使用 context API 以便将来 API 变化时可以很容易地进行升级。

## 如何使用 Context

假设你有一个这样的结构：

```javascript
class Button extends React.Component {
  render() {
    return (
      <button style={{background: this.props.color}}>
        {this.props.children}
      </button>
    );
  }
}

class Message extends React.Component {
  render() {
    return (
      <div>
        {this.props.text} <Button color={this.props.color}>Delete</Button>
      </div>
    );
  }
}

class MessageList extends React.Component {
  render() {
    const color = "purple";
    const children = this.props.messages.map((message) =>
      <Message text={message.text} color={color} />
    );
    return <div>{children}</div>;
  }
}
```

在这个示例中，我们手动传递了一个 `color` prop 来适当地设置 `Button` 和 `Message` 组件的样式。我们可以通过使用 context 在组件树中自动传递它。

```javascript{6,13-15,21,28-30,40-42}
import PropTypes from 'prop-types';

class Button extends React.Component {
  render() {
    return (
      <button style={{background: this.context.color}}>
        {this.props.children}
      </button>
    );
  }
}

Button.contextTypes = {
  color: PropTypes.string
};

class Message extends React.Component {
  render() {
    return (
      <div>
        {this.props.text} <Button>Delete</Button>
      </div>
    );
  }
}

class MessageList extends React.Component {
  getChildContext() {
    return {color: "purple"};
  }

  render() {
    const children = this.props.messages.map((message) =>
      <Message text={message.text} />
    );
    return <div>{children}</div>;
  }
}

MessageList.childContextTypes = {
  color: PropTypes.string
};
```

通过向 `MessageList`（context 的提供者）添加 `childContextTypes` 和 `getChildContext`，React 向下自动传递信息，其子树中的任何组件（本示例中是 `Button` 组件）都可以通过定义 `contextTypes` 来访问到这些信息。


如果 `contextTypes` 没有定义，那么 `context` 会是一个空对象。

## 父子组件解耦

Context 也可以创建父子组件通信的 API。例如 [React Router V4](https://reacttraining.com/react-router) 这个库就是这样实现的。

```javascript
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

const BasicExample = () => (
  <Router>
    <div>
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/about">About</Link></li>
        <li><Link to="/topics">Topics</Link></li>
      </ul>

      <hr />

      <Route exact path="/" component={Home} />
      <Route path="/about" component={About} />
      <Route path="/topics" component={Topics} />
    </div>
  </Router>
);
```

通过 `Router` 组件向下传递一些信息，每个 `Link` 和 `Route` 组件都可以与包含的 `Router` 进行通信。

在通过类似这样的 API 创建你的组件之前，请考虑下是否有更好的替代方案。例如，如果你喜欢，你可以将整个 React 组件作为 props 进行传递。

## 在生命周期方法中引用 Context

If `contextTypes` is defined within a component, the following [lifecycle methods](/docs/react-component.html#the-component-lifecycle) will receive an additional parameter, the `context` object:

如果在组件中定义 `contextTypes`，那么下面的 [生命周期方法](/docs/react-component.html#the-component-lifecycle)将接收一个额外的 `context` 对象参数。

- [`constructor(props, context)`](/docs/react-component.html#constructor)
- [`componentWillReceiveProps(nextProps, nextContext)`](/docs/react-component.html#componentwillreceiveprops)
- [`shouldComponentUpdate(nextProps, nextState, nextContext)`](/docs/react-component.html#shouldcomponentupdate)
- [`componentWillUpdate(nextProps, nextState, nextContext)`](/docs/react-component.html#componentwillupdate)

> 注意：
>
> 在 React 16 中，`componentDidUpdate` 不再接收 `prevContext` 参数。

## 在无状态函数组件中引用 Context

如果 `contextTypes` 作为属性被定义了，无状态函数组件也可以引用 `context`。下面的代码展示了用无状态函数组件写法来写 `Button` 组件：

```javascript
import PropTypes from 'prop-types';

const Button = ({children}, context) =>
  <button style={{background: context.color}}>
    {children}
  </button>;

Button.contextTypes = {color: PropTypes.string};
```

## 更新 Context

请不要这么干。

React 有一个更新 context 的 API，但基本已经被废弃了，你不应该使用它。

当 state 或者 props 改变时，`getChildContext` 函数将被调用。为了更新 context 中的数据，通过 `this.setState` 来触发当前组件 state 更新。这会触发一个新的 context，并且子组件会接收到这个更新。

```javascript
import PropTypes from 'prop-types';

class MediaQuery extends React.Component {
  constructor(props) {
    super(props);
    this.state = {type:'desktop'};
  }

  getChildContext() {
    return {type: this.state.type};
  }

  componentDidMount() {
    const checkMediaQuery = () => {
      const type = window.matchMedia("(min-width: 1025px)").matches ? 'desktop' : 'mobile';
      if (type !== this.state.type) {
        this.setState({type});
      }
    };

    window.addEventListener('resize', checkMediaQuery);
    checkMediaQuery();
  }

  render() {
    return this.props.children;
  }
}

MediaQuery.childContextTypes = {
  type: PropTypes.string
};
```

这里的问题是，如果其中有一个父组件的 `shouldComponentUpdate` 返回了 `false`，那么组件更新提供的 context 值不会在这个父组件的子组件中更新。那么使用 context 的组件就完全失控了，所以更新 context 基本上没有一个可靠的方式。[这篇文章](https://medium.com/@mweststrate/how-to-safely-use-react-context-b7e343eff076) 很好地解释了为什么这会是一个问题以及如何避免。
