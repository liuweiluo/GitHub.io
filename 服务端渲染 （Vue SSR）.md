## 服务端渲染 （Vue SSR）

### 是什么
- 官方文档：https://ssr.vuejs.org/
- Vue SSR（Vue.js Server-Side Rendering） 是 Vue.js 官方提供的一个服务端渲染（同构应用）解决方案
- 使用它可以构建同构应用
- 还是基于原有的 Vue.js 技术栈

> 官方文档的解释：Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。
服务器渲染的 Vue.js 应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在服务器和客户端上运行。

### 渲染一个 Vue 实例

首先我们来学习一下服务端渲染中最基础的工作：模板渲染。

说白了就是如何在服务端使用 Vue 的方式解析替换字符串。

在它的官方文档中其实已经给出了示例代码，下面我们来把这个案例的实现过程以及其中含义演示一下。

```
  npm install vue vue-server-renderer
```
```
// 第 1 步：创建一个 Vue 实例
const Vue = require("vue");
const app = new Vue({
    template: `<div>{{ message }}</div>`,
    data: {
        message: "Hello World"
    }
});
// 第 2 步：创建一个 renderer
const renderer = require("vue-server-renderer").createRenderer();
// 第 3 步：将 Vue 实例渲染为 HTML
renderer.renderToString(app, (err, html) => {
    if (err) throw err;
    console.log(html);
    // => <div data-server-rendered="true">Hello World</div>
});
// 在 2.5.0+，如果没有传入回调函数，则会返回 Promise：
renderer
    .renderToString(app)
    .then(html => {
        console.log(html);
    })
    .catch(err => {
        console.error(err);
    });  
```
### 与服务器集成

在 Node.js 服务器中使用时相当简单直接，例如 Express。

首先安装 Express 到项目中：

```
npm i express
```

然后使用 Express 创建一个基本的 Web 服务：

```
const express = require("express");
const app = express();
app.get("/", (req, res) => {
    res.send("Hello World!");
});
app.listen(3000, () => console.log("app listening at http://localhost:port"));

```
启动 Web 服务：

```
nodemon index.js
```

在 Web 服务中渲染 Vue 实例：

```
const Vue = require("vue");
const server = require("express")();
const renderer = require("vue-server-renderer").createRenderer();
server.get("*", (req, res) => {
    const app = new Vue({
        data: {
            url: req.url
        },
        template: `<div>访问的 URL 是： {{ url }}</div>`
    });
    renderer.renderToString(app, (err, html) => {
        if (err) {
            res.status(500).end("Internal Server Error");
            return;
        }
        res.end(`
<!DOCTYPE html>
<html lang="en">
<head>
<title>Hello</title>
<meta charset="UTF-8">
</head>
<body>${html}</body>
</html>
`);
    });
});
server.listen(8080);
```

### 使用页面模板

创建模板文件 index.template.html

![image](https://user-images.githubusercontent.com/37037802/138071803-bb722e0e-15c5-4264-8223-40c284127e22.png)

```
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>

<body>
  //此处标记内容会由渲染后的内容替换
  <!--vue-ssr-outlet-->
</body>

</html>
```

```
const Vue = require("vue");
const server = require("express")();
const fs = require("fs");
const renderer = require("vue-server-renderer").createRenderer({
    template: fs.readFileSync('./index.template.html','utf-8') //fs.readFileSync方法默认返回Buffer类型（二进制），需设置 utf-8 返回字符串，因为此处template需接收字符串类型
});
server.get("*", (req, res) => {
    const app = new Vue({
        data: {
            url: req.url
        },
        template: `<div>访问的 URL 是： {{ url }}</div>`
    });
    renderer.renderToString(app, (err, html) => {
        if (err) {
            res.status(500).end("Internal Server Error");
            return;
        }
        res.end(html); //这里html是已经通过模板解析后的字符串
    });
});
server.listen(8080);
```

### 模板使用外部数据

```
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  {{{ meta}}}
  <title>{{title}}</title>
</head>

<body>
  //此处标记内容会由渲染后的内容替换
  <!--vue-ssr-outlet-->
</body>

</html>
```

```
const Vue = require("vue");
const server = require("express")();
const fs = require("fs");
const renderer = require("vue-server-renderer").createRenderer({
    template: fs.readFileSync("./index.template.html", "utf-8") //fs.readFileSync方法默认返回Buffer类型（二进制），需设置 utf-8 返回字符串，因为此处template需接收字符串类型
});
server.get("*", (req, res) => {
    const app = new Vue({
        data: {
            url: req.url
        },
        template: `<div>访问的 URL 是： {{ url }}</div>`
    });
    renderer.renderToString(
        app,
        {
            title: "liuweiluo",
            meta: `<meta name="descripiton" content=""`
        },
        (err, html) => {
            if (err) {
                res.status(500).end("Internal Server Error");
                return;
            }
            res.end(html); //这里html是已经通过模板解析后的字符串
        }
    );
});
server.listen(8080);
```

###
