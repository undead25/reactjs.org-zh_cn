---
id: higher-order-components
title: 高阶组件
permalink: docs/higher-order-components.html
---

高阶组件（HOC）是 React 用来复用组件逻辑的高级技术，但它不是一个 React API，它是 React 组合性质必然产生的一种模式。

具体来说，**高阶组件是一个使用组件作为参数并返回一个新组件的函数**

```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

鉴于组件将 props 转换成 UI，高阶组件则是将一个组件转换成另一个组件。

HOC 在 React 第三方库中是很常见的，例如 Redux 的 [`connect`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) 和 Relay 的 [`createFragmentContainer`](http://facebook.github.io/relay/docs/en/fragment-container.html)。

本文将讨论为什么高阶组件很有用，以及如何编写一个高阶组件。

## Use HOCs For Cross-Cutting Concerns 使用高阶组件解决交叉问题

> **注意**
>
> 我们以前推荐使用混合（mixins）来解决交叉问题，然而我们发现使用混合会产生更多问题。阅读关于为什么我们移除混合，以及如何转换已有组件的[更多信息](/blog/2016/07/13/mixins-considered-harmful.html)

组件是 React 代码复用的主要单元，但你会发现有一些模式并不适合传统组件。

例如，假设你有一个 `CommentList` 组件来订阅外部数据源并渲染评论列表：

```js
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // “DataSource”是某个全局数据源
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // 订阅变化
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // 取消订阅
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // 数据源变化时更新组件状态
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

然后，你写了一个订阅单个博文的组件，它遵循类似的模式：

```js
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```

`CommentList` 和 `BlogPost` 并不相同，它们调用 `DataSource` 上不同的方法，渲染不同的输出数据。但它们的大部分实现逻辑是相同的：

- 组件挂载后，添加一个变化监听器到 `DataSource`。
- 在监听器内，当数据源变化时，调用 `setState`。
- 组件卸载后，移除变化监听器。

可以想象，在一个大型应用中，类似这样的订阅 `DataSource` 和调用 `setState` 的模式会不断发生。我们需要一个抽象，使我们能够在一个地方定义这个逻辑，并在多个组件之间共享，这就是高阶组件所擅长的。

我们可以写一个函数来创建像 `CommentList` 和 `BlogPost` 那样订阅 `DataSource`的组件。这个函数接受一个子组件作为其参数，并将获取的订阅数据作为属性。让我们称这个函数为 `withSubscription`。

```js
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```

第一个参数是被包裹的组件，第二个参数是一个函数，它将通过 `DataSource` 和当前的 props 来检索出我们感兴趣的数据。

当渲染 `CommentListWithSubscription` 和 `BlogPostWithSubscription` 时，会向 `CommentList` 和 `BlogPost` 传入一个 从 `DataSource` 检索出的最新数据的 `data` 属性。

```js
// 这个函数接受一个组件作为参数……
function withSubscription(WrappedComponent, selectData) {
  // ……并返回另一个组件……
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ……关心订阅……
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ……并用最新的数据渲染被包裹的组件！
      // 注意将任何附加的属性传递给被包裹的组件
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

需要注意的是，高阶组件既不会修改输入的组件，也不会使用继承来复制它的行为。相反，高阶组件通过将原始组件包裹在一个容器组件中来**组合**原始组件。高级组件是一个无副作用的纯函数。

就是这样，被包裹的组件接收容器组件所有的属性以及一个新的用来渲染其输出的 `data` 属性。高阶组件不关心数据如何或者为什么被使用，被包裹的组件也不关心数据是从哪里来的。

由于 `withSubscription` 是一个普通函数，你可以添加任意数量的参数。例如，你也许想让 `data` 属性可配置来进一步的分离高阶组件和被包裹的组件。或者你想接收一个用于配置 `shouldComponentUpdate` 的参数，或者一个配置数据源的参数。所有的这些都是可能的，因为高阶组件可以完全控制组件的定义。

像组件一样，`withSubscription` 和其包裹的组件之间的通信是完全基于属性的。这让你可以很轻松地将一个高阶组件换成另外一个，只要它们提供同样的属性给被包裹的组件。例如，如果你变更获取数据的库，这会很有用。

## 不要改变原始组件，使用组合

不要在高阶组件内部修改（或者改变）组件的原型。

```js
function logProps(InputComponent) {
  InputComponent.prototype.componentWillReceiveProps = function(nextProps) {
    console.log('Current props: ', this.props);
    console.log('Next props: ', nextProps);
  };
  // 实际上我们返回的原始组件已经发生了改变
  return InputComponent;
}

// EnhancedComponent 会打印所有收到的 props
const EnhancedComponent = logProps(InputComponent);
```

这会有一些问题。一个是 input 组件不能够在脱离 `EnhancedComponent` 的情况下复用。更重要的是，如果 `EnhancedComponent` 也应用了一个改变 `componentWillReceiveProps` 的高阶组件，`EnhancedComponent` 的功能就会被覆盖！这个高阶组件也不适用于没有生命周期方法的函数式组件。

改变高阶组件泄露了组件的抽象性 —— 使用者必须知道它们是怎么实现的来避免和其他高阶组件的冲突。

高阶组件应该使用组合而不是改变，通过将 input 组件包裹在一个容器组件中：

```js
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // 将 input 组件包裹在一个容器中，而不是改变它。
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

这个高阶组件和上面改变原型的版本有着同样的功能，但它避免了发生冲突的可能性。它适用于类组件和函数式组件。由于它是一个纯函数，所以它可以和其它或者自身进行组合。


你可能已经注意到高阶组件和**容器组件**的相似性。容器组件是高层和低层关注点之间，权责分离策略的一部分。容器组件管理类似订阅和状态的事情，并传递 props 给组件来渲染 UI。高阶组件使用容器作为其实现的一部分。你可以认为高阶组件是参数化的容器组件。

## 约定：向被包裹的组件传入无关的 Props

高阶组件给组件添加了新功能。它们不应该大幅度地修改组件的原有功能，其返回的组件期望有着和被包裹的组件相同的接口。

高阶组件应该传递与其具体实现无关的 props。大多数高阶组件都包含一个这样的 render 方法：

```js
render() {
  // 过滤掉这个高阶组件额外特有的，并且不应该传递的 props
  const { extraProp, ...passThroughProps } = this.props;

  // 向被包裹组件注入 props，通常是状态值或者实例方法。
  const injectedProp = someStateOrInstanceMethod;

  // 向被包裹的组件传递 props
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

这个约定帮助确保高阶组件尽可能的灵活和可复用。

## 约定：最大化组合性

并不是所有的高阶组件看起来都是一样的。某些时候它们只接收一个参数，即被包裹的组件：

```js
const NavbarWithRouter = withRouter(Navbar);
```

通常情况下，高阶组件会接收额外的参数。在下面这个来自 Relay 的例子中，配置对象用于指定组件的数据依赖关系：

```js
const CommentWithRelay = Relay.createContainer(Comment, config);
```

最常见的高阶组件看起来像这样：

```js
// React Redux 的 `connect`
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

**这是啥**？！如果你将它拆解就能够很容易地明白是怎么回事了。

```js
// connect 是一个函数，它返回另一个函数
const enhance = connect(commentListSelector, commentListActions);
// 返回的函数是一个高阶组件，这个高阶组件返回一个与 Redux store 关联的组件
const ConnectedComment = enhance(CommentList);
```
换句话说，`connect` 是一个返回高阶组件的高阶函数！

这种形式看起来可能很困惑或者没有必要，但是它有一个实用的属性，那就是类似 `connect` 函数返回的单参数高阶组件具有 `Component => Component` 的签名。输出类型和输入类型相同的函数可以很容易地组合在一起。

```js
// 不要这样写……
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ……你可以实用函数组合工具
// compose(f, g, h) 和 (...args) => f(g(h(...args))) 是一样的
const enhance = compose(
  // 它们都是单参数的高阶组件
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```

（同样的属性也允许 `connect` 和其它增强型高阶组件作为装饰器来使用，装饰器是一个实验性的 JavaScript 提议。）

包括 lodash（例如 [`lodash.flowRight`](https://lodash.com/docs/#flowRight)）、[Redux](http://redux.js.org/docs/api/compose.html) 和 [Ramda](http://ramdajs.com/docs/#compose) 在内的许多第三方库都提供了 `compose` 这个工具函数。

## 约定：包装显示名称用于调试

高阶组件创建的容器组件和其它组件在 [React Developer Tools](https://github.com/facebook/react-devtools) 中的显示是一样的。为了便于调试，选择一个显示名称来表述它是高阶组件返回的。

最常见的做法是包装被包裹组件的显示名称。因此，如果你的高阶组件被命名为 `withSubscription`，并且被包裹组件的显示名称是`CommentList`，则使用 `WithSubscription(CommentList)` 作为显示名称：

```js
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

## 注意事项
如果你是 React 新手，高阶组件的一些注意事项不会立即显现。

### 不要在 render 方法中使用高阶组件

React 的 diff 算法（叫做协调）使用组件标识来决定是否需要更新现有的子对象树或者丢掉它并挂载一个新的。如果 `render` 返回的组件和之前 render 返回的组件相等（`===`），React 通过比较新旧子对象树来进行递归更新。如果它们不相等，那么旧的子对象树会被完全卸载。

通常情况下你不需要关系这些，但它影响高阶组件，因为这意味着你不能在组件的 render 方法中应用高阶组件：

```js
render() {
  // 每一次调用 render 都会创建一个新的 EnhancedComponent
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 每一次调用 render 都会使整个子对象树卸载或者重载
  return <EnhancedComponent />;
}
```

The problem here isn't just about performance — remounting a component causes the state of that component and all of its children to be lost.


Instead, apply HOCs outside the component definition so that the resulting component is created only once. Then, its identity will be consistent across renders. This is usually what you want, anyway.

In those rare cases where you need to apply a HOC dynamically, you can also do it inside a component's lifecycle methods or its constructor.

### Static Methods Must Be Copied Over

Sometimes it's useful to define a static method on a React component. For example, Relay containers expose a static method `getFragment` to facilitate the composition of GraphQL fragments.

When you apply a HOC to a component, though, the original component is wrapped with a container component. That means the new component does not have any of the static methods of the original component.

```js
// Define a static method
WrappedComponent.staticMethod = function() {/*...*/}
// Now apply a HOC
const EnhancedComponent = enhance(WrappedComponent);

// The enhanced component has no static method
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

To solve this, you could copy the methods onto the container before returning it:

```js
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Must know exactly which method(s) to copy :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

However, this requires you to know exactly which methods need to be copied. You can use [hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics) to automatically copy all non-React static methods:

```js
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

Another possible solution is to export the static method separately from the component itself.

```js
// Instead of...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...export the method separately...
export { someFunction };

// ...and in the consuming module, import both
import MyComponent, { someFunction } from './MyComponent.js';
```

### Refs Aren't Passed Through

While the convention for higher-order components is to pass through all props to the wrapped component, it's not possible to pass through refs. That's because `ref` is not really a prop — like `key`, it's handled specially by React. If you add a ref to an element whose component is the result of a HOC, the ref refers to an instance of the outermost container component, not the wrapped component.

If you find yourself facing this problem, the ideal solution is to figure out how to avoid using `ref` at all. Occasionally, users who are new to the React paradigm rely on refs in situations where a prop would work better.

That said, there are times when refs are a necessary escape hatch — React wouldn't support them otherwise. Focusing an input field is an example where you may want imperative control of a component. In that case, one solution is to pass a ref callback as a normal prop, by giving it a different name:

```js
function Field({ inputRef, ...rest }) {
  return <input ref={inputRef} {...rest} />;
}

// Wrap Field in a higher-order component
const EnhancedField = enhance(Field);

// Inside a class component's render method...
<EnhancedField
  inputRef={(inputEl) => {
    // This callback gets passed through as a regular prop
    this.inputEl = inputEl
  }}
/>

// Now you can call imperative methods
this.inputEl.focus();
```

This is not a perfect solution by any means. We prefer that refs remain a library concern, rather than require you to manually handle them. We are exploring ways to solve this problem so that using a HOC is unobservable.
