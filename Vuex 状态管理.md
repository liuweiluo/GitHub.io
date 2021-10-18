## Vuex 状态管理

### 组件内的状态管理流程

每个组件都有自己的状态、视图和行为等组成部分

```
new Vue({
    // state
    data() {
        return {
            count: 0
        };
    },
    // view
    template: `
    <div>{{ count }}</div>
    `,
    // actions
    methods: {
        increment() {
            this.count++;
        }
    }
});
```

#### 状态管理包含以下几部分：

- state，驱动应用的数据源；
- view，以声明方式将 state 映射到视图；
- actions，响应在 view 上的用户输入导致的状态变化

![image](https://user-images.githubusercontent.com/37037802/137696393-8abd7044-76e0-4f29-bcec-a599131a3f42.png)

### 组件间通信方式回顾

![image](https://user-images.githubusercontent.com/37037802/137700234-b1baab16-bd86-470a-9901-01313bf7b65d.png)

- 父传子：Props Down

  ```
    <blog-post title="My journey with Vue"></blog-post>
    
    Vue.component("blog-post", {
        props: ["title"],
        template: "<h3>{{ title }}</h3>"
    });
  ```
- 子传父：Event Up
  
  在子组件中使用 $emit 发布一个自定义事件：
  
  ```
  <button v-on:click="$emit('enlargeText', 0.1)">Enlarge text</button>;
  ```
  
  在使用这个组件的时候，使用 v-on 监听这个自定义事件
  
  ```
  <blog-post v-on:enlargeText="hFontSize += $event"></blog-pos>
  ```
  
- 非父子组件：Event Bus

  我们可以使用一个非常简单的 Event Bus 来解决这个问题：
  
  eventbus.js
  
  ```
  export default new Vue()
  ```
  
  然后在需要通信的两端：
  
  使用 $on 订阅：

  ```
  // 没有参数
  bus.$on("自定义事件名称", () => {
      // 执行操作
  });
  // 有参数
  bus.$on("自定义事件名称", data => {
      // 执行操作
  });
  ```
  
  使用 $emit 发布：
  
  ```
  // 没有自定义传参
  bus.$emit("自定义事件名称");
  // 有自定义传参
  bus.$emit("自定义事件名称", 数据);
  ```
  
- 父直接访问子组件：通过 ref 获取子组件
  
  ref 有两个作用：
  
   - 如果你把它作用到普通 HTML 标签上，则获取到的是 DOM
   - 如果你把它作用到组件标签上，则获取到的是组件实例


### 简易的状态管理方案
  
如果多个组件之间要共享状态(数据)，使用上面的方式虽然可以实现，但是比较麻烦，而且多个组件之间互相传值很难跟踪数据的变化，如果出现问题很难定位问题。当遇到多个组件需要共享状态的时候，典型的场景：购物车。我们如果使用上述的方案都不合适，我们
会遇到以下的问题

  - 多个视图依赖于同一状态
  - 来自不同视图的行为需要变更同一状态。
  
对于问题一，传参的方法对于多层嵌套的组件将会非常繁琐，并且对于兄弟组件间的状态传递无能为力。

对于问题二，我们经常会采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝。以上的这些模式非常脆弱，通常会导致无法维护的代码。

因此，我们为什么不把组件的共享状态抽取出来，以一个全局单例模式管理呢？在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为！

我们可以把多个组件的状态，或者整个程序的状态放到一个集中的位置存储，并且可以检测到数据的更改。你可能已经想到了 Vuex。

这里我们先以一种简单的方式来实现


