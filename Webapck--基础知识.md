
## Webapck--相关配置

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

#### Webpack与 ES 2015
- 因为模块打包需要，webpack只对import和export进行ES5的转换，其他ES6语法webpack并不能进行转换，若转换需要用到其他加载器，例如bable-loader

####  Webpack 加载资源的方式
- 遵循ES Modules标准的import声明
- 遵循CommonJs标准的require函数
- 遵循AMD标准的define函数和require函数
- Loader加载的非JavaScript也会触发资源加载：例如在安装了css-loader的环境下，样式代码中的@import指令和url函数。在安装了html-loader的环境下，HTML代码中图片标签的src属性

#### webpack核心工作原理

![image](https://user-images.githubusercontent.com/37037802/133885949-9978a9d3-dce1-443e-a387-77646af993b3.png)

以普通的前端项目为例，在项目当中会存在各种各样的资源文件（例如:.js、.html、.css、.png、.scss.....），webpack会根据我们配置文件中的入口配置，找到对应的入口文件（一个或多个js文件）作为入口，然后会根据代码中出现import或require的语句去加载解析该文件的的所有依赖模块，然后解析的依赖模块又会继续解析它对应的依赖模块，一直递归至结束后就会形成项目中所用的资源文件之间的依赖关系（依赖树）如下图

![image](https://user-images.githubusercontent.com/37037802/133887239-b2abe3b0-9295-488a-9901-377282f5e78e.png)

有了这个依赖关系树过后， Webpack会遍历（递归）这个依赖树，找到每个节点对应的资源文件，然后根据配置选项中的Loader配置，交给对应的Loader去加载这个模块，最后将加载的结果放入 bundle.js（打包结果）中，从而实现整个项目的打包

![image](https://user-images.githubusercontent.com/37037802/133887777-0e47273b-b4ab-474a-99aa-b8b87e0ee874.png)

<strong>整个打包过程中，Loader机制起了很重要的作用，因为如果没有Loader的话，Webpack就无法实现各种各样类型的资源文件加载</strong>，那Webpack也就只能算是一个用来合并JS模块代码的工具了。
