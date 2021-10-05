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


