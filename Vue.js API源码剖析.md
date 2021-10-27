## Vue.js API源码剖析

### vm.$set

- 功能

  向响应式对象中添加一个属性，并确保这个新属性同样是响应式的，且触发视图更新。它必须用于向响应式对象上添加新属性，因为 Vue 无法探测普通的新增属性 (比如this.myObject.newProperty = 'hi')
  
> 注意：对象不能是 Vue 实例，或者 Vue 实例的根数据对象。

- 示例
 ```
  vm.$set(obj, 'foo', 'test')
 ```
 
- 源码

observer/index.js

```
/**
 * Set a property on an object. Adds the new property and
 * triggers change notification if the property doesn't
 * already exist.
 */
export function set (target: Array<any> | Object, key: any, val: any): any {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot set reactive property on undefined, null, or primitive value: ${(target: any)}`)
  }
  // 判断 target 是否是对象，key 是否是合法的索引
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key)
    // 通过 splice 对key位置的元素进行替换
    // splice 在 array.js 进行了响应化的处理
    target.splice(key, 1, val)
    return val
  }
  // 如果 key 在对象中已经存在直接赋值
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  // 获取 target 中的 observer 对象
  const ob = (target: any).__ob__
  // 如果 target 是 vue 实例或者 $data 直接返回
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - declare it upfront in the data option.'
    )
    return val
  }
  // 如果 ob 不存在，target 不是响应式对象直接赋值
  if (!ob) {
    target[key] = val
    return val
  }
  // 把 key 设置为响应式属性
  defineReactive(ob.value, key, val)
  // 发送通知
  ob.dep.notify()
  return val
}
```

### vm.$delete

- 功能

  删除对象的属性。如果对象是响应式的，确保删除能触发更新视图。这个方法主要用于避开 Vue不能检测到属性被删除的限制，但是你应该很少会使用它。
  
> 注意：目标对象不能是一个 Vue 实例或 Vue 实例的根数据对象。

- 示例

 ```
  vm.$delete(vm.obj, 'msg')
 ```
 
 - 源码

src\core\observer\index.js

```
/**
 * Delete a property and trigger change if necessary.
 */
export function del (target: Array<any> | Object, key: any) {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(`Cannot delete reactive property on undefined, null, or primitive value: ${(target: any)}`)
  }
  // 判断是否是数组，以及 key 是否合法
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // 如果是数组通过 splice 删除
    // splice 做过响应式处理
    target.splice(key, 1)
    return
  }
  // 获取 target 的 ob 对象
  const ob = (target: any).__ob__
  // target 如果是 Vue 实例或者 $data 对象，直接返回
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid deleting properties on a Vue instance or its root $data ' +
      '- just set it to null.'
    )
    return
  }
  // 如果 target 对象没有 key 属性直接返回
  if (!hasOwn(target, key)) {
    return
  }
  // 删除属性
  delete target[key]
  if (!ob) {
    return
  }
  // 通过 ob 发送通知
  ob.dep.notify()
}
```

### 具备懒加载和缓存功能的watcher ---- 计算watcher（computed属性）

PS：我们经常被问题到：Vue中computed属性(计算watcher)和watch属性(用户watcher)的区别

首先在组件初始化的时候，会进入到初始化 computed 的函数

```
if (opts.computed) { initComputed(vm, opts.computed); }
```

进入 initComputed 看看

```
var watchers = vm._computedWatchers = Object.create(null);
 
// 依次为每个 computed 属性定义
for (const key in computed) {
  const userDef = computed[key]
  watchers[key] = new Watcher(
      vm, // 实例
      getter, // 用户传入的求值函数 sum
      noop, // 回调函数 可以先忽视
      { lazy: true } // 声明 lazy 属性 标记 computed watcher
  )
 
  // 用户在调用 this.sum 的时候，会发生的事情
  defineComputed(vm, key, userDef)
}
```

首先定义了一个空的对象，用来存放所有计算属性相关的watcher，后文我们会把它叫做计算watcher。然后循环为每个computed属性生成了一个计算watcher。它的形态保留关键属性简化后是这样的：

```
{
    deps: [],
    dirty: true,
    getter: ƒ sum(),
    lazy: true,
    value: undefined
}
```

可以看到它的 value 刚开始是 undefined，lazy 是 true，说明它的值是惰性计算的，只有到真正在模板里去读取它的值后才会计算。这个 dirty 属性其实是缓存的关键，先记住它。

接下来看看比较关键的 defineComputed，它决定了用户在读取 this.sum 这个计算属性的值后会发生什么，继续简化，排除掉一些不影响流程的逻辑。

```
Object.defineProperty(target, key, { 
    get() {
        // 从刚刚说过的组件实例上拿到 computed watcher
        const watcher = this._computedWatchers && this._computedWatchers[key]
        if (watcher) {
          // ✨ 注意！这里只有dirty了才会重新求值
          if (watcher.dirty) {
            // 这里会求值 调用 get
            watcher.evaluate()
          }
          // ✨ 这里也是个关键 等会细讲
          if (Dep.target) {
            watcher.depend()
          }
          // 最后返回计算出来的值
          return watcher.value
        }
    }
})
```

这个函数需要仔细看看，它做了好几件事，我们以初始化的流程来讲解它：

首先 dirty 这个概念代表脏数据，说明这个数据需要重新调用用户传入的 sum 函数来求值了。我们暂且不管更新时候的逻辑，第一次在模板中读取到  {{sum}} 的时候它一定是 true，所以初始化就会经历一次求值。

```
evaluate () {
  // 调用 get 函数求值
  this.value = this.get()
  // 把 dirty 标记为 false
  this.dirty = false
}
```

这个函数其实很清晰，它先求值，然后把 dirty 置为 false。

再回头看看我们刚刚那段 Object.defineProperty 的逻辑，

下次没有特殊情况再读取到 sum 的时候，发现 dirty是false了，是不是直接就返回 watcher.value 这个值就可以了，这其实就是计算属性缓存的概念。


