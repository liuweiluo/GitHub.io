## Vue响应式原理分析

### 数据驱动

数据响应式：数据模型仅仅是普通的 JavaScript 对象，而当我们修改数据时，视图会进行更新，避免了繁 琐的 DOM 操作，提高开发效率 

双向绑定：数据改变，视图改变；视图改变，数据也随之改变，我们可以使用 v-model 在表单元素上创建双向数据绑定 

数据驱动是 Vue 独特的特性之一，开发过程中仅需要关注数据本身，不需要关心数据是如何渲染到视图 

### 数据响应式的核心原理

#### Vue 2.x

前沿：当你把一个普通的 JavaScript 对象传入 Vue 实例作为 data 选项，Vue 将遍历此对象所有的 property，并使用 Object.defineProperty 把这些 property 全部转为 getter/setter。Object.defineProperty 是 ES5 中一个无法 shim 的特性，这也就是 Vue 不支持 IE8 以及更低版本浏览器的原因

- Object.defineProperty
- 浏览器兼容 IE8 以上（不兼容 IE8)

```
    // 模拟 Vue 中的 data 选项
    let data = {
      msg: 'hello'
    }

    // 模拟 Vue 的实例
    let vm = {}

    // 数据劫持：当访问或者设置 vm 中的成员的时候，做一些干预操作
    Object.defineProperty(vm, 'msg', {
      // 可枚举（可遍历）
      enumerable: true,
      // 可配置（可以使用 delete 删除，可以通过 defineProperty 重新定义）
      configurable: true,
      // 当获取值的时候执行
      get () {
        console.log('get: ', data.msg)
        return data.msg
      },
      // 当设置值的时候执行
      set (newValue) {
        console.log('set: ', newValue)
        if (newValue === data.msg) {
          return
        }
        data.msg = newValue
        // 数据更改，更新 DOM 的值
        document.querySelector('#app').textContent = data.msg
      }
    })

    // 测试
    vm.msg = 'Hello World'
    console.log(vm.msg)
```

如果有一个对象中多个属性需要转换 getter/setter 如何处理？ 

```
    // 模拟 Vue 中的 data 选项
    let data = {
      msg: 'hello',
      count: 10
    }

    // 模拟 Vue 的实例
    let vm = {}

    proxyData(data)

    function proxyData(data) {
      // 遍历 data 对象的所有属性
      Object.keys(data).forEach(key => {
        // 把 data 中的属性，转换成 vm 的 setter/setter
        Object.defineProperty(vm, key, {
          enumerable: true,
          configurable: true,
          get () {
            console.log('get: ', key, data[key])
            return data[key]
          },
          set (newValue) {
            console.log('set: ', key, newValue)
            if (newValue === data[key]) {
              return
            }
            data[key] = newValue
            // 数据更改，更新 DOM 的值
            document.querySelector('#app').textContent = data[key]
          }
        })
      })
    }

    // 测试
    vm.msg = 'Hello World'
    console.log(vm.msg)
```

#### Vue 3.x
- Proxy
- 直接监听对象，而非属性
- ES 6中新增，IE 不支持，性能由浏览器优化

```
    // 模拟 Vue 中的 data 选项
    let data = {
      msg: 'hello',
      count: 0
    }

    // 模拟 Vue 实例
    let vm = new Proxy(data, {
      // 执行代理行为的函数
      // 当访问 vm 的成员会执行
      get (target, key) {
        console.log('get, key: ', key, target[key])
        return target[key]
      },
      // 当设置 vm 的成员会执行
      set (target, key, newValue) {
        console.log('set, key: ', key, newValue)
        if (target[key] === newValue) {
          return
        }
        target[key] = newValue
        document.querySelector('#app').textContent = target[key]
      }
    })

    // 测试
    vm.msg = 'Hello World'
    console.log(vm.msg)
```

### 发布/订阅模式

- 发布/订阅模式由三部分组成：订阅者、发布者、信号中心

- 前言：假设你是学生家长，在学校每次考完试后都想要获得孩子的成绩，由于之前没有班级成绩订阅系统，家长只能每天都催促孩子成绩出了没。现在班级有了成绩订阅系统，家长可以到孩子班级订阅成绩（相当于注册事件），一旦老师公布成绩（相当于触发了事件），成绩由班级订阅系统以短形式发送给家长（不需要天都催促孩子），这案例中，家长就是订阅者，老师就是发布者，班级订阅系统就是信号中心。

- 概述：我们假定，存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做"发布/订阅模式"（publish-subscribe pattern）

- Vue自定义事件

https://cn.vuejs.org/v2/guide/migration.html#dispatch-%E5%92%8C-broadcast-%E6%9B%BF%E6%8D%A2

```
let vm = new Vue()
vm.$on('dataChange', () => {
console.log('dataChange')
})
vm.$on('dataChange', () => {
console.log('dataChange1')
})
vm.$emit('dataChange')

```

- 兄弟组件通信过程
```
// eventBus.js
// 事件中心
let eventHub = new Vue()
// ComponentA.vue
// 发布者
addTodo: function () {
// 发布消息(事件)
eventHub.$emit('add-todo', { text: this.newTodoText })
this.newTodoText = ''
}
// ComponentB.vue
// 订阅者
created: function () {
// 订阅消息(事件)
eventHub.$on('add-todo', this.addTodo)
}
```

- 模拟 Vue 自定义事件的实现

```
class EventEmitter {
constructor () {
// { eventType: [ handler1, handler2 ] }
this.subs = {}
}
// 订阅通知
$on (eventType, handler) {
this.subs[eventType] = this.subs[eventType] || []
this.subs[eventType].push(handler)
}
// 发布通知
$emit (eventType) {
if (this.subs[eventType]) {
this.subs[eventType].forEach(handler => {
handler()
})
}
}
}
// 测试
var bus = new EventEmitter()
// 注册事件
bus.$on('click', function () {
console.log('click')
})
bus.$on('click', function () {
console.log('click1')
})
// 触发事件
bus.$emit('click')

```

### 观察者模式

- 观察者模式由2部分组成：观察者(订阅者)-Watcher、目标(发布者)-Dep

- 观察者(订阅者) -- Watcher  --> update()：当事件发生时，具体要做的事情

- 目标(发布者) -- Dep --> 1.subs 数组：存储所有的观察者   2.addSub()：添加观察者    3.notify()：当事件发生，调用所有观察者的 update() 方法

- ps：观察者模式与发布/订阅模式区别：1.观察者模式没有事件中心 2.观察者模式下，发布者需要知道订阅者的存在，而发布/订阅模式下却不需要

```
// 目标(发布者)
// Dependency
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
// 通知所有观察者
notify () {
this.subs.forEach(sub => {
sub.update()
})
}
}
// 观察者(订阅者)
class Watcher {
update () {
console.log('update')
}
}
// 测试
let dep = new Dep()
let watcher = new Watcher()
dep.addSub(watcher)
dep.notify()

```

#### 总结

- 观察者模式是由具体目标调度，比如当事件触发，Dep 就会去调用观察者的方法，所以观察者模式的订阅者与发布者之间是存在依赖的。
- 发布/订阅模式由统一调度中心调用，因此发布者和订阅者不需要知道对方的存在。

![image](https://user-images.githubusercontent.com/37037802/137469212-09cf9311-09b9-4da1-817a-15cf21c0af9a.png)








