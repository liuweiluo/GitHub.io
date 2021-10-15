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
    // 1. 通过属性保存选项的数据
    this.$options = options || {}
    this.$data = options.data || {}
    this.$el = typeof options.el === 'string' ? document.querySelector(options.el) : options.el
    // 2. 把data中的成员转换成getter和setter，注入到vue实例中
    this._proxyData(this.$data)
    // 3. 调用observer对象，监听数据的变化
    new Observer(this.$data)
    // 4. 调用compiler对象，解析指令和差值表达式
    new Compiler(this)
  }
  _proxyData (data) {
    // 遍历data中的所有属性
    Object.keys(data).forEach(key => {
      // 把data的属性注入到vue实例中
      Object.defineProperty(this, key, {
        enumerable: true,
        configurable: true,
        get () {
          return data[key]
        },
        set (newValue) {
          if (newValue === data[key]) {
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
class Observer {
  constructor (data) {
    this.walk(data)
  }
  walk (data) {
    // 1. 判断data是否是对象
    if (!data || typeof data !== 'object') {
      return
    }
    // 2. 遍历data对象的所有属性
    Object.keys(data).forEach(key => {
      this.defineReactive(data, key, data[key])
    })
  }
  defineReactive (obj, key, val) {
    let that = this
    // 负责收集依赖，并发送通知
    let dep = new Dep()
    // 如果val是对象，把val内部的属性转换成响应式数据
    this.walk(val)
    Object.defineProperty(obj, key, {
      enumerable: true,
      configurable: true,
      get () {
        // 收集依赖
        Dep.target && dep.addSub(Dep.target)
        return val
      },
      set (newValue) {
        if (newValue === val) {
          return
        }
        val = newValue
        that.walk(newValue)
        // 发送通知
        dep.notify()
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
class Compiler {
  constructor (vm) {
    this.el = vm.$el
    this.vm = vm
    this.compile(this.el)
  }
  // 编译模板，处理文本节点和元素节点
  compile (el) {
    let childNodes = el.childNodes
    Array.from(childNodes).forEach(node => {
      // 处理文本节点
      if (this.isTextNode(node)) {
        this.compileText(node)
      } else if (this.isElementNode(node)) {
        // 处理元素节点
        this.compileElement(node)
      }

      // 判断node节点，是否有子节点，如果有子节点，要递归调用compile
      if (node.childNodes && node.childNodes.length) {
        this.compile(node)
      }
    })
  }
  // 编译元素节点，处理指令
  compileElement (node) {
    // console.log(node.attributes)
    // 遍历所有的属性节点
    Array.from(node.attributes).forEach(attr => {
      // 判断是否是指令
      let attrName = attr.name
      if (this.isDirective(attrName)) {
        // v-text --> text
        attrName = attrName.substr(2)
        let key = attr.value
        this.update(node, key, attrName)
      }
    })
  }

  update (node, key, attrName) {
    let updateFn = this[attrName + 'Updater']
    updateFn && updateFn.call(this, node, this.vm[key], key)
  }

  // 处理 v-text 指令
  textUpdater (node, value, key) {
    node.textContent = value
    new Watcher(this.vm, key, (newValue) => {
      node.textContent = newValue
    })
  }
  // v-model
  modelUpdater (node, value, key) {
    node.value = value
    new Watcher(this.vm, key, (newValue) => {
      node.value = newValue
    })
    // 双向绑定
    node.addEventListener('input', () => {
      this.vm[key] = node.value
    })
  }

  // 编译文本节点，处理差值表达式
  compileText (node) {
    // console.dir(node)
    // {{  msg }}
    let reg = /\{\{(.+?)\}\}/
    let value = node.textContent
    if (reg.test(value)) {
      let key = RegExp.$1.trim()
      node.textContent = value.replace(reg, this.vm[key])

      // 创建watcher对象，当数据改变更新视图
      new Watcher(this.vm, key, (newValue) => {
        node.textContent = newValue
      })
    }
  }
  // 判断元素属性是否是指令
  isDirective (attrName) {
    return attrName.startsWith('v-')
  }
  // 判断节点是否是文本节点
  isTextNode (node) {
    return node.nodeType === 3
  }
  // 判断节点是否是元素节点
  isElementNode (node) {
    return node.nodeType === 1
  }
}
```

### Dep(Dependency)

![image](https://user-images.githubusercontent.com/37037802/137483257-afa70f33-a2bb-47bb-9dbe-cb43c6c3431e.png)

- 功能
 
 收集依赖，添加观察者(watcher)
 
 通知所有观察者

- 结构

  ![image](https://user-images.githubusercontent.com/37037802/137483400-c2fe51ce-c7a8-4976-ace2-fb7de9ec58cf.png)

- 代码
```
class Dep {
  constructor () {
    // 存储所有的观察者
    this.subs = []
  }
  // 添加观察者
  addSub (sub) {
    if (sub && sub.update) {
      this.subs.push(sub)
    }
  }
  // 发送通知
  notify () {
    this.subs.forEach(sub => {
      sub.update()
    })
  }
}
```

### Watcher

![image](https://user-images.githubusercontent.com/37037802/137484074-efa29bf3-f8ec-47f7-a20e-d0b4a5d0afd0.png)

- 功能
 
  当数据变化触发依赖， dep 通知所有的 Watcher 实例更新视图
  
  自身实例化的时候往 dep 对象中添加自己
  
- 结构
  
  ![image](https://user-images.githubusercontent.com/37037802/137484194-0f467dfe-6f0c-4010-bb95-8175b133aa08.png)
  
- 代码

```
class Watcher {
  constructor (vm, key, cb) {
    this.vm = vm
    // data中的属性名称
    this.key = key
    // 回调函数负责更新视图
    this.cb = cb

    // 把watcher对象记录到Dep类的静态属性target
    Dep.target = this
    // 触发get方法，在get方法中会调用addSub
    this.oldValue = vm[key]
    Dep.target = null
  }
  // 当数据发生变化的时候更新视图
  update () {
    let newValue = this.vm[this.key]
    if (this.oldValue === newValue) {
      return
    }
    this.cb(newValue)
  }
}
```

### 总结

- 给属性重新赋值成对象，是否是响应式的？

  是

- 给 Vue 实例新增一个成员是否是响应式的？

  不是
  
- 回顾整体流程

![image](https://user-images.githubusercontent.com/37037802/137484793-119c3d8a-025a-415f-8d59-590821b81ce1.png)

  
 

