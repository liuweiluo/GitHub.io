## Proxy

### Proxy 对象

Proxy 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。

PS: 在vue3.0之前的版本，是使用Object.defineProperty方法实现数据响应，从而完成双向数据绑定。但在vue3.0之后就使用Proxy来代替它，相比于Object.defineProperty方法，Proxy功能更为强大，使用更为方便。

```markdown
 const person = {
   name: 'zce',
   age: 20
 }
 
//构造函数的第一个参数为目标对象，第二个参数为代理处理对象 
const personProxy = new Proxy(person, {
  // 监视属性读取
  get (target, property) {
    return property in target ? target[property] : 'default'
    // console.log(target, property)
    // return 100
  },
  // 监视属性设置
  set (target, property, value) {
    if (property === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError(`${value} is not an int`)
      }
    }

    target[property] = value
    // console.log(target, property, value)
  }
})

personProxy.age = 100

personProxy.gender = true

console.log(personProxy.name)
console.log(personProxy.xxx)
```

### Proxy 对比 Object.defineProperty()

#### 优势1：Proxy 可以监视读写以外的操作
```markdown
const person = {
  name: 'zce',
  age: 20
}

const personProxy = new Proxy(person, {
  deleteProperty (target, property) {
    console.log('delete', property)
    delete target[property]
  }
})

delete personProxy.age
console.log(person)
```

#### 优势2：Proxy 可以很方便的监视数组操作
```markdown
const list = []

const listProxy = new Proxy(list, {
  set (target, property, value) {
    console.log('set', property, value)
    target[property] = value
    return true // 表示设置成功
  }
})

listProxy.push(100)
listProxy.push(100)
```

#### 优势3：Proxy 不需要侵入对象（Proxy直接代理对象,defineProperty只能代理对象的属性）
```markdown
const person = {}

Object.defineProperty(person, 'name', {
  get () {
    console.log('name 被访问')
    return person._name
  },
  set (value) {
    console.log('name 被设置')
    person._name = value
  }
})
Object.defineProperty(person, 'age', {
  get () {
    console.log('age 被访问')
    return person._age
  },
  set (value) {
    console.log('age 被设置')
    person._age = value
  }
})

person.name = 'jack'

console.log(person.name)
```
