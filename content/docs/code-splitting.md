---
id: code-splitting
title: 代码分割
permalink: docs/code-splitting.html
---

## 打包

大多数 React 应用都会使用像 [Webpack](https://webpack.js.org/) 或者 [Browserify](http://browserify.org/) 这样的工具来对文件进行打包。打包是根据导入的文件来将它们合并到一个 `bundle` 文件的过程。然后这个 `bundle` 可以被包含在一个网页上，来一次性加载整个应用。

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

Bundling is great, but as your app grows, your bundle will grow too. Especially if you are including large third-party libraries. You need to keep an eye on the code you are including in your bundle so that you don't accidentally make it so large that your app takes a long time to load.

To avoid winding up with a large bundle, it's good to get ahead of the problem and start "splitting" your bundle. [Code-Splitting](https://webpack.js.org/guides/code-splitting/) is a feature supported by bundlers like Webpack and Browserify (via [factor-bundle](https://github.com/browserify/factor-bundle)) which can create multiple bundles that can be dynamically loaded at runtime.

Code-splitting your app can help you "lazy-load" just the things that are currently needed by the user, which can dramatically improve the performance of your app. While you haven't reduced the overall amount of code in your app, you've avoided loading code that the user may never need, and reduced the amount of code needed during the initial load.

## `import()`

The best way to introduce code-splitting into your app is through the dynamic `import()` syntax.

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

> Note:
>
> The dynamic `import()` syntax is a ECMAScript (JavaScript)
> [proposal](https://github.com/tc39/proposal-dynamic-import) not currently
> part of the language standard. It is expected to be accepted within the
> near future.

When Webpack comes across this syntax, it automatically start code-splitting your app. If you're using Create React App, this is already configured for you and you can [start using it](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#code-splitting) immediately. It's also supported out of the box in [Next.js](https://github.com/zeit/next.js/#dynamic-import).

If you're setting up Webpack yourself, you'll probably want to read Webpack's [guide on code splitting](https://webpack.js.org/guides/code-splitting/). Your Webpack config should look vaguely [like this](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269).

When using [Babel](http://babeljs.io/), you'll need to make sure that Babel can parse the dynamic import syntax but is not transforming it. For that you will need [babel-plugin-syntax-dynamic-import](https://yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import).

## 库

### React Loadable

[React Loadable](https://github.com/thejameskyle/react-loadable) wraps dynamic imports in a nice, React-friendly API for introducing code splitting into your app at a given component.

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

React Loadable helps you create [loading states](https://github.com/thejameskyle/react-loadable#creating-a-great-loading-component), [error states](https://github.com/thejameskyle/react-loadable#loading-error-states), [timeouts](https://github.com/thejameskyle/react-loadable#timing-out-when-the-loader-is-taking-too-long), [preloading](https://github.com/thejameskyle/react-loadable#preloading), and more. It can even help you [server-side render](https://github.com/thejameskyle/react-loadable#------------server-side-rendering) an app with lots of code-splitting.

## 基于路由的代码分割

Deciding where in your app to introduce code splitting can be a bit tricky. You want to make sure you choose places that will split bundles evenly, but won't disrupt the user experience.

A good place is to start is with routes. Most people on the web are used to page transitions taking some amount of time to load. You also tend to be re-rendering the entire page at once so your users are unlikely to be interacting with other elements on the page at the same time.

Here's an example of how to setup route-based code splitting into your app using libraries like [React Router](https://reacttraining.com/react-router/) and [React Loadable](https://github.com/thejameskyle/react-loadable).

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
