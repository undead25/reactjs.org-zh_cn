---
id: hello-world
title: Hello World
permalink: docs/hello-world.html
prev: cdn-links.html
next: introducing-jsx.html
redirect_from:
  - "docs/"
  - "docs/index.html"
  - "docs/getting-started.html"
  - "docs/getting-started-ko-KR.html"
  - "docs/getting-started-zh-CN.html"
---

开始使用 React 最简单的方式就是使用[这个在 CodePen 上的 Hello World 示例](codepen://hello-world)。你不需要安装任何东西，只要在新标签页中打开它，然后按照示例进行操作。如果你想使用本地开发环境，请查看[安装](/docs/try-react.html)章节。

最基本的 React 示例：

```js
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

它渲染了一个 “Hello, world!” 的标题。

接下来的几个章节将逐步向你介绍如何使用 React。我们将了解到 React 应用的构成模块：元素和组件。一旦掌握了它们，你就可以使用简单可复用的代码来创建复杂的应用。

## JavaScript 注意事项

React 是一个 JavaScript 库，所以我们假设你对 JavaScript 有一个基本的了解。如果你还不是很了解，我们建议你[温习下你的 JavaScript 知识](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)以便更轻松地学习 React。

我们在示例中也使用了一些 ES6 的语法。由于它们还比较新，我们也是尽量少用，但还是鼓励你去熟悉下 [`箭头函数`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)、[`类`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)、[`模板字符串`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals)、[`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) 和 [`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)。你可以使用 [Babel REPL](babel://es5-syntax-example) 来查看 ES6 的编译结果.
