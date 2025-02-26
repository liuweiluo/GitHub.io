##  Virtual DOM 及 Diff 算法

### 1.  JSX 到底是什么
使用 React 就一定会写 JSX，JSX 到底是什么呢？它是一种由 React 团队创造的 JavaScript 语法的扩展，React 使用它来描述用户界面长成什么样子。虽然它看起来非常像 HTML，但它确实是 JavaScript 。在 React 代码执行之前，Babel 会对将 JSX 编译为 React API。

PS: 浏览器本身不可执行 JSX，需经过 Babel 将 JSX 编译为 React API 后才可执行。

```react
// JSX 语法
<div className="container">
  <h3>Hello React</h3>
  <p>React is great </p>
</div>
```

```react
// 经 Babel 编译后为 React API
React.createElement(
  "div",
  {
    className: "container"
  },
  React.createElement("h3", null, "Hello React"),
  React.createElement("p", null, "React is great")
);
```

这是 Babel 的测试网站：[Babel REPL](https://babeljs.io/repl)，它能更好地让我们理解 JSX 与 JS 之间的转换。

以下为总体流程：

![image](https://github.com/user-attachments/assets/66956b55-64c3-4931-b6d7-5406b1864218)


### 2. DOM 操作问题

在现代 web 应用程序中使用 JavaScript 操作 DOM 是必不可少的，但遗憾的是它比其他大多数 JavaScript 操作要慢的多。

大多数 JavaScript 框架对于 DOM 的更新远远超过其必须进行的更新，从而使得这种缓慢操作变得更糟和卡顿。

例如假设你有包含十个项目的列表，你仅仅更改了列表中的第一项，大多数 JavaScript 框架会重建整个列表，这比必要的工作要多十倍。

更新效率低下已经成为严重问题，为了解决这个问题，React 普及了一种叫做 Virtual DOM 的东西，Virtual DOM 出现的目的就是为了提高 JavaScript 操作 DOM 对象的效率。

### 3. 什么是 Virtual DOM

在 React 中，每个 DOM 对象都有一个对应的 Virtual DOM 对象，它是 DOM 对象的 JavaScript 对象表现形式，其实就是使用 JavaScript 对象来描述 DOM 对象信息，比如 DOM 对象的类型是什么，它身上有哪些属性，它拥有哪些子元素。

可以把 Virtual DOM 对象理解为 DOM 对象的副本，但是它不能直接显示在屏幕上。

```react
// JSX 形式
<div className="container">
  <h3>Hello React</h3>
  <p>React is great </p>
</div>
```

```react
// 经转换后的 Virtual DOM 对象
{
  type: "div",
  props: { className: "container" },
  children: [
    {
      type: "h3",
      props: null,
      children: [
        {
          type: "text",
          props: {
            textContent: "Hello React"
          }
        }
      ]
    },
    {
      type: "p",
      props: null,
      children: [
        {
          type: "text",
          props: {
            textContent: "React is great"
          }
        }
      ]
    }
  ]
}
```

### 4. Virtual DOM 如何提升效率

精准找出发生变化的 DOM 对象，只更新发生变化的部分。

在 React 第一次创建 DOM 对象后，会为每个 DOM 对象创建其对应的 Virtual DOM 对象，在 DOM 对象发生更新之前，React 会先更新所有的 Virtual DOM 对象，然后 React 会将更新后的 Virtual DOM 和 更新前的 Virtual DOM 进行比较，从而找出发生变化的部分，React 会将发生变化的部分更新到真实的 DOM 对象中，React 仅更新必要更新的部分。

Virtual DOM 对象的更新和比较仅发生在内存中，不会在视图中渲染任何内容，所以这一部分的性能损耗成本是微不足道的。

```react
<div id="container">
	<p>Hello React</p>
</div>
```

```react
<div id="container">
	<p>Hello Angular</p>
</div>
```

```react
const before = {
  type: "div",
  props: { id: "container" },
  children: [
    {
      type: "p",
      props: null,
      children: [
        { type: "text", props: { textContent: "Hello React" } }
      ]
    }
  ]
}
```

```react
const after = {
  type: "div",
  props: { id: "container" },
  children: [
    {
      type: "p",
      props: null,
      children: [
        { type: "text", props: { textContent: "Hello Angular" } }
      ]
    }
  ]
}
```

### 5. Virtual DOM 对象转化为真实 DOM 对象，即 Diff 过程

Diff过程可分为2种情况：

1.如果容器中没有旧的 DOM， 直接把 Virtual DOM 转换真实 DOM 后挂载在容器上即可 

2.如果容器中存在旧的 DOM， 要先把 Virtual DOM 与旧 DOM 的 Virtual DOM 进行对比后把不同部分更新到容器上（注意这里并非重新挂载） 

### 6. 实现一个精简版React框架（TinyReact）

#### 1.配置.babelrc文件，让 Babel 编译时使用 TinyReact.createElement 方法，否则默认使用 React.createElement
```
{
  "presets": [
    "@babel/preset-env",
    [
      "@babel/preset-react",
      {
        "pragma": "TinyReact.createElement"
      }
    ]
  ]
}
```

#### 2.实现 createElement 方法用于创建 Virtual DOM 对象 ，并在 TinyReact目录下 index.js 中导出该方法
``` JavaScript
export default function createElement(type, props, ...children) {
  const childElements = [].concat(...children).reduce((result, child) => {
    if (child !== false && child !== true && child !== null) {
      if (child instanceof Object) {
        result.push(child)
      } else {
        result.push(createElement("text", { textContent: child }))
      }
    }
    return result
  }, [])
  return {
    type,
    props: Object.assign({ children: childElements }, props),
    children: childElements
  }
}
```

#### 3.实现 render 方法用于 Virtual DOM 对象转化真实 DOM 对象，并在 TinyReact目录下 index.js 中导出该方法
``` JavaScript
import diff from "./diff"

export default function render(
  virtualDOM,
  container,
  oldDOM = container.firstChild
) {
  diff(virtualDOM, container, oldDOM)
}

```

#### 4.实现 diff 方法用于比对 新旧 Virtual DOM 对象

``` JavaScript
待补充
```
