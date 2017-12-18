---
id: installation
title: Installation
permalink: docs/installation.html
redirect_from:
  - "download.html"
  - "downloads.html"
  - "docs/tooling-integration.html"
  - "docs/package-management.html"
  - "docs/language-tooling.html"
  - "docs/environments.html"
next: hello-world.html
---

React 的灵活性使其可以应用在各种类型的项目上。你可以用它创建新的应用，也可以将它逐步地引入到现有的代码库中，而无需重写。

以下是几种上手的方法：

* [尝试 React](#trying-out-react)
* [创建新应用](#creating-a-new-application)
* [添加 React 到现有的应用](#adding-react-to-an-existing-application)

## 尝试 React

如果你只是想尝试以下，那你可以使用 CodePen。先从 [这个 Hello World 的示例代码](http://codepen.io/gaearon/pen/rrpgNB?editors=0010)开始，你不需要安装任何东西就可以通过修改代码来看到效果。

如果你喜欢使用自己的文本编辑器，那你也可以[下载这个 HTML 文件](https://raw.githubusercontent.com/reactjs/reactjs.org/master/static/html/single-file-example.html)，编辑并在本地浏览器中打开。它会缓慢地运行代码转换，所以不要在生产环境下使用。

如果你想用 React 来编写一个完整的应用，有两种流行的方式：使用 [Create React App](http://github.com/facebookincubator/create-react-app) 或者将它添加到现有的应用中。

## 创建新应用

[Create React App](http://github.com/facebookincubator/create-react-app) 是搭建一个新的 React 单页应用的最佳方式。它配置好了开发环境以便你可以使用最新的 JavaScript 特性，提供不错的开发体验，以及优化用于生产环境的应用。你需要在你的设备上安装 Node >= 6。

```bash
npm install -g create-react-app
create-react-app my-app

cd my-app
npm start
```

如果你安装了 npm 5.2.0+，也可以使用 [npx](https://www.npmjs.com/package/npx)。

```bash
npx create-react-app my-app

cd my-app
npm start
```

Create React App 不会处理后端逻辑或者数据库，它只是创建了一个前端的构建管道，所以你可以和任何后端搭配使用。它使用了像 [Babel](http://babeljs.io/) 和 [webpack](https://webpack.js.org/) 这样的构建工具，但使用起来零配置。

当你准备部署到生产环境时，运行 `npm run build` 将会在 `build` 文件夹中创建一个优化后的应用。你可以从 Create React App 的 [README](https://github.com/facebookincubator/create-react-app#create-react-app-) 和 [用户指南](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#table-of-contents) 中了解更多信息。

## 添加 React 到现有的应用

你不需要为了使用 React 重写你的应用。

我们建议在应用的一小部分中使用 React，例如某个独立部件，以便你可以验证它是否合适。

虽然 React 可以[在没有构建管道的情况下使用](/docs/react-without-es6.html)，但为了提高生产力，我们还是建议将它搭建起来。现代的构建管道通常包括：

* **包管理器**，例如 [Yarn](https://yarnpkg.com/) 或者 [npm](https://www.npmjs.com/)。这可以让你充分利用庞大的第三发包生态系统，并且很方便地安装或者更新它们。
* **构建工具**，例如 [webpack](https://webpack.js.org/) 或者 [Browserify](http://browserify.org/)。这可以让你编写模块化代码，并且将它打包来优化加载时间。
* **编译器**，例如 [Babel](http://babeljs.io/)。这可以让你编写在旧版本浏览器上仍然可用的，包含最新特性的 JavaScript 代码。

### 安装 React

>**注意：**
>
> 安装完成后，我们强烈建议配置[生产构建流程](/docs/optimizing-performance.html#use-the-production-build)，来确保生产环境正在使用最新版本的 React。

我们建议使用 [Yarn](https://yarnpkg.com/) 或者 [npm](https://www.npmjs.com/) 来管理前端的依赖关系。如果你还没有使用过包管理器，那 [Yarn 文档](https://yarnpkg.com/en/docs/getting-started) 是一个不错的入门指南。 

使用 Yarn 安装：

```bash
yarn init
yarn add react react-dom
```

使用 npm 安装：

```bash
npm init
npm install --save react react-dom
```

Yarn 和 npm 都是从 [npm 注册表](http://npmjs.com/)下载包。

### 使用 ES6 和 JSX

为了在 JavaScript 代码中使用 ES6 和 JSX，我们建议 React 搭配 [Babel](http://babeljs.io/) 使用。ES6 包含一系列的 JavaScript 新特性来简化开发，JSX 则是与 React 搭配使用的 JavaScript 拓展。

[Babel 配置说明](https://babeljs.io/docs/setup/)解释了如何在不同的构建环境下配置 Babel。请确保你已经安装了 [`babel-preset-react`](http://babeljs.io/docs/plugins/preset-react/#basic-setup-with-the-cli-) 和 [`babel-preset-env`](http://babeljs.io/docs/plugins/preset-env/)，并在你的 [`.babelrc` 配置文件](http://babeljs.io/docs/usage/babelrc/)中启用了它们，然后就可以开始使用 React 了。

### 使用 ES6 和 JSX 编写 Hello World

我们建议使用像 [webpack](https://webpack.js.org/) 或者 [Browserify](http://browserify.org/) 这样的构建工具。这可以让你编写模块化代码，并且将它打包来优化加载时间。

最基本的 React 示例：

```js
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

这段代码将内容渲染到 id 为 `root` 的 DOM 元素，所以在你的 HTML 文件中的某个地方需要有 `<div id="root"></div>`。

同样，你可以在使用了任何其他 JavaScript UI 库的现有应用中，将 React 组件渲染到某个 DOM 元素里面。

[了解更多关于在现有代码中集成 React 的信息](/docs/integrating-with-other-libraries.html#integrating-with-other-view-libraries)

### 开发和生成版本

默认情况下，React 包含了很多在开发中非常有用的警告。

**然而，它们使得React 的开发版本体积过大并且运行过慢，所以你应该在部署应用时使用生产版本**

了解[如何判断你的网站是否使用了正确版本的 React](/docs/optimizing-performance.html#use-the-production-build)，以及如何最有效地配置生产构建过程：

* [使用 Create React App](/docs/optimizing-performance.html#create-react-app)
* [使用独立文件](/docs/optimizing-performance.html#single-file-builds)
* [使用 Brunch](/docs/optimizing-performance.html#brunch)
* [使用 Browserify](/docs/optimizing-performance.html#browserify)
* [使用 Rollup](/docs/optimizing-performance.html#rollup)
* [使用 Webpack](/docs/optimizing-performance.html#webpack)

### 使用 CDN

如果你不想使用 npm 来管理客户端包，`react` 和 `react-dom` 也在其 `umd` 文件夹中提供了托管在 CDN 上的独立文件。

```html
<script crossorigin src="https://unpkg.com/react@16/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
```

上面的版本仅用于开发，并不适用于生产。压缩优化后的生产版本：

```html
<script crossorigin src="https://unpkg.com/react@16/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
```

想加载指定版本的 `react` 和 `react-dom`，请用版本号替换 `16`。

如果你使用的是 Bower，可以使用 `react` 包。

#### 为什么使用 `crossorigin` 属性？

如果通过 CDN 使用 React，我们建议保留 [`crossorigin`](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes) 属性：

```html
<script crossorigin src="..."></script>
```

我们还建议设置 `Access-Control-Allow-Origin: *` 的 HTTP 头来验证你正在使用的 CDN。

![Access-Control-Allow-Origin: *](../images/docs/cdn-cors-header.png)

这可以在 React 16 及更高的版本中启用更好的[错误处理体验](/blog/2017/07/26/error-handling-in-react-16.html)。
