---
id: thinking-in-react
title: React 编程思想
permalink: docs/thinking-in-react.html
redirect_from:
  - 'blog/2013/11/05/thinking-in-react.html'
  - 'docs/thinking-in-react-zh-CN.html'
prev: composition-vs-inheritance.html
---

在我们看来，React 是通过 JavaScript 构建大型高性能 Web 应用的首选。它在我们的 Facebook 和 Instagram 上表现得很好。

React 的重要组成部分之一就是让你思考如何构建应用。在本文中，我们将介绍使用 React 构建可搜索的产品数据表格的思考过程。

## 从原型开始

想象我们已有一个 JSON API 和一个设计师设计的原型。原型是这样的：

![Mockup](../images/blog/thinking-in-react-mock.png)

JSON API 返回的数据是这样的：

```
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## 步骤 1：将 UI 拆解为组件结构

你要做的第一件事就是在原型中的每个组件（和子组件）周围画框并命名。如果你正在与设计师一起工作，他们可能已经完成了这项工作，和他们谈谈！他们的 Photoshop 图层名称可能最终会成为你的 React 组件的名称！

但你怎么知道自己的组件应该是什么呢？只需使用决定是否应该创建新的函数或对象一样的技巧。其中一种技术是[单一责任原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)，也就是说，一个组件理想情况下只应该做一件事。如果它最终增长，那么它应该被分解成更小的子组件。

由于你经常向用户展示 JSON 数据模型，你会发现，如果你的模型构建正确，那么 UI（以及组件结构）将会很好地映射。这是因为 UI 和数据模型都倾向于遵循相同的**信息架构**，这意味着将 UI 分解为组件通常比较琐碎。只需将其分解为与某部分数据模型对应的组件。

![组件图](../images/blog/thinking-in-react-components.png)

在这里你会看到，在这个简单的应用中我们有五个组件。我们将每个组件所代表的数据用粗斜体表示。

  1. **`FilterableProductTable`（橙色）：** 包含整个示例
  2. **`SearchBar`（蓝色）：** 接收所有的**用户输入**
  3. **`ProductTable`（绿色）：** 根据**用户输入**展示和过滤**数据集**
  4. **`ProductCategoryRow`（宝石绿）：** 展示每个**类别**的标题
  5. **`ProductRow`（红色）：** 展示每条**产品**

如果你观察 `ProductTable`，你会看到表头（包含“名称”和“价格”标签）不是它自己的组件。这个是个人偏好的问题，有受争论的其他方法。对于这个例子，我们将它作为 `ProductTable` 的一部分，因为它是*产品集合*渲染的一部分，这是 `ProductTable` 的责任。但如果这个头部变得复杂（比如为分类添加排序功能），那独立一个 `ProductTableHeader` 组件会更好。

现在我们已经确定了原型中的组件，让我们将它们排列到一个层次结构中。这很容易。在原型中，出现在另一个组件内部的组件应该作为子组件出现在层次结构中：

  * `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
      * `ProductCategoryRow`
      * `ProductRow`

## Step 2: 在 React 中构建一个静态版本

<p data-height="600" data-theme-id="0" data-slug-hash="BwWzwm" data-default-tab="js" data-user="lacker" data-embed-version="2" class="codepen">See the Pen <a href="https://codepen.io/gaearon/pen/BwWzwm">Thinking In React: Step 2</a> on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

现在你已经拥有了组件的层次结构，是时候实施你的应用了。最简单的方法是构建一个接收数据模型并渲染 UI 但不具有交互的版本。最好解耦这些过程，因为构建一个静态版本需要大量的代码和少量思考，而添加交互需要大量的思考，而不是大量的代码。我们会看到这是为什么。

要构建渲染数据模型的静态版本应用，你需要构建可以重用其他组件，并使用 **props** 传递数据的组件。**props** 是一种将数据从父组件传递给子组件的方式。如果你熟悉 **state** 的概念，**请不要使用 state** 来构建这个静态版本。状态仅用于交互，即数据随时会改变。由于这是静态版本的应用，所以你不需要它。

您可以自上而下或者自下而上进行构建。也就是说，你可以从层次结构中较高的组件（即从 `FilterableProductTable` 开始）或者较低的组件（`ProductRow`）开始构建。在更简单的例子中，自上而下通常会更容易，而在大型项目中，自下而上和在构建时编写测试会更容易。

在这一步结束时，你会拥有一个渲染数据模型的可重用组件库。这些组件只有 `render()` 方法，因为这是一个静态版本。层次结构顶部的组件（`FilterableProductTable`）会把你的数据模型作为一个 prop。如果对底层数据模型进行更改，并再次调用 `ReactDOM.render()`，那UI 将会更新。这有利于观察 UI 是如何更新的以及在哪里进行更改，因为这里没有任何复杂的事情发生。React 的**单向数据流**（也称为**单向绑定**）使得所有内容模块化和高性能。

如果执行此步骤需要帮助，请参阅 [React文档](/docs/)。

### 小插曲：Props 和 State

在 React 中有两种类型的数据“模型”：props 和 state。理解它们之间的差别很重要，如果你不确定它们之间的区别，可以看下 [React 官方文档](/docs/interactivity-and-dynamic-uis.html)

## 步骤 3：确定最小（但完整）的 UI State

To make your UI interactive, you need to be able to trigger changes to your underlying data model. React makes this easy with **state**.

To build your app correctly, you first need to think of the minimal set of mutable state that your app needs. The key here is [DRY: *Don't Repeat Yourself*](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). Figure out the absolute minimal representation of the state your application needs and compute everything else you need on-demand. For example, if you're building a TODO list, just keep an array of the TODO items around; don't keep a separate state variable for the count. Instead, when you want to render the TODO count, simply take the length of the TODO items array.

Think of all of the pieces of data in our example application. We have:

  * The original list of products
  * The search text the user has entered
  * The value of the checkbox
  * The filtered list of products

Let's go through each one and figure out which one is state. Simply ask three questions about each piece of data:

  1. Is it passed in from a parent via props? If so, it probably isn't state.
  2. Does it remain unchanged over time? If so, it probably isn't state.
  3. Can you compute it based on any other state or props in your component? If so, it isn't state.

The original list of products is passed in as props, so that's not state. The search text and the checkbox seem to be state since they change over time and can't be computed from anything. And finally, the filtered list of products isn't state because it can be computed by combining the original list of products with the search text and value of the checkbox.

So finally, our state is:

  * The search text the user has entered
  * The value of the checkbox

## Step 4: Identify Where Your State Should Live

<p data-height="600" data-theme-id="0" data-slug-hash="qPrNQZ" data-default-tab="js" data-user="lacker" data-embed-version="2" class="codepen">See the Pen <a href="https://codepen.io/gaearon/pen/qPrNQZ">Thinking In React: Step 4</a> on <a href="http://codepen.io">CodePen</a>.</p>

OK, so we've identified what the minimal set of app state is. Next, we need to identify which component mutates, or *owns*, this state.

Remember: React is all about one-way data flow down the component hierarchy. It may not be immediately clear which component should own what state. **This is often the most challenging part for newcomers to understand,** so follow these steps to figure it out:

For each piece of state in your application:

  * Identify every component that renders something based on that state.
  * Find a common owner component (a single component above all the components that need the state in the hierarchy).
  * Either the common owner or another component higher up in the hierarchy should own the state.
  * If you can't find a component where it makes sense to own the state, create a new component simply for holding the state and add it somewhere in the hierarchy above the common owner component.

Let's run through this strategy for our application:

  * `ProductTable` needs to filter the product list based on state and `SearchBar` needs to display the search text and checked state.
  * The common owner component is `FilterableProductTable`.
  * It conceptually makes sense for the filter text and checked value to live in `FilterableProductTable`

Cool, so we've decided that our state lives in `FilterableProductTable`. First, add an instance property `this.state = {filterText: '', inStockOnly: false}` to `FilterableProductTable`'s `constructor` to reflect the initial state of your application. Then, pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as a prop. Finally, use these props to filter the rows in `ProductTable` and set the values of the form fields in `SearchBar`.

You can start seeing how your application will behave: set `filterText` to `"ball"` and refresh your app. You'll see that the data table is updated correctly.

## Step 5: Add Inverse Data Flow

<p data-height="600" data-theme-id="0" data-slug-hash="LzWZvb" data-default-tab="js,result" data-user="rohan10" data-embed-version="2" data-pen-title="Thinking In React: Step 5" class="codepen">See the Pen <a href="https://codepen.io/gaearon/pen/LzWZvb">Thinking In React: Step 5</a> on <a href="http://codepen.io">CodePen</a>.</p>

So far, we've built an app that renders correctly as a function of props and state flowing down the hierarchy. Now it's time to support data flowing the other way: the form components deep in the hierarchy need to update the state in `FilterableProductTable`.

React makes this data flow explicit to make it easy to understand how your program works, but it does require a little more typing than traditional two-way data binding.

If you try to type or check the box in the current version of the example, you'll see that React ignores your input. This is intentional, as we've set the `value` prop of the `input` to always be equal to the `state` passed in from `FilterableProductTable`.

Let's think about what we want to happen. We want to make sure that whenever the user changes the form, we update the state to reflect the user input. Since components should only update their own state, `FilterableProductTable` will pass callbacks to `SearchBar` that will fire whenever the state should be updated. We can use the `onChange` event on the inputs to be notified of it. The callbacks passed by `FilterableProductTable` will call `setState()`, and the app will be updated.

Though this sounds complex, it's really just a few lines of code. And it's really explicit how your data is flowing throughout the app.

## And That's It

Hopefully, this gives you an idea of how to think about building components and applications with React. While it may be a little more typing than you're used to, remember that code is read far more than it's written, and it's extremely easy to read this modular, explicit code. As you start to build large libraries of components, you'll appreciate this explicitness and modularity, and with code reuse, your lines of code will start to shrink. :)
