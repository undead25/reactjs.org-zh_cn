---
id: reconciliation
title: 协调
permalink: docs/reconciliation.html
---

React 提供声明式的 API 来让你无需关心每次更新背后的变化。这让应用的编写容易了很多，但在 React 内部是如何实现的可能并不明显。这篇文章解释了我们在 React 的 diff 算法中所作出的选择，以便让组件的更新可预测，并使得高性能应用足够快。

## 动机

当你使用 React 时，你可以把 `render()` 函数想象成创建一棵 React 元素树。在下一次 state 或者 props 更新时，`render()` 函数将返回一棵不同的 React 元素树。React 需要计算如何高效地更新 UI 来匹配最新的树。

有一些通用方案来解决将一棵树转换为另一棵树的最小操作数的算法问题。然而，[最先进的算法](http://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf)的复杂度为O(n<sup>3</sup>)，其中 n 是树中元素的数量。

如果我们在 React 中这样使用，那么展示 1000 个元素需要进行十亿次比较，这太昂贵了。React 基于两个假设来实现一个启发式的 O(n) 算法：

1. 两个不同类型的元素将产生不同的树。
2. 开发者可以通过 `key` 属性来示意哪些子元素是稳定的。

实际上，这些假设几乎适用于大部分使用场景。

## Diff 算法

当对比两棵树时，React 首先会对比它们的根元素。根元素的类型不同，其行为也会不同。

### 不同类型的元素

每当根元素具有不同的类型时，React 将拆掉旧的树并从头开始创建新的树。从 `<a>` 到 `<img>`、从 `<Article>` 到 `<Comment>` 或者从 `<Button>` 到 `<div>` —— 所有这些都会导致重建。

当树被拆掉时，旧的 DOM 节点会被销毁。组件实例会接收 `componentWillUnmount()`。当创建新的树时，新的 DOM 节点会被插入到 DOM 中。组件实例会接收 `componentWillMount()` 以及之后的 `componentDidMount()`。任何旧树的 state 都会丢失。

任何根元素下的组件也将会被卸载，而它们的 state 也会被销毁。例如，当进行对比时：

```xml
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

这将会销毁旧的 `Counter` 并重载一个新的。

### 同类型的 DOM 元素

当对比具有相同类型的 React DOM 元素时，React 会观察它们的属性，保持相同的底层 DOM 节点，仅更新变化的属性。例如：

```xml
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

通过对比这两个元素，React 知道只需要修改底层 DOM 节点的 `className`。

当更新 `style` 时，React 也知道只更新变化的属性。例如：

```xml
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

当在转换这两个元素时，React 知道需要修改的是 `color` 而不是 `fontWeight`。

在处理完 DOM 节点后，React 会递归处理其子节点。

### 同类型的组件元素

当组件更新时，其实例保持不变以便在渲染过程保留 state。React 更新底层组件实例的 props 来匹配新的元素，并在底层实例上调用 `componentWillReceiveProps()` 和 `componentWillUpdate()`。

然后，`render()` 方法被调用，同时 diff 算法会递归处理之前的结果和新的结果。

### 递归子节点

默认情况下，当递归 DOM 节点的子节点时，React 会同时迭代两个子节点列表，并在有差异时生成一个变更。

例如，当在子节点末尾添加一个元素时，两棵树的转换表现地很好：

```xml
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

React 将匹配两棵  `<li>first</li>` 树，并匹配两棵 `<li>second</li>` 树，然后插入 `<li>third</li>` 树。

如果你在开始插入元素，那就会使得性能更差。例如，这两棵树之间的转换就不好：

```xml
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

React 会改变每个子节点，而没有意识到可以保持 `<li>Duke</li>` 和 `<li>Villanova</li>` 子树。这种低效率可能是一个问题。

### Keys

React 支持 `key` 属性来解决这个问题。当子节点带有 key 时，React 使用 key 来匹配原始树和新树的子节点。例如，增加一个 `key` 到上面低效的例子中，这能让树的转换变得高效：

```xml
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

现在 React 知道 key 为 `'2014'`的元素是新的，而 key 为 `'2015'` 和 `'2016'` 是移过来的。

在实践中，找到一个 key 通常不难。你要显示的元素可能已经有了一个唯一的 ID，所以 key 可以来自于你的数据：

```js
<li key={item.id}>{item.name}</li>
```

如果情况不是这样，那可以将新的 ID 属性添加到你的模型中，或者对内容的某些部分进行 hash 来生成 key。key 必须在其兄弟节点中是唯一的，并不需要全局唯一。

不得已的情况下，你可以使用数组中的索引作为 key。如果数据项没有重新排序，可以这样做，但重新排序则会很慢。

当使用索引作为 key 时，重新排序也会导致组件的 state 问题。组件实例是根据其 key 进行更新和复用的。如果 key 是索引，移动其中一个数据就会改变这个索引。因此，像输入类的受控组件的 state 可能会搞混，并以意想不到的方式进行更新。

[这里](codepen://reconciliation/index-used-as-key)是一个在 CodePen 上使用索引作为 key 可能产生问题的例子，[这里](codepen://reconciliation/no-index-used-as-key)是同样例子的改进版，其展示了不使用索引作为 key 将解决排序和一些悬而未决的问题。

## 权衡

记住协调算法是一个实现细节是很重要的。React 可以在每次操作时重新渲染整个应用，其结果是一样的。需要清楚的是，在这种情况下，重新渲染意味着为所有组件调用 `render`，这并不意味着 React 会卸载和重载它们，它只会按照前面章节所述的规则进行处理。

为了更快地创建通用用例，我们会定期改进启发式。在目前的实现中，可以表明一个事实，那就是一个子树已经在兄弟节点之间移动了，但是你无法告知移动的具体位置，这个算法会重新渲染整个子树。

因为 React 依赖于启发式，如果背后的假设不符合，将会影响性能：

1. 该算法不会尝试匹配不同组件类型的子树。如果你发现两个输出非常相似的组件类型交替出现，那你可能需要使其类型相同。实际上，我们并没有发现这是一个问题。

2. key 应该是稳定的、可预测的和唯一的。不稳定 key（如 `Math.random()` 生产的）会使许多组件实例和 DOM 节点被不必要地重新创建，这会导致子组件中的性能下降和 state 丢失。