## Hash模式和History模式

不管是Hash模式还是History模式，当路径发生变化后，不会向服务器发生请求，而是使用JS监视路径的变化，然后根据不同地址去渲染不同内容，若需使用服务器请求则发送ajax获取。

### Hash模式和History模式区别

#### 表现形式的区别

- Hash模式
https://music.163.com/#/playlist?id=33123315

- History模式
https://music.163.com/playlist/33123315

#### 原理的区别

- Hash模式：基于描点以及onhashchange事件

- History模式：基于HTML5的History API（history.pushState()、history.replaceState()、onpopstate事件）

PS: 调用history.push()时，浏览器地址发生改变同时会向服务器发送请求，而调用history.pushState()浏览器地址发生改变但不用向服务器发送请求。

### History模式的使用

- History 需要服务器的支持

- 单页应用中，服务端不存在 http://www.testurl.com/login 这样的地址会返回找不到该页面

- 在服务端应该除了静态资源外都返回单页应用的 index.html

#### Node.js 环境

使用中间件（connect-history-api-fallback）去处理History模式，客户端发送地址请求后，服务器通过判断前路由是否存在，不存在则返回index.html给浏览器，浏览器接收到index.html就继续判断前端路由地址并加载对应组件
```
const path = require('path')
// 导入处理 history 模式的模块
const history = require('connect-history-api-fallback')
// 导入 express
const express = require('express')

const app = express()
// 注册处理 history 模式的中间件
app.use(history())
// 处理静态资源的中间件，网站根目录 ../web
app.use(express.static(path.join(__dirname, '../web')))

// 开启服务器，端口是 3000
app.listen(3000, () => {
  console.log('服务器开启，端口：3000')
})
```

#### nginx 环境配置

- 从官网下载 nginx 的压缩包
- 把压缩包解压到 c 盘根目录，c:\nginx-1.18.0 文件夹
- 修改 conf\nginx.conf 文件

```
location / {
    root html;
    index index.html index.htm;
    #新添加内容
    #尝试读取$uri(当前请求的路径)，如果读取不到读取$uri/这个文件夹下的首页
    #如果都获取不到返回根目录中的 index.html
    try_files $uri $uri/ /index.html;
  }
```

- 打开命令行，切换到目录 c:\nginx-1.18.0
- nginx 启动、重启和停止

```
# 启动
start nginx
# 重启
nginx -s reload
# 停止
nginx -s stop

```
