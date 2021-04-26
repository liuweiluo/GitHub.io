## Reflect 对象

Reflect 是一个内置的对象，它提供拦截 JavaScript 操作的方法。这些方法与proxy handlers的方法相同。Reflect不是一个函数对象，因此它是不可构造的。

### Reflect成员方法就是Proxy处理对象的默认实现

```markdown
const obj = {
  foo: '123',
  bar: '456'
}

const proxy = new Proxy(obj, {
  get (target, property) {
    console.log('watch logic~')
    
    return Reflect.get(target, property)
  }
})

console.log(proxy.foo)
```

### 统一提供一套用于操作对象的API

```markdown
const obj = {
  name: 'zce',
  age: 18
}

console.log('name' in obj)
console.log(delete obj['age'])
console.log(Object.keys(obj))

console.log(Reflect.has(obj, 'name'))
console.log(Reflect.deleteProperty(obj, 'age'))
console.log(Reflect.ownKeys(obj))
```
