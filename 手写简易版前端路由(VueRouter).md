## 手写简易版前端路由(VueRouter)

```
// console.dir(Vue)
let _Vue = null
export default class VueRouter {
  static install (Vue) {
    // console.dir(this)
    // 1 判断当前插件是否被安装
    if (VueRouter.install.installed) {
      return
    }
    VueRouter.install.installed = true
    // 2 把Vue的构造函数记录在全局
    _Vue = Vue
    // 3 把创建Vue的实例传入的router对象注入到Vue实例
    // _Vue.prototype.$router = this.$options.router
    _Vue.mixin({
      beforeCreate () {
        if (this.$options.router) {
          _Vue.prototype.$router = this.$options.router
        }
        // console.log(this)
      }
    })
  }

  constructor (options) {
    this.options = options
    this.routeMap = {}

    // mode为hash时,需要在域名后加上 "/#"
    if (this.options.mode !== 'history') {
      // 判断当前路径是否为hash模式
      if (window.location.pathname === '/' && window.location.hash) {
        history.replaceState('', '', `/#${window.location.hash.slice(1)}`)
      } else {
        history.replaceState('', '', `/#${window.location.pathname}`)
      }
    }

    // observable
    this.data = _Vue.observable({
      current:
        this.options.mode === 'history'
          ? window.location.pathname
          : window.location.hash.slice(1)
    })

    this.init()
  }

  init () {
    this.createRouteMap()
    this.initComponent(_Vue)
    this.initEvent()
  }

  createRouteMap () {
    // 遍历所有的路由规则 吧路由规则解析成键值对的形式存储到routeMap中
    this.options.routes.forEach(route => {
      this.routeMap[route.path] = route.component
    })
  }

  initComponent (Vue) {
    const self = this
    Vue.component('router-link', {
      props: {
        to: String
      },
      render (h) {
        return h(
          'a',
          {
            attrs: {
              href: this.to
            },
            on: {
              click: this.clickhander
            }
          },
          [this.$slots.default]
        )
      },
      methods: {
        clickhander (e) {
          if (self.options.mode === 'history') {
            history.pushState({}, '', this.to)
          } else {
            history.pushState({}, '', `/#${this.to}`)
          }

          this.$router.data.current = this.to
          e.preventDefault()
        }
      }
      // template:"<a :href='to'><slot></slot><>"
    })

    Vue.component('router-view', {
      render (h) {
        // self.data.current
        const cm = self.routeMap[self.data.current]
        return h(cm)
      }
    })
  }

  initEvent () {
    // mode为histroy是使用popsate事件,否则mode为hash，使用hashchange事件
    if (this.options.mode === 'history') {
      window.addEventListener('popstate', () => {
        this.data.current = window.location.pathname
      })
    } else {
      window.addEventListener('hashchange', () => {
        this.data.current = window.location.hash.slice(1)
      })
    }
  }
}

```
