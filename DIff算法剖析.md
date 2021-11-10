## DIff算法剖析

### 相关知识点

- 虚拟DOM(Virtual DOM)
  - 什么是虚拟dom
  - 为什么要使用虚拟dom
  - 虚拟DOM库
- DIFF算法
  - init函数
  - h函数
  - patch函数
  - patchVnode函数
  - updateChildren函数 

### 虚拟DOM(Virtual DOM)

#### 什么是虚拟DOM

一句话总结虚拟DOM就是一个用来描述真实DOM的javaScript对象

这样说可能不够形象,那我们来举个例:分别用代码来描述真实DOM以及虚拟DOM

- 真实DOM
```
<ul class="list">
    <li>a</li>
    <li>b</li>
    <li>c</li>
</ul>

```

- 对应的虚拟DOM
```
let vnode = h('ul.list', [
  h('li','a'),
  h('li','b'),
  h('li','c'),
])

console.log(vnode)
```

控制台打印出来的Vnode:

![image](https://user-images.githubusercontent.com/37037802/140739037-fce67aa7-1f8b-4e11-b830-5aa8c27b16cb.png)

h函数生成虚拟DOM(Vnode)源码:

```
export interface VNodeData {
    props?: Props
    attrs?: Attrs
    class?: Classes
    style?: VNodeStyle
    dataset?: Dataset
    on?: On
    hero?: Hero
    attachData?: AttachData
    hook?: Hooks
    key?: Key
    ns?: string // for SVGs
    fn?: () => VNode // for thunks
    args?: any[] // for thunks
    [key: string]: any // for any other 3rd party module
}

export type Key = string | number

const interface VNode = {
    sel: string | undefined, // 选择器
    data: VNodeData | undefined, // VNodeData上面定义的VNodeData
    children: Array<VNode | string> | undefined, //子节点,与text互斥
    text: string | undefined, // 标签中间的文本内容
    elm: Node | undefined, // 转换而成的真实DOM
    key: Key | undefined // 字符串或者数字
}
```

补充:上面的h函数大家可能有点熟悉的感觉但是一时间也没想起来,没关系我来帮大伙回忆; 开发中常见的现实场景,render函数渲染案例：
```
// 案例1 vue项目中的main.js的创建vue实例
new Vue({
  router,
  store,
  render: h => h(App)
}).$mount("#app");

//案例2 列表中使用render渲染
columns: [
    {
        title: "操作",
        key: "action",
        width: 150,
        render: (h, params) => {
            return h('div', [
                h('Button', {
                    props: {
                        size: 'small'
                    },
                    style: {
                        marginRight: '5px',
                        marginBottom: '5px',
                    },
                    on: {
                        click: () => {
                            this.toEdit(params.row.uuid);
                        }
                    }
                }, '编辑')
            ]);
        }
    }
]

```

### 为什么要使用虚拟DOM

- MVVM框架解决视图和状态同步问题
- 模板引擎可以简化视图操作,没办法跟踪状态
- 虚拟DOM跟踪状态变化
- 参考github上virtual-dom的动机描述
  - 虚拟DOM可以维护程序的状态,跟踪上一次的状态
  - 通过比较前后两次状态差异更新真实DOM
- 跨平台使用
  - 浏览器平台渲染DOM
  - 服务端渲染SSR(Nuxt.js/Next.js),前端是vue向,后者是react向
  - 原生应用(Weex/React Native)
  - 小程序(mpvue/uni-app)等
- 真实DOM的属性很多，创建DOM节点开销很大
- 虚拟DOM只是普通JavaScript对象，描述属性并不需要很多，创建开销很小
- 复杂视图情况下提升渲染性能（ps：复杂视图情况操作dom性能消耗大，减少操作dom的范围可以提升性能）

#### 使用了虚拟DOM就一定会比直接渲染真实DOM快吗？答案当然是否定的，且听我说：

举例:当一个节点变更时DOMA->DOMB

![image](https://user-images.githubusercontent.com/37037802/140743873-0ee971c7-9cfd-423f-9311-b19f41524297.png)

上述情况:
示例1是创建一个DOMB然后替换掉DOMA；
示例2去创建虚拟DOM+DIFF算法比对发现DOMB跟DOMA不是相同的节点,最后还是创建一个DOMB然后替换掉DOMA；
可以明显看出1是更快的,同样的结果,2还要去创建虚拟DOM+DIFF算啊对比；
所以说使用虚拟DOM比直接操作真实DOM就一定要快这个说法是错误的,不严谨的

#### 注意：对虚拟节点的DOM操作，并不会触发重绘和回流，把处理后的虚拟节点映射到真实DOM上，只触发一次重绘和回流。

#### 虚拟DOM库 ----- Snabbdom

- Vue.js2.x内部使用的虚拟DOM就是改造的Snabbdom
- 大约200SLOC(single line of code)
- 通过模块可扩展
- 源码使用TypeScript开发
- 最快的Virtual DOM之一

### Diff算法

在看完上述的文章之后相信大家已经对Diff算法有一个初步的概念,没错,Diff算法其实就是找出两者之间的差异;

> diff 算法首先要明确一个概念就是 Diff 的对象是虚拟DOM（virtual dom），更新真实 DOM 是 Diff 算法的结果。

#### snabbdom的核心
- init()设置模块.创建patch()函数
- 使用h()函数创建JavaScript对象(Vnode)描述真实DOM
- patch()比较新旧两个Vnode
- 把变化的内容更新到真实DOM树

#### init函数

init函数时设置模块,然后创建patch()函数,我们先通过场景案例来有一个直观的体现:

```
import {init} from 'snabbdom/build/package/init.js'
import {h} from 'snabbdom/build/package/h.js'

// 1.导入模块
import {styleModule} from "snabbdom/build/package/modules/style";
import {eventListenersModule} from "snabbdom/build/package/modules/eventListeners";

// 2.注册模块
const patch = init([
  styleModule,
  eventListenersModule
])

// 3.使用h()函数的第二个参数传入模块中使用的数据(对象)
let vnode = h('div', [
  h('h1', {style: {backgroundColor: 'red'}}, 'Hello world'),
  h('p', {on: {click: eventHandler}}, 'Hello P')
])

function eventHandler() {
  alert('疼,别摸我')
}

const app = document.querySelector('#app')

patch(app,vnode)

```

当init使用了导入的模块就能够在h函数中用这些模块提供的api去创建虚拟DOM(Vnode)对象;在上文中就使用了<strong>样式模块</strong>以及<strong>事件模块</strong>让创建的这个虚拟DOM具备样式属性以及事件属性,最终通过patch函数对比两个虚拟dom(会先把app转换成虚拟dom),更新视图;

![image](https://user-images.githubusercontent.com/37037802/140879115-9c327c7b-dd3a-4981-a641-9a93d61666cf.png)

我们再简单看看init的源码部分:

```
// src/package/init.ts
/* 第一参数就是各个模块
   第二参数就是DOMAPI,可以把DOM转换成别的平台的API,
也就是说支持跨平台使用,当不传的时候默认是htmlDOMApi,见下文
   init是一个高阶函数,一个函数返回另外一个函数,可以缓存modules,与domApi两个参数,
那么以后直接只传oldValue跟newValue(vnode)就可以了*/
export function init (modules: Array<Partial<Module>>, domApi?: DOMAPI) {

...

return function patch (oldVnode: VNode | Element, vnode: VNode): VNode {}
}

```

#### h函数
有些地方（Vue源码中的Virtual DOM）也会用createElement来命名,它们是一样的东西,都是创建虚拟DOM的,在上述文章中相信大伙已经对h函数有一个初步的了解并且已经联想了使用场景,就不作场景案例介绍了,直接上源码部分:
```
// h函数
export function h (sel: string): VNode
export function h (sel: string, data: VNodeData | null): VNode
export function h (sel: string, children: VNodeChildren): VNode
export function h (sel: string, data: VNodeData | null, children: VNodeChildren): VNode
export function h (sel: any, b?: any, c?: any): VNode {
  var data: VNodeData = {}
  var children: any
  var text: any
  var i: number
    ...
  return vnode(sel, data, children, text, undefined) //最终返回一个vnode函数
};

```

```
// vnode函数
export function vnode (sel: string | undefined,
  data: any | undefined,
  children: Array<VNode | string> | undefined,
  text: string | undefined,
  elm: Element | Text | undefined): VNode {
  const key = data === undefined ? undefined : data.key
  return { sel, data, children, text, elm, key } //最终生成Vnode对象
}

```
##### 总结：h函数先生成一个vnode函数，然后vnode函数再生成一个Vnode对象(虚拟DOM对象)

#### patch函数(核心)
- pactch(oldVnode,newVnode)
- 把新节点中变化的内容渲染到真实DOM,最后返回新节点作为下一次处理的旧节点(核心)
- 对比新旧VNode是否相同节点(节点的key和sel相同)
- 如果不是相同节点,删除之前的内容,重新渲染
- 如果是相同节点,再判断新的VNode是否有text,如果有并且和oldVnode的text不同直接更新文本内容(patchVnode)
- 如果新的VNode有children,判断子节点是否有变化(updateChildren,最麻烦,最难实现)

源码:
```
return function patch(oldVnode: VNode | Element, vnode: VNode): VNode {    
    let i: number, elm: Node, parent: Node
    const insertedVnodeQueue: VNodeQueue = []
    // cbs.pre就是所有模块的pre钩子函数集合
    for (i = 0; i < cbs.pre.length; ++i) cbs.pre[i]()
    // isVnode函数时判断oldVnode是否是一个虚拟DOM对象
    if (!isVnode(oldVnode)) {
        // 若不是即把Element转换成一个虚拟DOM对象
        oldVnode = emptyNodeAt(oldVnode)
    }
    // sameVnode函数用于判断两个虚拟DOM是否是相同的,源码见补充1;
    if (sameVnode(oldVnode, vnode)) {
        // 相同则运行patchVnode对比两个节点,关于patchVnode后面会重点说明(核心)
        patchVnode(oldVnode, vnode, insertedVnodeQueue)
    } else {
        elm = oldVnode.elm! // !是ts的一种写法代码oldVnode.elm肯定有值
        // parentNode就是获取父元素
        parent = api.parentNode(elm) as Node

        // createElm是用于创建一个dom元素插入到vnode中(新的虚拟DOM)
        createElm(vnode, insertedVnodeQueue)

        if (parent !== null) {
            // 把dom元素插入到父元素中,并且把旧的dom删除
            api.insertBefore(parent, vnode.elm!, api.nextSibling(elm))// 把新创建的元素放在旧的dom后面
            removeVnodes(parent, [oldVnode], 0, 0)
        }
    }

    for (i = 0; i < insertedVnodeQueue.length; ++i) {
        insertedVnodeQueue[i].data!.hook!.insert!(insertedVnodeQueue[i])
    }
    for (i = 0; i < cbs.post.length; ++i) cbs.post[i]()
    return vnode
}

```

sameVnode函数
```
function sameVnode(vnode1: VNode, vnode2: VNode): boolean { 通过key和sel选择器判断是否是相同节点
    return vnode1.key === vnode2.key && vnode1.sel === vnode2.sel
}

```

#### patchVnode

- 第一阶段触发prepatch函数以及update函数(都会触发prepatch函数,两者不完全相同才会触发update函数)
- 第二阶段,真正对比新旧vnode差异的地方
- 第三阶段,触发postpatch函数更新节点

源码:

```
function patchVnode(oldVnode: VNode, vnode: VNode, insertedVnodeQueue: VNodeQueue) {
    const hook = vnode.data?.hook
    hook?.prepatch?.(oldVnode, vnode)
    const elm = vnode.elm = oldVnode.elm!
    const oldCh = oldVnode.children as VNode[]
    const ch = vnode.children as VNode[]
    if (oldVnode === vnode) return
    if (vnode.data !== undefined) {
        for (let i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
        vnode.data.hook?.update?.(oldVnode, vnode)
    }
    if (isUndef(vnode.text)) { // 新节点的text属性是undefined
        if (isDef(oldCh) && isDef(ch)) { // 当新旧节点都存在子节点
            if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue) //并且他们的子节点不相同执行updateChildren函数,后续会重点说明(核心)
        } else if (isDef(ch)) { // 只有新节点有子节点
            // 当旧节点有text属性就会把''赋予给真实dom的text属性
            if (isDef(oldVnode.text)) api.setTextContent(elm, '') 
            // 并且把新节点的所有子节点插入到真实dom中
            addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
        } else if (isDef(oldCh)) { // 清除真实dom的所有子节点
            removeVnodes(elm, oldCh, 0, oldCh.length - 1)
        } else if (isDef(oldVnode.text)) { // 把''赋予给真实dom的text属性
            api.setTextContent(elm, '')
        }
    } else if (oldVnode.text !== vnode.text) { //若旧节点的text与新节点的text不相同
        if (isDef(oldCh)) { // 若旧节点有子节点,就把所有的子节点删除
            removeVnodes(elm, oldCh, 0, oldCh.length - 1)
        }
        api.setTextContent(elm, vnode.text!) // 把新节点的text赋予给真实dom
    }
    hook?.postpatch?.(oldVnode, vnode) // 更新视图
}
```

看得可能有点蒙蔽,下面再上一副思维导图:

![image](https://user-images.githubusercontent.com/37037802/140885965-3e4a20e1-520c-42b3-bc5f-ac27b68dca60.png)

### diff算法简介

#### 传统diff算法

- 查找两颗树每一个节点的差异,会运行n1(dom1的节点数)*n2(dom2的节点数)次方去对比,找到差异的部分再去更新

![image](https://user-images.githubusercontent.com/37037802/140890702-27dd6b43-118b-4acd-9ef7-3f24cff925d1.png)


#### snabbdom的diff算法
- Snbbdom根据DOM的特点对传统的diff算法做了优化
- DOM操作时候很少会跨级别操作节点
- 只比较同级别的节点

![image](https://user-images.githubusercontent.com/37037802/140891128-c73e424a-916f-4853-a533-6a3b9664ce42.png)

#### 因为DOM操作时候很少会跨级别操作节点，所以在节点比较的时候只需比较同级别的节点，如果同级别的节点不相同，则直接删除重新创建，同级别的节点只需比较一次，从而达到减少对比次数的目的。

同级别节点比较的4种情况:

- oldStartVnode/newStartVnode(旧开始节点/新开始节点)相同
- oldEndVnode/newEndVnode(旧结束节点/新结束节点)相同
- oldStartVnode/newEndVnode(旧开始节点/新结束节点)相同
- oldEndVnode/newStartVnode(旧结束节点/新开始节点)相同
- 上述4种情况都不符合（即旧新开始节点和结束节点都不相同）

![image](https://user-images.githubusercontent.com/37037802/140893476-e35109db-9443-4ab6-b583-08561b4efacc.png)

执行过程是一个循环,在每次循环里,只要执行了上述的情况的4种之一就会结束一次循环

循环结束的收尾工作:直到oldStartIdx>oldEndIdx || newStartIdx>newEndIdx(代表旧节点或者新节点已经遍历完)

#### 比较新开始节点和旧开始节点(情况1)

- 如果新旧开始节点是sameVnode(key和sel相同)，则执行patchVnode找出两者之间的差异,更新节点（把这部分节点转换为真实DOM，并存储在vode.elm属性，注意这里并没有渲染至页面），
比较完成后索引会移动至下个索引的位置（oldStartIdx++/newStartIdx++）
- 如果新旧开始节点不是sameVnode(key和sel相同)，则继续往下判断（进行情况2）

![image](https://user-images.githubusercontent.com/37037802/141061808-289f6722-ba31-4683-962c-cfbd5db0ee6b.png)

#### 比较新结束节点和旧结束节点(情况2)

- 如果新旧结束节点是sameVnode(key和sel相同)，则执行patchVnode找出两者之间的差异,更新节点（把这部分节点转换为真实DOM，并存储在vode.elm属性，注意这里并没有渲染至页面），
比较完成后索引会往前移动一个位置（即倒数第二个节点作为新旧结束节点索引的位置）（oldEndIdx--/newEndIdx--）
- 如果新结束旧节点不是sameVnode(key和sel相同)，则继续往下判断（进行情况3）

![image](https://user-images.githubusercontent.com/37037802/141064292-a90536e3-e6d8-44d9-a5b9-9298809520fa.png)

#### 比较旧开始节点和新结束节点(情况3)
- 如果旧开始节点和新结束节点是sameVnode(key和sel相同)，则执行patchVnode找出两者之间的差异,更新节点（把这部分节点转换为真实DOM，并存储在vode.elm属性，注意这里并没有渲染至页面），
比较完成后旧开始节点转为对应的真实dom后会移动至最后的位置，旧开始节点的索引会往后移动一个位置，新结束节点会往前移动一个位置（oldStartIdx++/newEndIdx--）
- 如果旧开始节点和新结束节点不是sameVnode(key和sel相同)，则继续往下判断（进行情况4）

![image](https://user-images.githubusercontent.com/37037802/141066163-6d4bb662-4df4-400b-afe2-b3b6f17a6280.png)

#### 比较旧结束节点和新开始节点(情况4)

- 如果旧结束节点和新开始节点是sameVnode(key和sel相同)，则执行patchVnode找出两者之间的差异,更新节点（把这部分节点转换为真实DOM，并存储在vode.elm属性，注意这里并没有渲染至页面），
比较完成后旧结束节点转为对应的真实dom并移动至最前的位置，旧结束节点的索引会往前移动一个位置，新开始节点会往后移动一个位置（oldEndIdx--/newStartIdx++）
- 如果旧结束节点和新开始节点不是sameVnode(key和sel相同)，则继续往下判断（进行情况5）

![image](https://user-images.githubusercontent.com/37037802/141069524-4772dfce-4e9e-417e-b438-52f6ca10227a.png)

#### 上述4种情况都不符合（即旧新开始节点和结束节点都不相同）(情况5)

- 上述4种情况都不符合，则在旧节点（oldVnodes）里面遍历寻找跟新开始节点（newStartVnode）相同的节点
- 若没有找到相同的节点（说明新开始节点是新的节点）则创建新开始节点对应的真实DOM节点并插入至旧节点对应的真实DOM的最前位置（即旧开始节点对应的真实DOM节点(oldStartVnode.elm)前）
- 若找到了相同的节点，则执行patchVnode找出两者之间的差异,更新节点，然后把找到的旧节点（elmToMove）对应的真实DOM移动至旧节点对应的真实DOM的最前位置(（即旧开始节点对应的真实DOM节点(oldStartVnode.elm)前)
- 新开始节点会往后移动一个位置（newStartIdx++）

![image](https://user-images.githubusercontent.com/37037802/141090349-e2ba3c97-dca5-4f63-9f06-5cf74a4e27be.png)


#### 结束循环后存在2种情况(oldStartIdx>oldEndIdx || newStartIdx>newEndIdx)

- 若旧节点先遍历完（oldStartIdx>oldEndIdx）说明新节点有剩余，然后把剩余的新节点插入至旧节点对应的真实DOM的末尾位置
- 若新节点先遍历完（newStartIdx>newEndIdx）说明旧节点有剩余，然后把旧节点中剩余节点批量删除

