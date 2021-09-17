
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

--- css-Loader: 把CSS代码转换为Javascript代码

- 文件操作类Loader：此类Loader会把资源模块拷贝至输出目录，同时把这个资源的访问路径向外导出

--- file

![image](https://user-images.githubusercontent.com/37037802/133782897-024a8fbe-44f5-4684-9ae4-2ecdbcb7de15.png)
