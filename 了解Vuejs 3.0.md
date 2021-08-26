## 了解Vuejs 3.0

### Vuejs 3.0 与 Vuejs 2.x 的区别

#### 1.源码组织方式不同

\- 源码采用 TypeScript 重写

\- 使用 Monorepo 管理项目结构

#### 2.构建版本不同
\- 去掉了UMD版本（通用的模块版本，支持多种模块方式）的支持

#### 3.新增Composition API
\- 基于函数的API，可以更灵活组织组件的逻辑，解决了Options API开发复杂组件，同一个功能逻辑的代码被拆分到不同选项的问题

#### 4.性能的提升
\- 响应式系统升级
- Vue.js 2.x 中响应式系统的核心 defineProperty
- Vue.js 3.0 中使用 Proxy 对象重写响应式系统   
   - 可以监听动态新增的属性
   - 可以监听删除的属性
   - 可以监听数组的索引和 length 属性
   
\- 编译优化
- Vue.js 2.x 中通过标记静态根节点，优化 diff 的过程
- Vue.js 3.0 中标记和提升所有的静态根节点，diff 的时候只需要对比动态节点内容

\- 优化打包体积
- Vue.js 3.0 中移除了一些不常用的 API 例如：inline-template、filter 等
- 对Tree-shaking的支持更佳
