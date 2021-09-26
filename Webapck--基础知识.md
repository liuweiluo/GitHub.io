
## Webapck--基础知识

### 工作模式

```
module.exports = {
  // 这个属性有三种取值，分别是 production、development 和 none。
  // 1. 生产模式下，Webpack 会自动优化打包结果；
  // 2. 开发模式下，Webpack 会自动优化打包速度，添加一些调试过程中的辅助；
  // 3. None 模式下，Webpack 就是运行最原始的打包，不做任何额外处理；
  mode: 'development',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist')
  }
}
```

### 加载器(Loader)

#### 加载器(Loader)是用于处理与加工打包过程中遇到的资源文件

#### 常用Loader分类

- 编译装换类Loader：此类Loader会把资源模块转换成Javascript代码

--- css-loader: 把CSS代码转换为Javascript代码

- 文件操作类Loader：此类Loader会把资源模块拷贝至输出目录，同时把这个资源的访问路径向外导出

--- file-loader

![image](https://user-images.githubusercontent.com/37037802/133782897-024a8fbe-44f5-4684-9ae4-2ecdbcb7de15.png)

- 代码检查类Loader：此类Loader会进行代码校验以达到统一风格和格式的效果

--- eslint-loader

#### file-loader与url-loader的区别

- file-loader是把资源拷贝至输出目录，同时把资源的访问路径向外导出，而url-loader直接把资源转为文本(例如:data-url)。

### Webpack与 ES 2015
- 因为模块打包需要，webpack只对import和export进行ES5的转换，其他ES6语法webpack并不能进行转换，若转换需要用到其他加载器，例如bable-loader

###  Webpack 加载资源的方式
- 遵循ES Modules标准的import声明
- 遵循CommonJs标准的require函数
- 遵循AMD标准的define函数和require函数
- Loader加载的非JavaScript也会触发资源加载：例如在安装了css-loader的环境下，样式代码中的@import指令和url函数。在安装了html-loader的环境下，HTML代码中图片标签的src属性

### webpack核心工作原理

![image](https://user-images.githubusercontent.com/37037802/133885949-9978a9d3-dce1-443e-a387-77646af993b3.png)

以普通的前端项目为例，在项目当中会存在各种各样的资源文件（例如:.js、.html、.css、.png、.scss.....），webpack会根据我们配置文件中的入口配置，找到对应的入口文件（一个或多个js文件）作为入口，然后会根据代码中出现import或require的语句去加载解析该文件的的所有依赖模块，然后解析的依赖模块又会继续解析它对应的依赖模块，一直递归至结束后就会形成项目中所用的资源文件之间的依赖关系（依赖树）如下图

![image](https://user-images.githubusercontent.com/37037802/133887239-b2abe3b0-9295-488a-9901-377282f5e78e.png)

有了这个依赖关系树过后， Webpack会遍历（递归）这个依赖树，找到每个节点对应的资源文件，然后根据配置选项中的Loader配置，交给对应的Loader去加载这个模块，最后将加载的结果放入 bundle.js（打包结果）中，从而实现整个项目的打包

![image](https://user-images.githubusercontent.com/37037802/133887777-0e47273b-b4ab-474a-99aa-b8b87e0ee874.png)

<strong>整个打包过程中，Loader机制起了很重要的作用，因为如果没有Loader的话，Webpack就无法实现各种各样类型的资源文件加载</strong>，那Webpack也就只能算是一个用来合并JS模块代码的工具了。

### 插件(Plugin)

插件机制的目的是增强Webpack在项目自动化构建方面的能力，Loader就是负责完成项目中各种各样资源模块的加载，从而实现整体项目的模块化，而Plugin则是用来解决项目中除了资源模块打包以外的其他自动化工作，所以说Plugin的能力范围更广，用途自然也就更多。

#### 先介绍几个插件最常见的应用场景
- 实现自动在打包之前清除 dist 目录（上次的打包结果）--- clean-webpack-plugin
- 拷贝不需要参与打包的资源文件到输出目录 --- copy-webpack-plugin
- 压缩 Webpack 打包完成后输出的代码 --- OptimizeCssAssetsWebpackPlugin
- 提取CSS到单个文件 --- MiniCssExtractPlugin

总之，有了Plugin，Webpack几乎’无所不能’，借助插件我们可以轻松实现前端工程化中的绝大多数功能，使得很多初学者把Webpack和前端工程化混淆。

#### 常见插件

<strong>clean-webpack-plugin</strong> ---- 自动清除输出目录的插件

背景：通过之前的尝试，我们发现webpack每次打包的结果就是直接覆盖到dist目录。但dist目录中可能存在一些在上次打包操作时遗留的文件，此时会出现一些已经移除的文件还冗余其中的情况，最终导致线上部署的时候出现多余文件，这显然很不合理。

安装：$ npm install clean-webpack-plugin --save-dev

案例：
```
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    publicPath: 'dist/'
  },
  
  .....
  
  plugins: [
    new CleanWebpackPlugin()
  ]
}

```

<strong> html-webpack-plugin</strong> ---- 用于生成 HTML 的插件

背景：首先，我们要知道html文件，一般都是通过硬编码的方式，单独存放在项目根目录中。此时会有问题：项目发布时，我们同时要发布根目录下的html+dist里的所有打包文件，还要确保上线后html代码中的资源文件路径正确，非常麻烦；如果打包结果输出的目录或者文件名发生变化，html代码中所对应的script标签也需要我们手动修改。

解决这个问题最好的办法就是让webpack在打包的同时，自动生成对应的html文件，让webpack将打包的bundle文件自动引入页面中；此时需要html-webpack-plugin

安装：$ npm install html-webpack-plugin --save-dev

案例：
```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    // publicPath: 'dist/'
  },
  
  ......
  
  plugins: [
    // 用于生成 index.html
    new HtmlWebpackPlugin({
      title: 'Webpack Plugin Sample',
      meta: {
        viewport: 'width=device-width'
      },
      template: './src/index.html'
    }),
    // 用于生成 about.html
    new HtmlWebpackPlugin({
      filename: 'about.html'
    })
  ]
}

```

<strong>MiniCssExtractPlugin</strong> ---- 提取CSS到单个文件的插件

```
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
  mode: 'none',
  entry: {
    main: './src/index.js'
  },
  output: {
    filename: '[name].bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 'style-loader', // 将样式通过 style 标签注入
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin()
  ]
}
```

<strong>copy-webpack-plugin</strong> ---- 用于复制文件的插件

在我们的项目中一般还有一些不需要参与构建的静态文件，那它们最终也需要发布到线上，例如网站的 favicon、robots.txt 等。

一般我们建议，把这类文件统一放在项目根目录下的 public 或者 static 目录中，我们希望 Webpack 在打包时一并将这个目录下所有的文件复制到输出目录。

对于这种需求，我们可以使用 copy-webpack-plugin 插件来帮我们实现。

### 配置多入口打包----多页应用程序

案例

![image](https://user-images.githubusercontent.com/37037802/134323045-d96dc75d-487c-4485-a500-c9d1d57f818b.png)


```
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: {
    index: './src/index.js',
    album: './src/album.js'
  },
  output: {
    filename: '[name].bundle.js'
  },
  
  ......
  
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Multi Entry',
      template: './src/index.html',
      filename: 'index.html',
      chunks: ['index']
    }),
    new HtmlWebpackPlugin({
      title: 'Multi Entry',
      template: './src/album.html',
      filename: 'album.html',
      chunks: ['album']
    })
  ]
}

```

### 配置提取公共模块

不同打包入口当中肯定会有公共模块，为了解决此类问题，我们需要提取公共模块

```
  optimization: {
    splitChunks: {
      // 自动提取所有公共模块到单独 bundle
      chunks: 'all'
    }
  },
```

打包后目录

![image](https://user-images.githubusercontent.com/37037802/134324354-0ef60dbb-406b-40e9-b8dd-fec6ad093665.png)

### 按需加载 ---- 动态导入

背景：所有代码最终都被打包到一起会导致bundle体积过大，容易造成流量与带宽的浪费，为解决此类问题，Webpack为我们提供了分包（按需加载）的功能。

Webpack已具备动态导入与自动分包的功能，而此功能无需配置。

![image](https://user-images.githubusercontent.com/37037802/134336528-85e48c90-c223-40eb-b089-a941d85d3e2c.png)

下图为打包后结果

![image](https://user-images.githubusercontent.com/37037802/134478781-c2c185cf-a025-49f1-b2e8-4106ffa8f11c.png)

### Tree Shaking --- "摇掉"代码未引用部分（dead-code）

Tree Shaking并不是指某个配置选项，produciton环境下自动开启（无需配置），但是在非produciton环境可以进行优化

```
module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js'
  },
  optimization: {
    // 模块只导出被使用的成员
    usedExports: true,
    // 尽可能合并每一个模块到一个函数中
    concatenateModules: true,
    // 压缩输出结果
    // minimize: true
  }
}

```

### HMR --- 模块热更新（热替换）

背景：自动刷新会导致页面状态丢失，热替换（HMR）可以很好解决此问题。

HMR概述：应用运行过程中实时替换某个模块而应用运行状态不受影响。

```
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  devServer: {
    hot: true
    // hotOnly: true // 只使用 HMR，不会 fallback 到 live reloading
  },

  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
}

```

PS：Webpack开启HMR后，HMR并不可以开箱即用（需要手动处理模块热替换逻辑）

#### Q1.为什么样式文件的热更新开箱即用？

那是因为样式文件经过style-Loader处理了，故不需要手动处理热更新。

![image](https://user-images.githubusercontent.com/37037802/134803603-09f1f053-2c0f-42ef-b2ae-11df01ba99d1.png)

#### Q2.凭什么样式文件可以自动处理，而脚本文件却需要手动处理？

那是因为样式模块更新后，只需把更新后的CSS即时替换到页面中就能覆盖之前样式，从而实现样式文件的更新。但是脚本（JS）模块并没任何规律的，因为脚本模块可能导出的是对象、字符串、函数......
Webpack面对这些毫无规律的JS模块不知道如何去处理，没办法实现通用的模块替换方案。

#### Q3.使用部分框架的项目没有手动处理，JS照样可以热替换?

因为框架下的开发，每种文件都是有规律的，有了规律就有通用的替换的办法。例如React.js框架下每个文件导出的必须是函数或则类，把导出的函数再执行一遍即可实现热替换。

### HMR API --- 手动处理热更新

```
if (module.hot) {
  let lastEditor = editor
  module.hot.accept('./editor', () => {
    // console.log('editor 模块更新了，需要这里手动处理热替换逻辑')
    // console.log(createEditor)

    const value = lastEditor.innerHTML
    document.body.removeChild(lastEditor)
    const newEditor = createEditor()
    newEditor.innerHTML = value
    document.body.appendChild(newEditor)
    lastEditor = newEditor
  })

  module.hot.accept('./better.png', () => {
    img.src = background
    console.log(background)
  })
}
```


