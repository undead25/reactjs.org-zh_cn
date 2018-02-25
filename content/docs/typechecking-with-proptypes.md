---
id: typechecking-with-proptypes
title: PropTypes 类型检查
permalink: docs/typechecking-with-proptypes.html
redirect_from:
  - "docs/react-api.html#typechecking-with-proptypes"
---

> 注意：
>
> `React.PropTypes` 自 React v15.5 起被移到了一个不同的包。请使用 [`prop-types`](https://www.npmjs.com/package/prop-types) 库。
>
> 我们提供了 [codemod 脚本](/blog/2017/04/07/react-v15.5.0.html#migrating-from-reactproptypes) 来进行自动转换。

随着应用的日渐庞大，你可以通过类型检查来捕获大量错误。对于某些应用，你可以使用像 [Flow](https://flowtype.org/) 或者 [TypeScript](https://www.typescriptlang.org/) 一样的 JavaScript 拓展来对整个应用进行类型检查。但即使你不使用它们，React 也有一些内置的类型检查功能。要检测组件的属性，你可以给组件定义特殊的 `propTypes` 属性：

```javascript
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```

`PropTypes` 导出了一系列验证器用于确保接收数据的有效性。在上面的例子中，我们使用的是 `PropTypes.string`。当给 prop 提供一个无效值时，JavaScript 控制台会显示一条警告。出于性能考虑，`propTypes` 仅在开发环境下进行验证。

### PropTypes

下面是提供不同验证器的例子：

```javascript
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以指定特定 JS 类型的 prop。默认情况下，这些都是可选的
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可以渲染的内容：数字、字符串、元素或者包含这些类型的数组（或者片段）
  optionalNode: PropTypes.node,

  // React 元素
  optionalElement: PropTypes.element,

  // 你也可以指定 prop 是一个类的实例。使用 JS 的 instanceof 操作符
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以通过枚举来确保 prop 被限定为指定的值
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 对象可以是多种类型中的一个
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 特定类型的数组
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 具有特定类型属性值的对象
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 特定格式的对象
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // 如果没有提供 prop，你可以通过 `isRequired` 链接上面任何一种类型来确保显示警告
  requiredFunc: PropTypes.func.isRequired,

  // 任意数据类型的值
  requiredAny: PropTypes.any.isRequired,

  // 你也可以指定自定义的验证器。如果严重失败则应该返回一个 Error 对象。
  // 不要使用 `console.warn` 或者 throw，因为这会在 `oneOfType` 中不起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 你也可以为 `arrayOf` 和 `objectOf` 提供一个自定义的验证器。
  // 如果严重失败则应该返回一个 Error 对象。在数组或者对象中的每一个 key 都将会调用它。
  // 验证器的前第一个参数是数组或者对象自身，第二个参数是当前项的 key。
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

### 单个子组件

你可以通过 `PropTypes.element` 来指定只有一个子组件可以作为子节点传入到组件。

```javascript
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // 这必须是一个元素，否则它会发出警告。
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

### Prop 默认值

你可以通过定义特殊的 `defaultProps` 属性来定义 `props` 的默认值：

```javascript
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// 指定 props 的默认值：
Greeting.defaultProps = {
  name: 'Stranger'
};

// 渲染“Hello, Stranger”：
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);
```

If you are using a Babel transform like [transform-class-properties](https://babeljs.io/docs/plugins/transform-class-properties/) , you can also declare `defaultProps` as static property within a React component class. This syntax has not yet been finalized though and will require a compilation step to work within a browser. For more information, see the [class fields proposal](https://github.com/tc39/proposal-class-fields).

如果你正在使用像 [transform-class-properties](https://babeljs.io/docs/plugins/transform-class-properties/) 这样的 Babel 转换，你也可以在 React 组件类中声明 `defaultProps` 作为静态属性。更多信息请参考 [类字段提议](https://github.com/tc39/proposal-class-fields)。

```javascript
class Greeting extends React.Component {
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    return (
      <div>Hello, {this.props.name}</div>
    )
  }
}
```

The `defaultProps` will be used to ensure that `this.props.name` will have a value if it was not specified by the parent component. The `propTypes` typechecking happens after `defaultProps` are resolved, so typechecking will also apply to the `defaultProps`.

`defaultProps` 将用于在父组件没有指定的情况下，确保 `this.props.name` 有值。`propTypes` 类型检查发生在 `defaultProps` 解析后，所以类型检查也作用于 `defaultProps`。