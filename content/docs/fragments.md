---
id: fragments
title: 片段
permalink: docs/fragments.html
---

React 一个常见的模式是组件返回多个元素。片段可以让你集合一个子节点列表，而无需在 DOM 中添加额外的节点。

```js
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

也有一个新的[简写语法](#short-syntax)，但并不是所有的工具都支持。

## 动机

一个常见的模式是组件返回一个子节点列表。请看下面的 React 代码片段示例：

```jsx
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```

`<Columns />` 需要返回多个 `<td>` 元素来渲染有效的 HTML。如果在 `<Columns />` 的 `render()` 里使用父 div，那最终的 HTML 将是无效的。

```jsx
class Columns extends React.Component {
  render() {
    return (
      <div>
        <td>Hello</td>
        <td>World</td>
      </div>
    );
  }
}
```

`<Table />` 的输出结果：

```jsx
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

所以我们引入了 `Fragment`。

## 用法

```jsx{4,7}
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```

正确的 `<Table />` 输出结果：

```jsx
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```

### 简写

有一个新的简写来声明片段，有点像空标签：

```jsx{4,7}
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```

你可以像使用其他元素一样使用 `<></>`，除了不支持 key 或属性。

需要注意的是**[许多工具还不支持它](/blog/2017/11/28/react-v16.2.0-fragment-support.html#support-for-fragment-syntax)**，所以你可能需要明确地写 `<React.Fragment>` 直到工具支持。

### 带 key

`<React.Fragment>` 的写法来声明片段可以使用 key。一个使用场景是映射一个集合为片段数组 —— 例如，创建一个这样的描述列表：

```jsx
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // 如果不带 `key` 将会触发 key 的警告
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

`key` 是 `Fragment` 唯一可以传递的属性。将来可能会支持其他属性，例如事件处理。

### 在线示例

You can try out the new JSX fragment syntax with this [CodePen](https://codepen.io/reactjs/pen/VrEbjE?editors=1000).

你可以在 [CodePen](https://codepen.io/reactjs/pen/VrEbjE?editors=1000) 上尝试新的 JSX 片段语法。
