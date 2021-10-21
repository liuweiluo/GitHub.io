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

### 构建同构渲染

#### 构建流程

![image](https://user-images.githubusercontent.com/37037802/138077382-aafdcd77-d57d-4f74-9b7b-c28756226251.png)

之前的代码我们只处理了服务端渲染部分，如果想让服务端渲染的内容拥有客户端动态交互的能力，那我们必须有客户端脚本去接管服务端渲染后内容

由上图可知

- Server entry（服务端入口）通过Webpakc打包后得到 Server Bundle, 负责处理服务端渲染但不具有动态交互的能力
- Client entry（客户端入口）通过Webpakc打包后得到 Client Bundle, 负责处理客户端渲染并接管服务端渲染后内容把它激活成具有动态交互的能力的动态页面

### 源码结构

```
src
├── components
│ ├── Foo.vue
│ ├── Bar.vue
│ └── Baz.vue
├── App.vue
├── app.js # 通用 entry(universal entry)
├── entry-client.js # 仅运行于浏览器
└── entry-server.js # 仅运行于服务器

```

#### App.vue

```
<template>
    <!-- 客户端渲染的入口节点 -->
    <div id="app">
        <h1>拉勾教育</h1>
    </div>
</template>
<script>
export default {
    name: "App"
};
</script>
<style></style>
```

#### app.js

app.js 是我们应用程序的「通用 entry」。在纯客户端应用程序中，我们将在此文件中创建根 Vue 实例，并直接挂载到 DOM。但是，对于服务器端渲染(SSR)，责任转移到纯客户端 entry 文件。 app.js简单地使用 export 导出一个 createApp 函数：
```
import Vue from "vue";
import App from "./App.vue";
// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp() {
    const app = new Vue({
        // 根实例简单的渲染应用程序组件。
        render: h => h(App)
    });
    return { app };
}
```

#### entry-client.js

客户端 entry 只需创建应用程序，并且将其挂载到 DOM 中：

```
import { createApp } from "./app";
// 客户端特定引导逻辑……
const { app } = createApp();
// 这里假定 App.vue 模板中根元素具有 `id="app"`
app.$mount("#app");
```

#### entry-server.js
服务器 entry 使用 default export 导出函数，并在每次渲染中重复调用此函数。此时，除了创建和返回应用程序实例之外，它不会做太多事情 - 但是稍后我们将在此执行服务器端路由匹配 (server-sideroute matching) 和数据预取逻辑 (data pre-fetching logic)。

```
import { createApp } from "./app";
export default context => {
    const { app } = createApp();
    return app;
};
```

### 安装依赖
--- 忽略

### webpack配置文件

（1）初始化 webpack 打包配置文件
```
build
├── webpack.base.config.js # 公共配置
├── webpack.client.config.js # 客户端打包配置文件
└── webpack.server.config.js # 服务端打包配置文件
```

webpack.base.config.js

```
/**
 * 公共配置
 */
const VueLoaderPlugin = require('vue-loader/lib/plugin')
const path = require('path')
const FriendlyErrorsWebpackPlugin = require('friendly-errors-webpack-plugin')
const resolve = file => path.resolve(__dirname, file)

const isProd = process.env.NODE_ENV === 'production'

module.exports = {
  mode: isProd ? 'production' : 'development',
  output: {
    path: resolve('../dist/'),
    publicPath: '/dist/',
    filename: '[name].[chunkhash].js'
  },
  resolve: {
    alias: {
      // 路径别名，@ 指向 src
      '@': resolve('../src/')
    },
    // 可以省略的扩展名
    // 当省略扩展名的时候，按照从前往后的顺序依次解析
    extensions: ['.js', '.vue', '.json']
  },
  devtool: isProd ? 'source-map' : 'cheap-module-eval-source-map',
  module: {
    rules: [
      // 处理图片资源
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
            },
          },
        ],
      },

      // 处理字体资源
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          'file-loader',
        ],
      },

      // 处理 .vue 资源
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },

      // 处理 CSS 资源
      // 它会应用到普通的 `.css` 文件
      // 以及 `.vue` 文件中的 `<style>` 块
      {
        test: /\.css$/,
        use: [
          'vue-style-loader',
          'css-loader'
        ]
      },
      
      // CSS 预处理器，参考：https://vue-loader.vuejs.org/zh/guide/pre-processors.html
      // 例如处理 Less 资源
      // {
      //   test: /\.less$/,
      //   use: [
      //     'vue-style-loader',
      //     'css-loader',
      //     'less-loader'
      //   ]
      // },
    ]
  },
  plugins: [
    new VueLoaderPlugin(),
    new FriendlyErrorsWebpackPlugin()
  ]
}
```

webpack.client.config.js

```
/**
 * 客户端打包配置
 */
const { merge } = require('webpack-merge')
const baseConfig = require('./webpack.base.config.js')
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin')

module.exports = merge(baseConfig, {
  entry: {
    app: './src/entry-client.js'
  },

  module: {
    rules: [
      // ES6 转 ES5
      {
        test: /\.m?js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
            cacheDirectory: true,
            plugins: ['@babel/plugin-transform-runtime']
          }
        }
      },
    ]
  },

  // 重要信息：这将 webpack 运行时分离到一个引导 chunk 中，
  // 以便可以在之后正确注入异步 chunk。
  optimization: {
    splitChunks: {
      name: "manifest",
      minChunks: Infinity
    }
  },

  plugins: [
    // 此插件在输出目录中生成 `vue-ssr-client-manifest.json`。
    new VueSSRClientPlugin()
  ]
})

```

webpack.server.config.js
```
/**
 * 服务端打包配置
 */
const { merge } = require('webpack-merge')
const nodeExternals = require('webpack-node-externals')
const baseConfig = require('./webpack.base.config.js')
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')

module.exports = merge(baseConfig, {
  // 将 entry 指向应用程序的 server entry 文件
  entry: './src/entry-server.js',

  // 这允许 webpack 以 Node 适用方式处理模块加载
  // 并且还会在编译 Vue 组件时，
  // 告知 `vue-loader` 输送面向服务器代码(server-oriented code)。
  target: 'node',

  output: {
    filename: 'server-bundle.js',
    // 此处告知 server bundle 使用 Node 风格导出模块(Node-style exports)
    libraryTarget: 'commonjs2'
  },

  // 不打包 node_modules 第三方包，而是保留 require 方式直接加载
  externals: [nodeExternals({
    // 白名单中的资源依然正常打包
    allowlist: [/\.css$/]
  })],

  plugins: [
    // 这是将服务器的整个输出构建为单个 JSON 文件的插件。
    // 默认文件名为 `vue-ssr-server-bundle.json`
    new VueSSRServerPlugin()
  ]
})

```

（2）在 npm scripts 中配置打包命令

```
"scripts": {
"build:client": "cross-env NODE_ENV=production webpack --config build/webpack.client.config.js",
"build:server": "cross-env NODE_ENV=production webpack --config build/webpack.server.config.js",
"build": "rimraf dist && npm run build:client && npm run build:server"
}
```

运行测试：

```
npm run build:client
```

![image](https://user-images.githubusercontent.com/37037802/138091576-92bfca9b-54f0-4ef5-9d4c-e308364d751d.png)


```
npm run build:server
```

![image](https://user-images.githubusercontent.com/37037802/138091845-5e8a2afa-4211-45a6-a5e4-559bb51b5e77.png)


```
npm run build
```

![image](https://user-images.githubusercontent.com/37037802/138091911-240cb156-cba7-41ce-a6f8-3ca03f94da9e.png)


### 启动应用

server.js
```
const Vue = require("vue");
const express = require("express");
const fs = require("fs");
const serverBundle = require("./dist/vue-ssr-server-bundle.json");
const template = fs.readFileSync("./index.template.html", "utf-8");
const clientManifest = require("./dist/vue-ssr-client-manifest.json");
//
const renderer = require("vue-serverrenderer").createBundleRenderer(serverBundle, {
    template,
    clientManifest
});
const server = express();
server.use("/dist", express.static("./dist"));
server.get("/", (req, res) => {
    renderer.renderToString(
        {
            title: "liuweiluo",
            meta: `
<meta name="description" content="liuweiluo">
`
        },
        (err, html) => {
            if (err) {
                return res.status(500).end("Internal Server Error.");
            }
            res.setHeader("Content-Type", "text/html; charset=utf8");
            res.end(html);
        }
    );
});
server.listen(3000, () => {
    console.log("server running at port 3000.");
});
```

### 解析渲染流程

- 服务端如何进行渲染

  当客户端请求后，就被服务端路由匹配(上述server.js文件可查看服务端路由)，调用renderer.renderToString()开始进行渲染，我们都知道renderToString()会接收Vue实例进行渲染，但是在上述server.js中并   没有Vue实例，那到底是否有Vue实例呢？答案是有的，只不过此时Vue实例是通过createBundleRenderer()创建，而createBundleRenderer()第一个参数serverBundle（服务端打包后的vue-ssr-server-         bundle.json文件）。
  
  vue-ssr-server-bundle.json
  
  ![image](https://user-images.githubusercontent.com/37037802/138230335-f3a16741-7054-4798-8263-30f5fdc0f0e5.png)

  - entry：入口
  - files：所有构建结果资源列表
  - maps：源代码 source map 信息

  当调用renderer.renderToString()进行渲染之前，renderer已经加载了serverBundle里的entry（这里为server-bundle.js），从而得到在server-entry.js里创建的Vue实例并同时进行渲染，把渲染后结果注入   template模板中，最后把渲染后的结果发送给客户端，这是就是服务端渲染过程。
  
- 客户端如何渲染并激活服务端渲染后的内容
  
  我们发现服务端发送过来的内容会包含了客户端脚本（之前测试是不会的），那服务端是如何知道需要加载的客户脚本？
  
  其实是通过我们配置createBundleRenderer()时的clientManifest（客户端打包后的vue-ssr-client-manifest.json文件）里查找得到的，在服务端渲染时会把initial资源文件自动注入到template模板中
   <!--vue-ssr-outlet--> 后面，从而把需要的客户端脚本注入。
   
   ![image](https://user-images.githubusercontent.com/37037802/138243244-c3f795b3-de59-4910-b0e4-b801a6251b69.png)

    <!--vue-ssr-outlet-->
  
   vue-ssr-client-manifest.json
   
  ![image](https://user-images.githubusercontent.com/37037802/138237973-9a5ba1e3-7d33-411c-b28c-c873c853e86d.png)
  
  - publicPath：访问静态资源的根相对路径，与 webpack 配置中的 publicPath 一致
  - all：打包后的所有静态资源文件路径
  - initial：页面初始化时需要加载的文件，会在页面加载时配置到 preload 中
  - async：页面跳转时需要加载的文件，会在页面加载时配置到 prefetch 中
  - modules：项目的各个模块包含的文件的序号，对应 all 中文件的顺序；moduleIdentifier和all数组中文件的映射关系（modules对象是我们查找文件引用的重要数据）
  
  所谓客户端激活，指的是 Vue 在浏览器端接管由服务端发送的静态 HTML，使其变为由 Vue 管理的动态 DOM 的过程。
  
  由于服务器已经渲染好了 HTML，我们显然无需将其丢弃再重新创建所有的 DOM 元素。相反，我们需要"激活"这些静态的 HTML，然后使他们成为动态的（能够响应后续的数据变化）。

  如果你检查服务器渲染的输出结果，你会注意到应用程序的根元素上添加了一个特殊的属性 <div id="app" data-server-rendered="true">
  
  data-server-rendered 特殊属性，让客户端 Vue 知道这部分 HTML 是由 Vue 在服务端渲染的，并且应该以激活模式进行挂载。注意，这里并没有添加 id="app"，而是添加 data-server-rendered 属性：你需   要自行添加 ID 或其他能够选取到应用程序根元素的选择器，否则应用程序将无法正常激活。
  
  如何进行激活是VueDom处理的事情...
  
  激活参考 https://ssr.vuejs.org/zh/guide/hydration.html  
 
