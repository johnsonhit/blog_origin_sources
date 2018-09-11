title: 今天开始学 React：建立一个简单页面
date: 2016/09/29
comments: true
tags: 
- Web 
- React
categories: 
- Web Front End
---

## 前言
由于公司的项目需要 React Native 的相关技术，之前已经开发过 React 创建 Native 的项目。但是，很久没做 Web 的项目了，觉得这是个机会，开始重拾前端，学习现代前端技术。
我是在 Mac OS X 环境进行学习的，Terminal + sublime + atom 是我学习使用的工具。

## 着手创建第一个 Web 页面
### 安装
安装 React 需要借助 npm 。在安装 node 之前推荐先安装 Homebrew
- Homebrew

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- node

```sh
brew install node
```

- create-react-app

```sh
npm install -g create-react-app
```

### Hello World
利用 **create-react-app** 是最佳的方式创建一个新的 React 单应用。

```sh
reate-react-app hello-world
cd hello-world
npm start
```
执行以上命令之后就会开启本地服务，并打开 http://localhost:3000/。

#### 项目目录结构
在 Terminal 中查看当前目录的文件结构：
```sh
➜  hello-world ls
README.md    node_modules package.json public       src
```
- README.md 说明文件
- node_modules node 依赖包
- package.json 依赖包的配置信息
- public 网页
- src 资源 

可以分别对该目录下的文件夹进行 ls 操作查看文件详情。
```sh
➜  hello-world ls public 
favicon.ico index.html
➜  hello-world ls src 
App.css     App.js      App.test.js index.css   index.js    logo.svg
```

#### 源码结构

- App.js

```JavaScript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
```
- index.html

```HTML
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
    <!--
      Notice the use of %PUBLIC_URL% in the tag above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start`.
      To create a production bundle, use `npm run build`.
    -->
  </body>
</html>
```
通过观察 App.js 和 index.html 文件可以得知，App 中利用 JSX 渲染的元素的父节点是 id 为 root 的 div 元素。

因此，要是想在 Root 里面添加子元素就需要在 App.js 中利用 JSX 编码。

### 整合进已有的应用
#### 安装 React DOM

```sh
npm install --save react react-dom
```
#### 通过 CDN

```sh
<script src="https://unpkg.com/react@15/dist/react.js"></script>
<script src="https://unpkg.com/react-dom@15/dist/react-dom.js"></script>
```

## 小结
- 学习技术的套路
我认为，React 的入门姿势还是很 easy 的，嗯，能创建 hello world 项目就算入门了。
这次的学习认识了 React 单应用的项目结构。刚开始学习应用层的开发，就是学习框架的使用。Web 开发如此，Native 的套路也是如此。但是，对于 React 的学习不能只停留在框架的使用层面，后面肯定需要学习 JavaScript ES6 和 node.js 语言原理，再往后面业务逻辑熟练之后就开始玩架构、玩性能。然而不论怎样终端的开发，最终都是对系统资源的使用，所以，了解操作系统原理才是最关键的，这样玩什么都很 High。

- 与 React Native 的区别
React 和 React Native 仍是有差别的。 React Native 存在 index.ios.js 这样的入口文件，作为应用的程序入口，进行 App 逻辑处理、UI 渲染等。而 React Web 应用则是还包含 HTML 模版的相关目录。
而在 React Native 中还有 StyleSheet 模块，这样就可以让 Native 用类似 CSS 的语法来渲染原生应用的样式。而在 React 的 Web 项目中样式使用的是标准 CSS 。
所以，**本来设想的 React Native 中的代码可否直接 Copy Paste 到 Web 的设想经验证失败了。**
毕竟，由于开发平台差异，还是很难做到一套代码多处复用的。
因此，还是要多多完善技能树，应对不同问题，给出不同解决方案。

- 跨平台的尝试
上面说，一套代码直接跨平台使用失败了。然而，目前已有大厂商提供了自己的跨平台解决方案，一套代码多处运行，比如携程开发了一套新的框架 Moles。
可以查看相关资料：
http://www.infoq.com/cn/articles/ctrip-moles
GitHub 地址：
https://github.com/nygard/class-dump/releases