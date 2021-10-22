## Vue.js 源码剖析

### Vue首次渲染的过程
- 01.定义Vue构造函数（包括定义实例成员和静态成员）
- 02.新建Vue实例（new Vue()），然后调用this._init()方法初始化实例成员和静态成员，最后调用vm.$mount() 方法挂载
- 03.调用vm.$mount() 方法有2种情况:
  - 完整版: 先调用 src\platforms\web\entry-runtime-with-compiler.js 里的$mount方法，该方法里先判断是否存在render选项，若存在直接使用render选项对应的render函数，否则先把template模板编译    成render函数，如果没有template模板，获取el的 outerHTML 作为template模板，然后把render函数记录在选项里（options.render=render），最后调用 src\platforms\web\runtime\index.js 中的        vm.$mount()方法（这里会重新获取el,因为运行时版需要el）。
  - 运行时版：直接调用 src\platforms\web\runtime\index.js 中的vm.$mount()方法（这里会获取el）。
- 04.调用mountComponent()方法: 调用 src\platforms\web\runtime\index.js 中的vm.$mount()方法会调用mountComponent()方法，在该方法里首先触发beforeMount钩子函数，然后定义updateComponent方法（注意：这里只是定义并没有调用，涉及2个函数待补充）
- 05.创建Watcher实例: 创建Watcher实例时会把updateComponent方法以参数形式传入，创建完watcher实例后会调用一次get()方法，get()方法里会调用传入的updateComponent方法
- 06.触发Mounted钩子函数
- 07.返回实例

###  Vue 响应式原理

- 01.调用initState()方法: 当新建Vue实例（new Vue()），会调用this._init()方法初始化实例成员，此时会调用initState()方法去始化 vm 的 _props/methods/_data/computed/watch...
- 02.调用initData()方法：调用initState()方法后继续调用initData()方法把data属性注入vm实例中。
- 03.调用observe()方法：在调用initData()方法里会调用observe()方法把data对象转换响应式对象。
- 04.在observe()方法中首先判断value是否是对象，如果不是对象直接返回，继续判断value对象是否存在_ob_属性，如果有直接返回该属性，如果没有创建observer对象并返回该对象实例。
- 05.创建observer对象中给value对象定义不可枚举的__ob__属性，该属性记录当前的observer对象，然后进行数组的响应式处理（对数组中的对象和数组部分特殊的方法进行响应式处理）和对象进行响应式处理（调用walk方法）
- 06.调用walk方法：walk方法里会遍历对象中每一个属性调用defineReactive方法设置为响应式数据（转换成 setter/getter）
- 07.调用defineReactive方法：在该方法里会为每个属性创建dep对象，如果当前属性的值是对象，会调用observe()方法转为响应式对象，然后定义getter和setter，getter方法作用是收集依赖和返回属性值，setter方法作用是保存新值，如果新值是对象则会调用observe()方法，同时因为数据发送变化，会派发更新（即发送通知，调用dep.notyfy方法）
- 08.收集依赖：在watcher对象的get方法中调用pushTarget,记录Dep.target属性，访问data中的成员的时候收集依赖，defineReactive的getter中收集依赖，把属性对应的 watcher 对象添加到dep的subs数组中，给childOb收集依赖，目的是子对象添加和删除成员时发送通知。
- 09.触发wacher对象的update()方法：派发更新(即发送通知，调用dep.notyfy方法)后，就会调用wacher对象的update()方法。

### 虚拟 DOM 中 Key 的作用和好处

- 作用：key是Vnode的唯一标记，在比较新旧Vnode（diff操作）中它能够跟踪每个Vnode，通过key可以精准判断两个节点是否是同一个，在某种情况下（注意这里是不是绝对的）可以避免频繁更新不同元素，可以减少dom的操作，减少diff和渲染所需要的时间，提升了性能
- ps: 不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法，某种情况下会减少dom操作

### diff算法过程

#### 打补丁过程从patch方法开始

```
  // 函数柯里化，让一个函数返回一个函数
  // createPatchFunction({ nodeOps, modules }) 传入平台相关的两个参数

  // core中的createPatchFunction (backend), const { modules, nodeOps } = backend
  // core中方法和平台无关，传入两个参数后，可以在上面的函数中使用这两个参数
  return function patch (oldVnode, vnode, hydrating, removeOnly) {
    // 新的 VNode 不存在
    if (isUndef(vnode)) {
      // 老的 VNode 存在，执行 Destroy 钩子函数
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
      return
    }

    let isInitialPatch = false
    const insertedVnodeQueue = []

    // 老的 VNode 不存在
    if (isUndef(oldVnode)) {
      // empty mount (likely as component), create new root element
      isInitialPatch = true
      // 创建新的 VNode
      createElm(vnode, insertedVnodeQueue)
    } else {
      // 新的和老的 VNode 都存在，更新
      const isRealElement = isDef(oldVnode.nodeType)
      // 判断参数1是否是真实 DOM，不是真实 DOM
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // 更新操作，diff 算法
        // patch existing root node
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
      } else {
        // 第一个参数是真实 DOM，创建 VNode
        // 初始化
        if (isRealElement) {
          ......
          // either not server-rendered, or hydration failed.
          // create an empty node and replace it
          oldVnode = emptyNodeAt(oldVnode)
        }

        // replacing existing element
        const oldElm = oldVnode.elm
        const parentElm = nodeOps.parentNode(oldElm)

        // create new node
        // 创建 DOM 节点
        createElm(
          vnode,
          insertedVnodeQueue,
          // extremely rare edge case: do not insert if old element is in a
          // leaving transition. Only happens when combining transition +
          // keep-alive + HOCs. (#4590)
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        )

        // update parent placeholder node element, recursively
        if (isDef(vnode.parent)) {
        ......
        }

        // destroy old node
        if (isDef(parentElm)) {
          removeVnodes([oldVnode], 0, 0)
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode)
        }
      }
    }

    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }
```
> 从上面可以看出，patch函数就是对比新老节点
> 
> 01 如果老节点不存在，直接创建新节点，但是创建后的新节点不会挂载到视图上（dom树），只会存到内存中。
> 
> 02 如果老节点存在且与新节点是同一节点，则执行patchVnode进行子节点比较
> 
> 03 如果老节点与新节点不是同一节点，新节点直接替换老节点
> 
> 04 最后返回新节点的dom元素并记录到Vue实例的el属性

#### 执行patchVnode方法，说明新老节点为同一节点

```
  function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    if (oldVnode === vnode) {
      return
    }

    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // clone reused vnode
      vnode = ownerArray[index] = cloneVNode(vnode)
    }

    const elm = vnode.elm = oldVnode.elm
    
    .......
    
    let i
    const data = vnode.data
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      i(oldVnode, vnode)
    }

    const oldCh = oldVnode.children
    const ch = vnode.children
    if (isDef(data) && isPatchable(vnode)) {
      // 调用 cbs 中的钩子函数，操作节点的属性/样式/事件....
      for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
      // 用户的自定义钩子
      if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
    }

    // 新节点没有文本
    if (isUndef(vnode.text)) {
      // 老节点和老节点都有有子节点
      // 对子节点进行 diff 操作，调用 updateChildren
      if (isDef(oldCh) && isDef(ch)) {
        if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
      } else if (isDef(ch)) {
        // 新的有子节点，老的没有子节点
        if (process.env.NODE_ENV !== 'production') {
          checkDuplicateKeys(ch)
        }
        // 先清空老节点 DOM 的文本内容，然后为当前 DOM 节点加入子节点
        if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
      } else if (isDef(oldCh)) {
        // 老节点有子节点，新的没有子节点
        // 删除老节点中的子节点
        removeVnodes(oldCh, 0, oldCh.length - 1)
      } else if (isDef(oldVnode.text)) {
        // 老节点有文本，新节点没有文本
        // 清空老节点的文本内容
        nodeOps.setTextContent(elm, '')
      }
    } else if (oldVnode.text !== vnode.text) {
      // 新老节点都有文本节点
      // 修改文本
      nodeOps.setTextContent(elm, vnode.text)
    }
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
    }
  }
```
> 01 新老节点一样，直接返回
> 
> 02 如果新老节点都只是文本节点且文本不相同，则直接用新节点的文本替换老节点的文本
> 
> 03 如果新节点不是文本节点，则分以下几点情况判断
> 
>> 03-1 如果新老节点都有子节点，则调用 updateChildren方法
>> 
>> 03-2 如果老节点没有子节点，新节点有子节点则把新节点的子节点转化为dom元素并添加在dom树上。
>> 
>> 03-3 如果老节点有子节点，新节点没有：删除老节点的子节点

#### 执行updateChildren方法，继续比较新老节点子节点
```
  // 更新新旧节点的子节点
  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx] 
    let newEndIdx = newCh.length - 1 
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm
    
    ......

    // diff 算法
    // 当新节点和旧节点都没有遍历完成
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        // oldStartVnode 和 newStartVnode 相同(sameVnode)
        // 直接将该 VNode 节点进行 patchVnode
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        // 获取下一组开始节点
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        // 直接将该 VNode 节点进行 patchVnode
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        // 获取下一组结束节点
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        // oldStartVnode 和 newEndVnode 相同(sameVnode)
        // 进行 patchVnode，把 oldStartVnode 移动到最后
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        // 移动游标，获取下一组节点
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        // oldEndVnode 和 newStartVnode 相同(sameVnode)
        // 进行 patchVnode，把 oldEndVnode 移动到最前面
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        // 以上四种情况都不满足
        // newStartNode 依次和旧的节点比较
        
        // 从新的节点开头获取一个，去老节点中查找相同节点
        // 先找新开始节点的key和老节点相同的索引，如果没找到再通过sameVnode找
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        // 如果没有找到
        if (isUndef(idxInOld)) { // New element
          // 创建节点并插入到最前面
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          // 获取要移动的老节点
          vnodeToMove = oldCh[idxInOld]
          // 如果使用 newStartNode 找到相同的老节点
          if (sameVnode(vnodeToMove, newStartVnode)) {
            // 执行 patchVnode，并且将找到的旧节点移动到最前面
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // 如果key相同，但是是不同的元素，创建新元素
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
    // 当结束时 oldStartIdx > oldEndIdx，旧节点遍历完，但是新节点还没有
    if (oldStartIdx > oldEndIdx) {
      // 说明新节点比老节点多，把剩下的新节点插入到老的节点后面
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      // 当结束时 newStartIdx > newEndIdx，新节点遍历完，但是旧节点还没有
      removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
  }
```
待补充
