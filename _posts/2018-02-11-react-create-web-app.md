---
layout:     post
title:     Web 应用前台快速搭建 ———— React + NodeJS 实现
category:  React
description: 这里介绍了从头开始搭建一个 Web 应用的过程，包括使用 React 实现前端，Bootstrap 构建 UI，以及一些集成工具的使用，如:yarn。
keywords:  React Bootstrap NodeJS yarn npm 
---
这里介绍了从头开始搭建一个 Web 应用的过程，包括使用 React 实现前端，NodeJS 实现后端，以及一些集成工具的使用，如: yarn。

相信现在在开发 React 应用时，已经没有人再手工在 HTML 文档里引入 React 的 js 源文件了。大家都是用 webpack + es6 来结合 React 开发前端应用。这个时候，我们可以手动使用 npm 来安装各种插件，来从头到尾自己搭建环境。

# 使用 create-react-app 初始化项目

比如：
* npm install react react-dom --save
* npm install babel babel-loader babel-core babel-preset-es2015
* babel-preset-react --save
* npm install babel webpack webpack-dev-server -g

虽然自己搭建的过程也是一个很好的学习过程，但是有时候难免遇到各种问题，特别是初学者，而且每次开发一个新应用，都要自己从头搭建，未免太繁琐。于是，有人根据自己的经验和最佳实践，开发了脚手架，避免开发过程中的重复造轮子和做无用功，从而节省开发时间。

类似这样的脚手架，比较多人用和关注的，一共有三个：
* react-boilerplate
* react-redux-starter-kit
* create-react-app

其中， create-react-app 由于其 Facebook 官方背景, 目前在 Github 上 star 数最多，相信使用的人也是最多的。

create-react-app 安装起来实在是太简单，只需要三条命令，就可以安装并启动。
```cmd
npm install -g create-react-app
create-react-app my-app
cd my-app/
npm start
```
## 结构清晰
所有的源码放到src目录下，什么配置文件，各种乱七八糟都不用管，你只需要专注开发就好了，create-react-app都给你处理好了。整个源码简单，又小又清爽！管理起来也方便！
```text
root
│  .gitignore
│  package.json
│  README.md
│  tree.txt
│  yarn.lock
│
├─node_modules(省略)│
├─public
│      favicon.ico
│      index.html
│      manifest.json
│
└─src
        App.css
        App.js
        App.test.js
        index.css
        index.js
        logo.svg
        registerServiceWorker.js
```
如果你使用过webpack-dev-server或webpack搭建过开发环境，你就会发现，create-react-app的开发环境也有类似webpack-dev-server的--inline --hot自动刷新的功能。

什么意思呢？

就是一旦源码文件，一更新，再保存之后，浏览器会自动刷新，让你能实时查看效果。你总要探究一下是怎么回事，难道create-react-app也用上了webpack-dev-server?翻看了一下源码，没有找到webpack.config.js文件，如果有使用webpack就应该有这个文件，好奇怪。

看了一下node_modules目录，也没找到webpack相关的东西。先源头入手，我是用npm start命令来运行项目的。就从package.json文件入手，它的内容是这样的：

看到了这行：
```json
"start": "react-scripts start"
```
react-scripts又是什么？

在node_modules目录中能找到它，它果然依赖了好多工具，其中就包括'webpack'。里面果然也有webpack的配置文件，也有好多脚本文件。原来它是 Facebook 开发的一个管理create-react-app 服务的工具。它隐藏了没必要的文件，大多数人的配置都是差不多的。除此之外，它还加入了eslint的功能。让你在开发过程中，更关注于代码，很不错。

## 编译命令
这个是create-react-app的一个大亮点，它能让你的应用骗译出在线上生产环境运行的代码，编译出来的文件很小，且文件名还带hash值，方便我们做cache，而且它还提供一个服务器，让我们在本地也能看到线上生产环境类似的效果，真的超级方便。

只需一行命令：
```cmd
npm run build
```
运行下面两条命令，可以查看线上生产环境的运行效果。
```cmd
npm install -g pushstate-server
pushstate-server build
```
编译好的文件都会放到build目录中。

## API proxy
在开发react应用时，难免与服务器进行数据交互，就是要跟api打交道。

这个时候，有一个问题。

api存在的服务器可能是跟react应用完全分开的，而且，开发环境跟线上环境又不太一样。

比如，开发环境中，你的react应用是跑在3000端口的，可是api服务可能跑在3001端口，这个时候，你跟api服务器交互的时候，可能会使用fetch或各种请求库，比如jquery的ajax。

这个时候可能会遇到CORS问题，毕竟端口不同，而线上环境却没有这个问题，因为你都控制线上环境的react应用和api应用，跑在同一个端口上。

按照以往思路，解决的方法可能是用环境变量，比如：
```javascript
const apiBaseUrl = process.env.NODE_ENV === 'development' ? 'localhost:3001' : '/'
```
但是这样搞起来，还是有些复杂，然而，create-react-app提供了一个超级简单的方法，只需要在package.json文件中，加一个配置项就可以了。

比如：
```json
"proxy": "http://localhost:3001/",
```
至于你用的是http的何种请求库，都是一样的，不用改任何代码。这个选项，只对开发环境有效，线上环境还是保持react应用和api应用同一个端口。


然而，在国内的网络环境下使用 npm 安装的速度太慢了，影响效率和心情。

# 用 yarn 替代 npm
yarn 的安装过程在官网上可以找到，此处略过不提。

## 初始化 react 项目
```cmd
yarn create react-app myappname
cd myappname
yarn start
```
执行 start 命令时可能报错，如果报错信息和 react-scripts 有关，则执行以下命令：
```text
yarn add react-scripts
```
这个命令也可以用来添加任意依赖模块。
比如：
```text
yarn add react-bootstrap
```

## 发布过程
```text
yarn build
```
执行完毕后，静态文件就都保存在 build/static 目录下了。

# UI
作为程序员而不是设计师，自然希望能够将更多的精力放在搭建应用的功能性上而不是站点的样式上面，这里我们选择使用已经建好的前端样式主题，选择 Start Bootsrtap 样式模板里面的 SB Admin 2。
安装必要的依赖包:
```text
yarn add react-bootstrap
yarn add bootstrap
```
在 public/index.html 中引入 react-bootstrap 的 css 和 js 库文件。
```html
<!-- Bootstrap Core CSS -->
  <link href="/css/bootstrap.min.css" rel="stylesheet">
  <!-- Custom CSS -->
  <link href="/css/clean-blog.min.css" rel="stylesheet">
  <link href="/css/cosmic-custom.css" rel="stylesheet">
  <!-- Custom Fonts -->
  <link href="//maxcdn.bootstrapcdn.com/font-awesome/4.1.0/css/font-awesome.min.css" rel="stylesheet" type="text/css">
  <link href="//fonts.googleapis.com/css?family=Lora:400,700,400italic,700italic" rel="stylesheet" type="text/css">
  <link href="//fonts.googleapis.com/css?family=Open+Sans:300italic,400italic,600italic,700italic,800italic,400,300,600,700,800" rel="stylesheet" type="text/css">
```

```html
  <script src="/js/jquery.min.js"></script>
  <script src="/js/bootstrap.min.js"></script>
  <script src="/js/clean-blog.min.js"></script>
  <script src="/dist/bundle.js"></script>
```
其他请参考: [这篇文章](https://segmentfault.com/a/1190000004703040)

# 多入口
每次新建项目都进行一次create-react-app，尤其有时候可能只是一个简单的页面，是比较繁琐的，因为create-react-app只支持一个文件入口，所以支持多入口是有必要。

要支持多入口，只需要对node_modules\react-scripts\config\paths.js进行一些改动。

读取npm命令时的参数：
```js
let appname = process.argv.find(arg => arg.indexOf('app=') === 0);
appname = appname ? appname.split('=')[1] : '';
```
然后修改一下输出路径：
```js
  appBuild: resolveApp(`build/${appname}`),
  appPublic: resolveApp(`public/${appname}`),
  appHtml: resolveApp(`public/${appname}/index.html`),
  appIndexJs: resolveApp(`src/${appname}/index.js`),
```
就这么简单，修改之后兼容之前的项目结构，也就是可以直接在src目录下添加代码文件，要使用多入口，需要分别在src和public目录下新建相同名称的目录，比如：
* public
  * react
  * react-router
* src
  * react
  * react-router

然后分别在目录下添加代码文件，具体可以参考react-starter的项目结构。

在npm命令时加入-- app=appname参数，示例如下：
```
npm start -- app=appname
npm run build -- app=appname
serve -s build/appname
```
