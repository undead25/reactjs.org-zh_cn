---
id: portals
title: Portals
permalink: docs/portals.html
---

Portal 提供了一个不错的方式来将子节点渲染到父组件 DOM 以外的节点中，

```js
ReactDOM.createPortal(child, container)
```

第一个参数（`child`）可以是任何[可被渲染的 React 子节点](/docs/react-component.html#render)，例如元素、字符串或者片段。第二个参数（`container`）是一个 DOM 元素。

## 用法

通常情况下，当从组件的 render 方法返回一个元素时，它将作为最近父节点的子节点来挂载到 DOM 中。

```js{4,6}
render() {
  // React 挂载一个新的 div 并渲染里面的子节点
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```

然而，有些时候在 DOM 的其他位置插入子节点是很有用的。

```js{6}
render() {
  // React **不会**创建一个新的 div，而是渲染子节点到 `domNode`。
  // `domNode` 可以是任何有效的 DOM 节点，不管它在 DOM 中的位置
  return ReactDOM.createPortal(
    this.props.children,
    domNode,
  );
}
```

Portal 一个典型的用例是，当父组件带有 `overflow: hidden` 或者 `z-index` 的样式时，而你需要子节点在显示上“打破”容器。例如，对话框、悬浮卡和工具提示。

> 注意：
>
> 当使用 portal，你需要确保遵循适当的可访问性准则，记住这点很重要。

[在 CodePen 上试一试](https://codepen.io/gaearon/pen/yzMaBd)。

## 通过 Portal 进行事件冒泡

即使 portal 可以在 DOM 树中的任何位置，但在其他方面的行为和普通的 React 子节点一致。像 context 的这样的功能完全一样，不管子节点是不是 portal，因为 portal 仍然存在于 **React 树** 中，不用考虑其在 **DOM 树**中的位置。

这包含事件冒泡。从 portal 里面触发的事件会冒泡到 **React 树**中的祖先，即使这些元素不是 **DOM 树**中的祖先。假设你有如下的 HTML 结构：

```html
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

在 `#app-root` 的 `Parent` 组件会能够捕获到来自于其兄弟节点 `#modal-root` 冒泡上来的事件。

```js{28-31,42-49,53,61-63,70-71,74}
// 这两个容器在 DOM 中是同级的
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // portal 元素在 Modal 的子节点挂载后会被插入到 DOM 树中，
    // 这意味着子节点会被挂载到分离的 DOM 节点上。如果子组件
    // 需要在挂载后立即附加到 DOM 树中，例如计算 DOM 节点的数量，
    // 或者在后代中使用 `autoFocus`，给 Modal 添加 state，并且
    // 在 Modal 插入到 DOM 树后才渲染其子节点。
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el,
    );
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {clicks: 0};
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // 这将在点击 Child 中的按钮后触发，更新 Parent 的 state，
    // 即使按钮不是 DOM 中的直接后代
    this.setState(prevState => ({
      clicks: prevState.clicks + 1
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p>
        <p>
          Open up the browser DevTools
          to observe that the button
          is not a child of the div
          with the onClick handler.
        </p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}

function Child() {
  // 这个按钮上的点击事件会冒泡到父级，因为没有定义 `onClick` 属性
  return (
    <div className="modal">
      <button>Click</button>
    </div>
  );
}

ReactDOM.render(<Parent />, appRoot);
```

[在 CodePen 上试一试](https://codepen.io/gaearon/pen/jGBWpE)

在父组件里捕获来自于 portal 的事件冒泡让开发更加灵活，而不是完全依赖于 portal。例如，如果你想渲染一个 `<Modal />` 组件，那父组件可以捕获它的事件，无论是否采用 portal 里的实现。
