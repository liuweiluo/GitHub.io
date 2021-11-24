## 开发一个 markdown-loader

- 需求：在代码当中使用加载器导入markdown文件
- 分析：我们都知道markdown文件转为html代码过后再呈现至页面上，所以希望通过loader可以把markdown文件内容转为html字符串即可

### 步骤

#### 1.新建文件：在根目录下新建markdown-loader.js文件，完成后可以把这模块发布npm作为一个独立的模块去使用

![image](https://user-images.githubusercontent.com/37037802/143236644-b1a2ac74-43bc-4144-a858-091ce3f4e88d.png)

#### 2.导出函数：每个webpack的loader都需要导出一个函数，而这函数就是对资源的处理过程,它的输入是需要加载的资源文件的内容（通过参数source接收输入），输出就是加工后的内容（通过retrun返回处理结果）

/markdown-loader.js
```
module.exports = source => {
  console.log(source)
  return 'hello ~'
}
```

#### 3.配置测试：在webpack.config.js配置文件里添加一个加载器的规则配置

```
const path = require('path')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    publicPath: 'dist/'
  },
  module: {
    rules: [
      {
        test: /.md$/,
        use: './markdown-loader' //这里use属性不仅可以使用模块名称，也可以使用模块文件路径（这其实与commonjs的require函数一样的）
      }
    ]
  }
}

```

#### 4.测试结果：可能出现解析错误

![image](https://user-images.githubusercontent.com/37037802/143242380-609cfe1d-9e01-4aaf-b577-0cad843efb3d.png)

报错大概意思是：我还需要额外的加载器来处理加载结果，为什么会出现这样的报错？

- 其实webpack加载资源的过程有点像工作管道

![image](https://user-images.githubusercontent.com/37037802/143243431-055734f3-06cc-4878-8d0d-63b9adf39fa8.png)

- 你可以在这个过程依次使用多个loader

![image](https://user-images.githubusercontent.com/37037802/143243554-e75a458d-d387-45b0-9f1a-bcfb1e64abcd.png)

- 但是最后处理结果必须是一段js代码

![image](https://user-images.githubusercontent.com/37037802/143243743-d18b8a6f-bfe8-4bb9-b780-c39e080b795e.png)


##### 所以这里报错的原因就是因为返回的内容是'hello ~'并不是标准的js代码才导致报错。

#### 5.解决办法：

- 解决办法1：返回标准的js代码

![image](https://user-images.githubusercontent.com/37037802/143244625-271ceb0b-d8f9-498f-bccd-ed5cfcb4f1a0.png)

```
module.exports = source => {
  console.log(source)
  return 'console.log("hello ~")'
}
```

打包结果：

![image](https://user-images.githubusercontent.com/37037802/143245261-bc13d283-f087-4cf7-b30b-cd2ed4fd3222.png)

通过打包结果可以看出：其实就是把加载后返回的结果（js代码）直接拼接到模块当中了，同时也解释了loader管道最后必须要返回js代码的原因

安装导入marked依赖去处理
```
const marked = require('marked')

module.exports = source => {
  const html = marked(source)
  // return `module.exports = "${html}"`  如果使用这方式会把html存在的换行符和引号拼接会造成语法上的错误
  return `export default ${JSON.stringify(html)}` //这里使用小技巧先把字符串转为JSON格式字符串，此时存在的换行符和引号都会被转义后再参与拼接就没问题了
}
```

- 解决办法2：再找一个合适的加载器接着去处理这里返回的结果

![image](https://user-images.githubusercontent.com/37037802/143244672-974a36d8-61a1-4637-9820-6c4c770499a5.png)

```
const marked = require('marked')

module.exports = source => {
  const html = marked(source)
  // 返回 html 字符串交给下一个 loader 处理 
  return html
}

```

这里可以使用html-loader去继续处理

```
module.exports = {
...
  module: {
    rules: [
      {
        test: /.md$/,
        use: [
          'html-loader',
          './markdown-loader'
        ]
      }
    ]
  }
}
```

总结：从以上步骤看出Loader负责资源文件从输入到输出的转换，对于一个资源额可以依次使用多个Loader







