---
id: accessibility
title: 可访问性
permalink: docs/accessibility.html
---

## 什么是可访问性

Web 可访问性（也被称为 [**a11y**](https://en.wiktionary.org/wiki/a11y)）指的是设计和创建可以被每个人使用的网站。可访问性支持是必要的，让辅助技术来解释网页。

React 完全支持创建可访问性网站，通常使用标准的 HTML 技术。

## 标准和指南

### WCAG

[网络内容可访问性指南](https://www.w3.org/WAI/intro/wcag)为创建可访问性站点提供了指导。

下面的 WCAG 列表提供了一个概览：

- [Wuhcag 的 WCAG 列表](https://www.wuhcag.com/wcag-checklist/)
- [WebAIM 的 WCAG 列表](http://webaim.org/standards/wcag/checklist)
- [A11Y 项目的列表](http://a11yproject.com/checklist.html)

### WAI-ARIA
[Web Accessibility Initiative - Accessible Rich Internet Applications](https://www.w3.org/WAI/intro/aria)文档包含了构建完全可访问的 JavaScript 部件的技术

需要注意的是，JSX 完全支持所有的 `aria-*` HTML 属性。尽管 React 中大多数的 DOM 属性都是小驼峰命名的，但是这些属性应该是小写的：

```javascript{3,4}
<input
  type="text" 
  aria-label={labelText}
  aria-required="true"
  onChange={onchangeHandler}
  value={inputValue}
  name="name"
/>
```

## 语义化 HTML
语义化 HTML 是 Web 应用程序可访问性的基础。使用各种 HTML 元素来加强网站信息的含义，通常会使得网站易于访问。

- [MDN 上的 HTML 元素参考](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

为了让 React 代码正常工作，我们有时会打破 HTML 语义化，在 JSX 中添加 `<div>` 元素，特别是在使用列表（`<ol>`、`<ul>` 和 `<dl>`）以及 HTML `<table>` 的时候。在这些情况下，我们应该使用 React Fragments 来将多个元素组合在一起。

当需要 `key` prop 时，使用 `<Fragment>`：

```javascript{1,8,11}
import React, { Fragment } from 'react';

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // 没有 `key` 会触发 React 的 key 警告
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

也可以在任何地方使用 `<></>`：

```javascript
function ListItem({ item }) {
  return ( 
    <>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>>
    </>
  );    
}
``` 

## 可访问表单

### 标签
每个 HTML 表单控件，例如 `<input>` 和 `<textarea>`，都需要打上可访问标签。我们需要提供描述性标签展示给屏幕阅读器。

下列资源向我们展示了如何去做：

- [W3C 如何标记元素的说明](https://www.w3.org/WAI/tutorials/forms/labels/)
- [WebAIM 如何标记元素的说明](http://webaim.org/techniques/forms/controls)
- [Paciello 小组阐述的可访问性名称](https://www.paciellogroup.com/blog/2017/04/what-is-an-accessible-name/)

虽然这些标准的 HTML 实践可以直接在 React 中使用，需要注意的是，在 JSX 中，`for` 属性被写作 `htmlFor`:

```javascript{1}
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name"/>
```

### 告知用户异常

异常情况需要被所有用户理解。下面的链接向我们展示了如何将错误文本展示给屏幕阅读器：

- [W3C 演示用户通知](https://www.w3.org/WAI/tutorials/forms/notifications/)
- [WebAIM 检查表单验证](http://webaim.org/techniques/formvalidation/)

## 焦点控件

确保你的应用完全可以仅通过键盘进行操作：

- [WebAIM 谈论键盘可访问性](http://webaim.org/techniques/keyboard/)

### 键盘聚焦和焦点边框

键盘聚焦指的是 DOM 中键盘选中，用于接受输入的当前元素。类似下面图片中的焦点边框，键盘聚焦到处可见：

<img src="../images/docs/keyboard-focus.png" alt="Blue keyboard focus outline around a selected link." />

如果你想用其他焦点边框，只能用 CSS 移除这个边框，例如设置 `outline: 0`。

### 跳转到期望内容的机制

提供一个机制来让用户跳过应用中之前的导航部分，因为这有助于加速键盘导航。

Skiplinks 或者 Skip Navigation Links 隐藏在导航链接中，只有当用户用键盘与页面进行交互时可见。他们可以很容易地通过页面内部锚点和一些样式来实现：

- [WebAIM - Skip Navigation Links](http://webaim.org/techniques/skipnav/)

同样，使用标记元素和角色来划分页面区域，例如 `<main>` 和 `<aside>`，因为辅助技术允许用户快速导航到这些部分。

阅读更多关于使用这些元素来增强可访问性的信息：

- [Deque 大学 —— HTML 5 和 ARIA 标记](https://dequeuniversity.com/assets/html/jquery-summit/html5/slides/landmarks.html)

### 编程式焦点管理

我们的 React 应用会在运行时持续修改 HTML DOM，有时会导致键盘焦点丢失或者聚焦到了未知的元素上。为了修复这个问题，我们需要编程式微调键盘焦点到正确的方向。例如，在模态框窗口关闭后，重设键盘焦点到打开它的按钮上。

MDN 研究和描述了如何建立 [JavaScript 键盘导航部件](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Keyboard-navigable_JavaScript_widgets)。

在 React 中设置焦点，我们可以使用 [DOM 元素的 ref](refs-and-the-dom.html)。

我们首先在类组件的 JSX 中为某个元素指定一个 ref：

```javascript{2,6}
render() {
  // 使用 `ref` 回调在实例字段（例如，this.textInput）中存储对文本输入 DOM 元素的引用。
  return (
    <input
      type="text"
      ref={(input) => { this.textInput = input; }} />
  );
}
```

然后，我们可以在需要时在组件的某个地方聚焦它：

 ```javascript
 focus() {
   // 使用原始 DOM API 聚焦文本输入
   this.textInput.focus();
 }
 ```
 
有些时候，父组件需要设置焦点到子组件中的某个元素。尽管我们可以给[类组件创建 ref](refs-and-the-dom.html#adding-a-ref-to-a-class-component)，但在使用函数组件以及[通过高阶组件使用 ref](higher-order-components.html#refs-arent-passed-through)时，我们需要一个模式来让它工作。为了确保父组件总是可以访问 ref，我们需要给子组件传递一个回调 prop 来将子组件的 [ref 暴露给父组件](refs-and-the-dom.html#exposing-dom-refs-to-parent-components)。

```js
// 通过回调 prop 来暴露 ref
function Field({ inputRef, ...rest }) {
  return <input ref={inputRef} {...rest} />;
}

// 在父类组件的 render 函数中...
<Field
  inputRef={(inputEl) => {
    // 这个回调会作为普通 prop 传递
    this.inputEl = inputEl
  }}
/>

// 现在你可以根据回调设置焦点
this.inputEl.focus();
```

一个比较不错的焦点管理示例是 [react-aria-modal](https://github.com/davidtheclark/react-aria-modal)。这是一个比较少见的完全可访问的模态框窗口的例子。它不仅仅给初始聚焦到了取消按钮（阻止用户意外地通过键盘触发成功操作）以及在模态框内部启用键盘聚焦，还重置焦点回到起初触发模态框的元素上。

> 注意：
>
> 虽然这是一个很重要的可访问性功能，但它也是一个应该审慎使用的技术。在遇到中断时使用它来修复键盘焦点，而不是尝试和预测用户如何使用应用。

## 更为复杂的部件

更复杂的用户体验并不意味着更少的可访问性。尽管通过尽可能地接近 HTML 编码来实现可访问性是最容易的，但是，即使最复杂的部件也可以进行可访问性编码。

在这里我们需要了解 [ARIA 角色](https://www.w3.org/TR/wai-aria/roles) 以及 [ARIA 状态和属性](https://www.w3.org/TR/wai-aria/states_and_properties)。这些工具箱涵盖了 JSX 完全支持的 HTML 属性，使得我们建立完全可访问、功能强大的 React 组件。


每一种部件都有一个特定的设计模式，并由用户和用户代理以某种方式运行：

- [WAI-ARIA 开发实践 —— 设计模式和部件](https://www.w3.org/TR/wai-aria-practices/#aria_ex)
- [Heydon Pickering —— ARIA 示例](http://heydonworks.com/practical_aria_examples/)
- [包容性组件](https://inclusive-components.design/)

## 其他要点

### 设置语言

声明页面文字语言，因为屏幕阅读器软件使用它来选择正确的发音设置：

- [WebAIM —— 文档语言](http://webaim.org/techniques/screenreader/#language)

### 设置文档标题

设置文档 `<title>` 来正确描述当前页面内容，因为这可以确保用户对当前页面的理解：

- [WCAG —— 了解文档标题需求](https://www.w3.org/TR/UNDERSTANDING-WCAG20/navigation-mechanisms-title.html)

我们可以使用 [React 文档标题组件](https://github.com/gaearon/react-document-title)来进行设置。

### 颜色对比度

确保网站上所有的可读文本都有足够的色彩对比度，以保障低视力用户的最大可读性：

- [WCAG —— 了解颜色对比度需求](https://www.w3.org/TR/UNDERSTANDING-WCAG20/visual-audio-contrast-contrast.html)
- [关于颜色对比度的以及为何需要重新审视它](https://www.smashingmagazine.com/2014/10/color-contrast-tips-and-tools-for-accessibility/)
- [A11yProject —— 什么是颜色对比度](http://a11yproject.com/posts/what-is-color-contrast/)

手动计算网站中所有用例的适当颜色组合可能非常繁琐，因此你可以[使用 Colorable 来计算整个可访问的调色板](http://jxnblk.com/colorable/)。

下面提到的 aXe 和 WAVE 工具都包含颜色对比度测试，以及报告对比度误差。

如果你想扩展对比度测试功能，你可以使用这些工具：
- [WebAIM —— 颜色对比度检查器](http://webaim.org/resources/contrastchecker/)
- [Paciello 组织 —— 颜色对比度分析器](https://www.paciellogroup.com/resources/contrastanalyser/)

## 开发及测试工具

我们可以使用许多工具来协助创建可访问的 web 应用程序。

### 键盘

目前最简单也是最重要的检查是测试整个网站是否可以通过键盘使用。做法如下：

1. 拔掉你的鼠标。
1. 使用 `Tab` 和 `Shift+Tab` 来浏览。
1. 使用 `Enter` 激活元素。
1. 如果需要，使用方向键来与某些元素交互，例如菜单和下拉列表。

### 开发助手

我们可以直接在 JSX 代码中检查一些可访问性功能。通常情况下，在可以识别 JSX 语法的 IDE 中已经提供了针对 ARIA 角色、状态和属性的智能检测。我们也可以采用以下工具：

#### eslint-plugin-jsx-a11y

ESLint 的 [eslint-plugin-jsx-a11y](https://github.com/evcohen/eslint-plugin-jsx-a11y) 插件提供了对于 JSX 中可访问性问题的 AST 检测反馈。许多 IDE 允许你将这些发现直接集成到代码分析和源码窗口中。

[Create React App](https://github.com/facebookincubator/create-react-app) 包含了这个插件并激活了部分规则。如果想要支持更多的可访问性规则，你可以在项目的根目录创建一个包含了以下内容的 `.eslintrc` 文件。

  ```json
  {
    "extends": ["react-app", "plugin:jsx-a11y/recommended"],
    "plugins": ["jsx-a11y"]
  }
  ```

### 浏览器中的可访问性测试

很多现成的工具都可以在浏览器中运行对于页面的可访问性检测。因为它们仅仅可以测试 HTML 中技术上的可访问性，所以请配合本文提到的其他可访问性检测工具一起使用。

#### aXe、aXe-core 和 react-axe

Deque Systems 提供的 [aXe-core](https://www.deque.com/products/axe-core/) 可以对你的应用进行自动化和端到端的可访问性测试。这个模块包含了对 Selenium 的集成。

[可访问性引擎](https://www.deque.com/products/axe/) 或者 aXe 是一个基于 `aXe-core` 的可访问性检测浏览器拓展工具。

你也可以在开发和调试模式下使用 [react-axe](https://github.com/dylanb/react-axe) 模块来在控制台直接输出这些可访问性发现。

#### WebAIM WAVE

[Web Accessibility Evaluation Tool](http://wave.webaim.org/extension/) 是另外一个可访问性浏览器拓展工具。

#### 可访问性检测器和可访问性树

[可访问性树](https://www.paciellogroup.com/blog/2015/01/the-browser-accessibility-tree/) 是 DOM 树的子集，它包含了每个 DOM 元素应该暴露给辅助技术（例如屏幕阅读器）的可访问性对象。

在某些浏览器中我们可以很容易地查看到可访问性树中每个元素的可访问性信息：

- [在 Chrome 中激活可访问性检测器](https://gist.github.com/marcysutton/0a42f815878c159517a55e6652e3b23a)
- [在 OS X 的 Safari 中使用可访问性检测器](https://developer.apple.com/library/content/documentation/Accessibility/Conceptual/AccessibilityMacOSX/OSXAXTestingApps.html)

### 屏幕阅读器

使用屏幕阅读器进行测试应该是可访问性测试的一部分。

请注意浏览器/屏幕阅读器的结合。建议在浏览器中选择最适合的屏幕阅读器来测试你的应用程序。

#### Firefox 的 NVDA

[NonVisual Desktop Access](https://www.nvaccess.org/) 或者 NVDA 是一个广泛使用的开源屏幕阅读器。

关于如何更好地使用 NVDA 请参考以下指南：

- [WebAIM —— 使用 NVDA 来评估可访问性](http://webaim.org/articles/nvda/)
- [Deque —— NVDA 快捷键](https://dequeuniversity.com/screenreaders/nvda-keyboard-shortcuts)

#### Safari 的 VoiceOver

VoiceOver 一个集成在苹果设备上的屏幕阅读器。

关于如激活合和使用 VoiceOver 请参考以下指南：

- [WebAIM —— 使用 VoiceOver 来评估可访问性](http://webaim.org/articles/voiceover/)
- [Deque —— VoiceOver OS X 快捷键](https://dequeuniversity.com/screenreaders/voiceover-keyboard-shortcuts)
- [Deque —— VoiceOver iOS 快捷键](https://dequeuniversity.com/screenreaders/voiceover-ios-shortcuts)

#### Internet Explorer 的 JAWS

[Job Access With Speech](http://www.freedomscientific.com/Products/Blindness/JAWS) 或者 JAWS 是一个广泛使用在 Windows 上的屏幕阅读器。

关于如何更好地使用 JAWS 请参考以下指南：

- [WebAIM —— 使用 JAWS 来评估可访问性](http://webaim.org/articles/jaws/)
- [Deque —— JAWS 快捷键](https://dequeuniversity.com/screenreaders/jaws-keyboard-shortcuts)