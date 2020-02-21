---
layout:     post
title:     React 组件模式
category:  MVVM
description: 这里介绍了 React 组件模式
keywords:  React
---

组件是 React 的核心，因此了解如何利用它们对于创建出色的设计结构至关重要。 

# 什么是组件？

根据 reactjs.org 的说法：

    Components let you split the UI into independent, reusable pieces, and think about each piece in isolation.
    组件允许开发者将UI拆分为独立的，可重用的部分，并独立思考每个部分。

每次运行npm install react时，你将会得到：React 组件及其API。 与 JavaScript函数类似，组件接受名为 props 的输入并返回 React 元素，它描述（声明）用户界面（UI）应该是什么样子。 这就是 React 被称为声明性API的原因，因为你只需要告诉 React 你的 APP 的 UI 是什么样子，React 负责其余部分的工作。

## 组件的 API

React 组件的 API 有哪些呢？分为五大类：

- render
- state
- props
- context
- lifecycle events

虽然组件具有充分利用上述所有 API 的能力，但大多数情况下，你会发现某些组件倾向于仅使用其中的一部分 API，而其他组件使用另外的 API。 两个分类之间的分界线，称为有状态和无状态分量。 有状态组件通常使用有状态 API：render、state 和 lifecycle events，而无状态组件使用 render、props 和 context。

# 组件模式

组件模式是 React 组件的最佳使用实践，它被引入来分割数据或逻辑层以及UI或视图层。 通过在组件之间划分职责，可以创建更多可重用、可组合的组件，组成复杂的UI。 在构建要扩展的应用程序时，这一点尤为重要。

常见的组件模式有：

- Container （容器组件）
- Presentational（展示组件）
- Higher order components (HOC)（高阶组件）
- Render callback

# Container

Container 是你的数据或逻辑层，它利用有状态的 API。 借助生命周期事件，你可以将组件连接到 Redux 或 Mobx 等状态管理，并将数据和回调作为 props 传递给子组件。 在 Container 的 render 方法中，你可以组成由展示（子）组件组成的UI。 为了能够访问所有有状态API，容器必须是类（Class）组件而不是纯函数组件。

在下面的示例中，我们有一个名为 Greeting 的类组件，它具有状态，生命周期事件 componentDidMount 和 render。

```js
class Greeting extends React.Component {
  constructor() {
    super();
    this.state = {
      name: "",
    };
  }

  componentDidMount() {
    // AJAX
    this.setState(() => {
      return {
        name: "William",
      };
    });
  }

  render() {
    return (
      <div>
        <h1>Hello! {this.state.name}</h1>
      </div>
    );
  }
}
```

这时，此组件是有状态的类组件。 为了使 Greeting 成为容器组件，我们可以将 UI 拆分为展示组件，如下。

# Presentational
展示组件使用 props、render 和 context（无状态API），并且可以是语法上非常实用的无状态组件（纯函数）：
```js
const GreetingCard = (props) => {
  return (
    <div>
      <h1>Hello! {props.name}</h1>
    </div>
  )
}
```
展示组件仅从 props 接收数据和回调，由其容器或父组件提供。

容器和展示组件一起将逻辑和视图封装到其目标组件中：

```js
const GreetingCard = (props) => {
  return (
    <div>
      <h1>{props.name}</h1>
    </div>
  )
}

class Greeting extends React.Component {
  constructor() {
    super();
    this.state = {
      name: "",
    };
  }

  componentDidMount() {
    // AJAX
    this.setState(() => {
      return {
        name: "William",
      };
    });
  }

  render() {
    return (
      <div>
       <GreetingCard name={this.state.name} />
      </div>
    );
  }
}
```
如上所见，我们已将 Greeting 类组件中的纯展示部分移除到其自身的纯函数无状态组件中。对于复杂的 APP，这是至关重要的。

# Higher order components (HOC)

高阶组件是一类将组件作为参数并返回新组件的函数。 这是一种功能强大的模式，可以为任意的组件提供数据或方法，并可用于重用组件逻辑。 例如 react-router 和 Redux。 使用 react-router，可以调用 withRouter 继承作为 props 传递给组件的方法。 使用 Redux，可以访问通过 connect 注入的作为 props 传递的 actions。
例如：
```js
import {withRouter} from 'react-router-dom';

@withRouter
class App extends React.Component {
  constructor() {
    super();
    this.state = {path: ''}
  }

  componentDidMount() {
    let pathName = this.props.location.pathname;
    this.setState(() => {
      return {
        path: pathName,
      }
    })
  }

  render() {
    return (
      <div>
        <h1>Hi! I'm being rendered at: {this.state.path}</h1>
      </div>
    )
  }
}

export default App;
```
借助装饰器，用 react-router 提供的 withRouter 包裹我们的组件。 在App的生命周期事件 componentDidMount 中，使用 this.props.location.pathname 提供的值更新状态。 通过使用 withRoute 包裹我的组件，我的类组件现在可以通过props访问react-router 的方法，因此可以访问到 pathname。 还有其他很多例子不一一赘述。

# Render callbacks

与高阶组件类似，render callback 或 render props 用于共享或重用组件逻辑。 虽然许多开发人员更多地倾向使用 HOC 来重用逻辑，但是使用 render callback 有一些非常好的理由和优势，有兴趣可以具体来看 Michael Jackson 的演讲 Michael Jackson - Never Write Another HoC - YouTube。其中几个关键点：render callback 减少命名空间冲突，并更好地说明了逻辑的来源。

```js
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };
  }

  increment = () => {
    this.setState(prevState => {
      return {
        count: prevState.count + 1,
      };
    });
  };

  render() {
    return (
      <div onClick={this.increment}>{this.props.children(this.state)}</div>
    );
  }
}

class App extends React.Component {
  render() {
    return (
      <Counter>
        {state => (
          <div>
            <h1>The count is: {state.count}</h1>
          </div>
        )}
      </Counter>
    );
  }
}
```

在 Counter 类中，我们在 render 方法中嵌入 this.props.children 并将 this.state 作为参数。 在 App 类的下面，能够将组件包装在 Counter 组件中，因此可以访问 Counter 的逻辑。 渲染回调部分是第28行，通过 {state => ()} 可以自动访问上面的 Counter 状态。
