---
id: refs-and-the-dom
title: Refs 和 DOM
redirect_from:
  - "docs/working-with-the-browser.html"
  - "docs/more-about-refs.html"
  - "docs/more-about-refs-ko-KR.html"
  - "docs/more-about-refs-zh-CN.html"
  - "tips/expose-component-functions.html"
  - "tips/children-undefined.html"
permalink: docs/refs-and-the-dom.html
---

在典型的 React 数据流中，[props](/docs/components-and-props.html) 是父组件与子组件通信的唯一方式。要修改子组件，你需要通过新的 props 来重新渲染它。然而，某些情况下，你需要在典型的数据流之外强制修改子组件。需要修改的子组件可以是一个 React 组件实例，也可以是一个 DOM 元素。对于这两种情况，React 提供了一个解决方案。

### 何时使用 Refs

以下是几个使用 refs 很好的例子：

* 管理焦点、文本选择或者媒体播放。
* 触发命令式动画。
* 与第三方 DOM 库集成。

避免在可以使用声明式实现的情况下使用 refs。

例如，不要在 `Dialog` 组件暴露 `open()` 和 `close()` 方法，通过 `isOpen` prop 传递。

### 请勿过度使用 Refs

你在应用中使用 refs 的目的是为了“让事情发生”。如果是这种情况，请花点时间仔细考虑一下在组件层次结构中的哪个位置放置 state。通常情况下，显然是在层次结构中较高的位置。相关示例请参考 [State 提升](/docs/lifting-state-up.html) 指南。

### 给 DOM 元素添加 Ref

React 支持你给任何组件添加一个特殊的属性。`ref` 属性接受一个回调函数参数，这个回调会在组件挂载或者卸载时立即被执行。

当 `ref` 属性在 HTML 元素上使用时，`ref` 回调会接收底层的 DOM 元素作为参数。例如，下面的代码使用 `ref` 回调来存储 DOM 节点的引用：

```javascript{8,9,18}
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // 使用原生 DOM API 来聚焦稳步输入
    this.textInput.focus();
  }

  render() {
    // 使用 `ref` 回调在实例字段中存储对文本输入 DOM 元素的引用（例如，this.textInput）。
    return (
      <div>
        <input
          type="text"
          ref={(input) => { this.textInput = input; }} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

React 会在组件挂载时调用带有 DOM 元素的 `ref` 回调，并在挂载时调用带有 `null` 的回调。`ref` 回调会在 `componentDidMount` 或者 `componentDidUpdate` 生命周期钩子之前执行。

使用 `ref` 回调来设置类的属性是访问 DOM 元素的常用模式。首选的方法是像上面的例子那样在 `ref` 回调中设置属性。甚至有更短的写法：`ref={input => this.textInput = input}`。

### 给类组件添加 Ref

当 `ref` 属性被用于声明为类的自定义组件时，`ref` 回调接收挂载组件的实例作为参数。例如，如果我们想包裹上面的 `CustomTextInput` 组件，来实现挂载后立即点击的效果：


```javascript{3,9}
class AutoFocusTextInput extends React.Component {
  componentDidMount() {
    this.textInput.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

需要注意的是，这只适用于以类声明的 `CustomTextInput`：

```js{1}
class CustomTextInput extends React.Component {
  // ...
}
```

### Refs 与函数式组件

**你不能在函数式组件上使用 `ref` 属性**，因为它们没有实例：

```javascript{1,7}
function MyFunctionalComponent() {
  return <input />;
}

class Parent extends React.Component {
  render() {
    // 这**不会**工作！
    return (
      <MyFunctionalComponent
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

如果你需要使用 ref，你需要将组件转换成类，就像你需要生命周期方法或者 state 时一样。

但是，你可以**在函数式组件内部使用 `ref` 属性**，只要其指向的是一个 DOM 元素或者类组件：

```javascript{2,3,6,13}
function CustomTextInput(props) {
  // textInput 必须在这里声明，这样 ref 回调才可以引用它
  let textInput = null;

  function handleClick() {
    textInput.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={(input) => { textInput = input; }} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );  
}
```

### 暴露 DOM Refs 给父组件

在极少数情况下，你可能想从父组件中访问子组件的 DOM 节点。通常不建议你这样做，因为这会破坏组件的封装，但偶尔有助于触发聚焦或者计算子 DOM 节点的大小或位置。

虽然你可以 [添加一个 ref 到子组件](#adding-a-ref-to-a-class-component)，但这不是一个理想的解决方案，因为你只能获取组件实例，而不是 DOM 节点。另外，这不适用于函数式组件。

在这种情况下，我们推荐在子组件上暴露一个特殊的 prop。子组件接收一个任意名称（例如 `inputRef`）的函数 prop，并将其作为 `ref` 属性附加到 DOM 节点。这让父组件通过中间组件将其 ref 回调传递个子组件的 DOM 节点。

这适用于类组件和函数式组件。

```javascript{4,13}
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```

在上面的例子中，`Parent` 将其 ref 回调作为一个 `inputRef` prop 传递给了 `CustomTextInput`，并且 `CustomTextInput` 将同样的函数作为一个特殊的 `ref` 传递了 `input`。结果是 `Parent` 中的 `this.inputElement` 会被设置为 `CustomTextInput` 中 `<input>` 元素对应的 DOM 节点。

需要注意的是，在上面的例子中，`inputRef` 这个名称没有特殊的含义，它只是一个普通的组件 prop。然而，在 `<input>` 上使用 `ref` 属性很重要，因为这会告诉 React 将 ref 附加到它的 DOM 节点。

即使 `CustomTextInput` 是一个函数式组件也有效。这与[只能为 DOM 元素和类组件指定的](#refs-and-functional-components) `ref` 属性不同，对于像 `inputRef` 这样的普通组件 prop 是没有限制的。

这种模式的另外一个好处是它作用于多层级组件。例如，想象一下 `Parent` 不需要那个 DOM 节点，但是渲染 `Parent`（让我们叫它 `Grandparent`）的组件需要访问那个 DOM 节点。然后我们可以在 `Grandparent` 里指定 `Parent` 的 `inputRef` prop，并让 `Parent` 将其转发给 `CustomTextInput`：

```javascript{4,12,22}
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

function Parent(props) {
  return (
    <div>
      My input: <CustomTextInput inputRef={props.inputRef} />
    </div>
  );
}


class Grandparent extends React.Component {
  render() {
    return (
      <Parent
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```

在这里，`Grandparent` 首先指定了 ref 回调。它作为一个叫做 `inputRef` 的普通 prop 被传递给了 `Parent`，`Parent` 也将其作为一个 prop 传递给了 `CustomTextInput`。最终，`CustomTextInput` 读取了 `inputRef` prop，并将传递的函数作为一个 `ref` 属性传递给了 `<input>`。结果是，在 `Grandparent` 中的 `this.inputElement` 将被设置为 `CustomTextInput` 中 `<input>` 元素对应的 DOM 节点。

总而言之，我们建议尽可能地不要暴露 DOM 节点，但这可能是一个可用的方法。需要注意的是，这种方法需要你在子组件中添加一些代码。如果你无法完全控制子组件的实现，你的最终选择则是使用 [`findDOMNode()`](/docs/react-dom.html#finddomnode)，但不推荐。

### 旧版 API：字符串 Refs

如果你之前使用过 React，那么你可能知道在老的 API 中 `ref` 属性是一个类似 `"textInput"` 的字符串，并且可以通过 `this.refs.textInput` 来访问 DOM 节点。我们不建议这样使用，因为字符串 ref [存在问题](https://github.com/facebook/react/pull/8333#issuecomment-271648615)，它已经过时并**可能会在未来的版本被移除**。如果你目前正在使用 `this.refs.textInput` 来访问 ref，我们推荐使用回调的方式来代替。

### 注意事项

如果 `ref` 回调被定义为一个内联函数，那在更新期间它会被调用两次，首先是 `null`，然后是 DOM 元素。这是因为每次渲染都会创建一个新的函数实例，所以 React 需要清除旧的引用并设置新的实例。你可以通过将 `ref` 回调定义为类的绑定方法来避免这种情况，但在大多数情况下，这应该不重要。