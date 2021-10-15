## Vue响应式原理模拟

### 整体分析

模拟最基本Vue结构需要以下5个Class

![image](https://user-images.githubusercontent.com/37037802/137479913-4193cd2f-4983-4290-80c7-b6e79ab68751.png)

- Vue

  把 data 中的成员注入到 Vue 实例，并且把 data 中的成员转成 getter/setter

- Observer

  能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知 Dep
  
- Compiler
  
  解析每个元素中的指令/插值表达式，并替换成相应的数据
  
- Dep

  添加观察者(watcher)，当数据变化通知所有观察者
  
- Watcher

  数据变化更新视图
   
### Vue
- 功能
  
  负责接收初始化的参数(选项
  
  负责把 data 中的属性注入到 Vue 实例，转换成 getter/setter
  
  负责调用 observer 监听 data 中所有属性的变化
  
  负责调用 compiler 解析指令/插值表达式
  
- 结构
  
  ![image](https://user-images.githubusercontent.com/37037802/137481195-aaa85749-f331-494f-95e7-f75399d2b8ce.png)
  
- 代码
```
class Vue {
constructor (options) {
// 1. 保存选项的数据
this.$options = options || {}
this.$data = options.data || {}
const el = options.el
this.$el = typeof options.el === 'string' ? document.querySelector(el)
: el
// 2. 负责把 data 注入到 Vue 实例
this._proxyData(this.$data)
// 3. 负责调用 Observer 实现数据劫持
// 4. 负责调用 Compiler 解析指令/插值表达式等
}
_proxyData (data) {
// 遍历 data 的所有属性
Object.keys(data).forEach(key => {
Object.defineProperty(this, key, {
get () {
return data[key]
},
set (newValue) {
if (data[key] === newValue) {
return
}
data[key] = newValue
}
})
})
}
}

```

### Observer
- 功能

  负责把 data 选项中的属性转换成响应式数据
  
  data 中的某个属性也是对象，把该属性转换成响应式数据
  
  数据变化发送通知
  
- 结构

 ![image](https://user-images.githubusercontent.com/37037802/137481629-9d70768a-8d9a-4e37-8c59-c9d16d88e6aa.png)

- 代码

```
// 负责数据劫持
// 把 $data 中的成员转换成 getter/setter
class Observer {
constructor (data) {
this.walk(data)
}
// 1. 判断数据是否是对象，如果不是对象返回
// 2. 如果是对象，遍历对象的所有属性，设置为 getter/setter
walk (data) {
if (!data || typeof data !== 'object') {
return
}
// 遍历 data 的所有成员
Object.keys(data).forEach(key => {
this.defineReactive(data, key, data[key])
})
}
// 定义响应式成员
defineReactive (data, key, val) {
const that = this
// 如果 val 是对象，继续设置它下面的成员为响应式数据
this.walk(val)
Object.defineProperty(data, key, {
configurable: true,
enumerable: true,
get () {
return val
},
set (newValue) {
if (newValue === val) {
return
}
// 如果 newValue 是对象，设置 newValue 的成员为响应式
that.walk(newValue)
val = newValue
}
})
}
}

```

### Compiler
- 功能

  负责编译模板，解析指令/插值表达式
  
  负责页面的首次渲染
  
  当数据变化后重新渲染视图
  
- 结构

   ![image](https://user-images.githubusercontent.com/37037802/137482276-965c30ad-4338-4cf8-a918-8a2eca1df663.png)
   
- 代码

```
// 负责解析指令/插值表达式
class Compiler {
constructor (vm) {
this.vm = vm
this.el = vm.$el
// 编译模板
this.compile(this.el)
}
// 编译模板
// 处理文本节点和元素节点
compile (el) {
const nodes = el.childNodes
Array.from(nodes).forEach(node => {
// 判断是文本节点还是元素节点
if (this.isTextNode(node)) {
this.compileText(node)
} else if (this.isElementNode(node)) {
this.compileElement(node)
}
if (node.childNodes && node.childNodes.length) {
// 如果当前节点中还有子节点，递归编译
this.compile(node)
}
})
}
// 判断是否是文本节点
isTextNode (node) {
return node.nodeType === 3
}
// 判断是否是属性节点
isElementNode (node) {
return node.nodeType === 1
}
// 判断是否是以 v- 开头的指令
isDirective (attrName) {
return attrName.startsWith('v-')
}
// 编译文本节点
compileText (node) {
}
// 编译属性节点
compileElement (node) {
}
}
```

compileText()
- 负责编译插值表达式

```
// 编译文本节点
compileText (node) {
const reg = /\{\{(.+)\}\}/
// 获取文本节点的内容
const value = node.textContent
if (reg.test(value)) {
// 插值表达式中的值就是我们要的属性名称
const key = RegExp.$1.trim()
// 把插值表达式替换成具体的值
node.textContent = value.replace(reg, this.vm[key])
}
}

```

compileElement()
- 负责编译元素的指令
- 处理 v-text 的首次渲染
- 处理 v-model 的首次渲染

```
// 编译属性节点
compileElement (node) {
// 遍历元素节点中的所有属性，找到指令
Array.from(node.attributes).forEach(attr => {
// 获取元素属性的名称
let attrName = attr.name
// 判断当前的属性名称是否是指令
if (this.isDirective(attrName)) {
// attrName 的形式 v-text v-model
// 截取属性的名称，获取 text model
attrName = attrName.substr(2)
// 获取属性的名称，属性的名称就是我们数据对象的属性 v-text="name"，获取的是
name
const key = attr.value
// 处理不同的指令
this.update(node, key, attrName)
}
})
}
// 负责更新 DOM
// 创建 Watcher
update (node, key, dir) {
// node 节点，key 数据的属性名称，dir 指令的后半部分
const updaterFn = this[dir + 'Updater']
updaterFn && updaterFn(node, this.vm[key])
}
// v-text 指令的更新方法
textUpdater (node, value) {
node.textContent = value
}
// v-model 指令的更新方法
modelUpdater (node, value) {
node.value = value
}

```


