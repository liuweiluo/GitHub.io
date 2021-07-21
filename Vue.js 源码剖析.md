## Vue.js 源码剖析

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
