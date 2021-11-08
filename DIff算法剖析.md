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
- 复杂视图情况下提升渲染性能(ps:复杂视图情况操作dom性能消耗大,减少操作dom的范围可以提升性能)
