---
id: code-splitting
title: 代码分割
permalink: docs/code-splitting.html
---

## 打包

大多数 React 应用都会使用像 [Webpack](https://webpack.js.org/) 或者 [Browserify](http://browserify.org/) 这样的工具来对文件进行打包。打包是根据导入的文件来将它们合并到一个包文件的过程。然后这个包文件可以被包含在一个网页上，来一次性加载整个应用。

#### 示例

**App:**

```js
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```

```js
// math.js
export function add(a, b) {
  return a + b;
}
```

**Bundle:**

```js
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```

> 注意
>
> 你的 bundle 看起来肯定和这个不同。

如果你正在使用  [Create React App](https://github.com/facebookincubator/create-react-app)、[Next.js](https://github.com/zeit/next.js/)、[Gatsby](https://www.gatsbyjs.org/) 或者类似工具，你会有一个开箱即用的 Webpack 来打包你的应用。

如果没有使用，那你需要自己设置打包。例如，查看 Webpack 文档中的[安装](https://webpack.js.org/guides/installation/) 和 [入门](https://webpack.js.org/guides/getting-started/)指南。

## 代码分割

打包很好，但随着应用的增长，你的包体积也会随着增长。特别是你引入了大型的第三方库。你需要密切关注包中的代码以便你不会意外地使包体积变得非常大，这样你的应用需要花很长的时间来加载它。

为了避免大体积的包，最好先解决问题并开始“分割”你的包。[代码分割](https://webpack.js.org/guides/code-splitting/)是像 Webpack 和 Browserify（通过[factor-bundle](https://github.com/browserify/factor-bundle)）这样的打包工具支持的一个功能，它们可以创建多个在运行时可以被动态加载的包。

在你的应用中分割代码可以帮助你“懒加载”那些用户当前所需的东西，这可以明显提高应用的性能。虽然你没有减少应用的代码总量，但你避免了加载用户可能永远都不会需要的代码，并减少了首次加载所需的代码。

## `import()`

引入代码分割的最佳方式是通过动态的 `import()` 语法。

**Before:**

```js
import { add } from './math';

console.log(add(16, 26));
```

**After:**

```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

> 注意：
>
> 动态 `import()` 语法是一个 ECMAScript (JavaScript) [提案](https://github.com/tc39/proposal-dynamic-import)，
> 目前还不是语言标准的一部分。预计在不远的将来会被纳入到标准。

当 Webpack 遇到这个语法时，它会自动对你的应用进行代码分割。如果你正在使用 Create React App，那这已经配置好了，你可以[立即使用](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#code-splitting)。[Next.js](https://github.com/zeit/next.js/#dynamic-import) 也支持这个语法。


如果你是自己配置的 Webpack，你可能需要阅读 Webpack 的[代码分割指南](https://webpack.js.org/guides/code-splitting/)。你的 Webpack 配置应该是[长这样的](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269)。

当使用 [Babel](http://babeljs.io/) 时，你需要确保 Babel 可以解析动态导入语法，但不会对其进行转换。为此，你需要 [babel-plugin-syntax-dynamic-import](https://yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import)。

## 库

### React Loadable

[React Loadable](https://github.com/thejameskyle/react-loadable) 通过一个 React 友好的 API 封装了动态导入，来让你的应用通过一个组件来引入代码分割。

**Before:**

```js
import OtherComponent from './OtherComponent';

const MyComponent = () => (
  <OtherComponent/>
);
```

**After:**

```js
import Loadable from 'react-loadable';

const LoadableOtherComponent = Loadable({
  loader: () => import('./OtherComponent'),
  loading: () => <div>Loading...</div>,
});

const MyComponent = () => (
  <LoadableOtherComponent/>
);
```

React Loadable 可以帮助你创建[加载状态](https://github.com/thejameskyle/react-loadable#creating-a-great-loading-component)、[错误状态](https://github.com/thejameskyle/react-loadable#loading-error-states)、[超时](https://github.com/thejameskyle/react-loadable#timing-out-when-the-loader-is-taking-too-long)、[预加载](https://github.com/thejameskyle/react-loadable#preloading)和其他。它也可以帮助你在[服务端渲染](https://github.com/thejameskyle/react-loadable#------------server-side-rendering)的应用中进行代码分割。

## 基于路由的代码分割

决定在你的应用中的哪个位置引入代码分割可能会有点棘手。你需要确保你选择的地方会均匀拆分包，而不会破坏用户体验。

一个比较好的位置从路由开始。大部分人都习惯了页面跳转需要花费加载时间。你也倾向于立即重新渲染整个页面，这样你的用户不可能同时与页面上的其他元素进行交互。

下面有一个如何使用像 [React Router](https://reacttraining.com/react-router/) 和 [React Loadable](https://github.com/thejameskyle/react-loadable) 这样的库来设置基于路由的代码分割。

```js
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Loadable from 'react-loadable';

const Loading = () => <div>Loading...</div>;

const Home = Loadable({
  loader: () => import('./routes/Home'),
  loading: Loading,
});

const About = Loadable({
  loader: () => import('./routes/About'),
  loading: Loading,
});

const App = () => (
  <Router>
    <Switch>
      <Route exact path="/" component={Home}/>
      <Route path="/about" component={About}/>
    </Switch>
  </Router>
);
```
