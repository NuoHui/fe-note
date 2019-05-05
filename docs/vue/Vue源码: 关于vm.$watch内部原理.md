## vm.$watch()用法

关于vm.$watch()详细用法可以见[官网](https://cn.vuejs.org/v2/api/#vm-watch)。

大致用法如下:

```
<script>
    const app = new Vue({
        el: "#app",
        data: {
            a: {
                b: {
                    c: 'c'
                }
            }
        },
        mounted () {
            this.$watch(function () {
                return this.a.b.c
            }, this.handle, {
                deep: true,
                immediate: true // 默认会初始化执行一次handle
            })
        },
        methods: {
            handle (newVal, oldVal) {
                console.log(this.a)
                console.log(newVal, oldVal)
            },
            changeValue () {
                this.a.b.c = 'change'
            }
        }
    })
</script>
```



![截图](https://user-gold-cdn.xitu.io/2019/4/23/16a4ae24fdd84027?w=1096&h=1224&f=png&s=172271)

可以看到data属性整个a对象被Observe, 只要被Observe就会有一个__ob__标示(即Observe实例), 可以看到__ob__里面有dep，前面讲过依赖(dep)都是存在Observe实例里面, subs存储的就是对应属性的依赖(Watcher)。
好了回到正文, vm.$watch()在源码内部如果实现的。

### 内部实现原理


```
// 判断是否是对象
export function isPlainObject (obj: any): boolean {
  return _toString.call(obj) === '[object Object]'
}

```

源码位置: vue/src/core/instance/state.js

```
// $watch 方法允许我们观察数据对象的某个属性，当属性变化时执行回调
// 接受三个参数: expOrFn(要观测的属性), cb, options(可选的配置对象)
// cb即可以是一个回调函数, 也可以是一个纯对象(这个对象要包含handle属性。)
// options: {deep, immediate}, deep指的是深度观测, immediate立即执行回掉
// $watch()本质还是创建一个Watcher实例对象。

Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    // vm指向当前Vue实例对象
    const vm: Component = this
    if (isPlainObject(cb)) {
      // 如果cb是一个纯对象
      return createWatcher(vm, expOrFn, cb, options)
    }
    // 获取options
    options = options || {}
    // 设置user: true, 标示这个是由用户自己创建的。
    options.user = true
    // 创建一个Watcher实例
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      // 如果immediate为真, 马上执行一次回调。
      try {
        // 此时只有新值, 没有旧值, 在上面截图可以看到undefined。
        // 至于这个新值为什么通过watcher.value, 看下面我贴的代码
        cb.call(vm, watcher.value)
      } catch (error) {
        handleError(error, vm, `callback for immediate watcher "${watcher.expression}"`)
      }
    }
    // 返回一个函数，这个函数的执行会解除当前观察者对属性的观察
    return function unwatchFn () {
      // 执行teardown()
      watcher.teardown()
    }
  }
```


#### 关于watcher.js。

源码路径: vue/src/core/observer/watcher.js

```
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component, // 组件实例对象
    expOrFn: string | Function, // 要观察的表达式
    cb: Function, // 当观察的表达式值变化时候执行的回调
    options?: ?Object, // 给当前观察者对象的选项
    isRenderWatcher?: boolean // 标识该观察者实例是否是渲染函数的观察者
  ) {
    // 每一个观察者实例对象都有一个 vm 实例属性，该属性指明了这个观察者是属于哪一个组件的
    this.vm = vm
    if (isRenderWatcher) {
      // 只有在 mountComponent 函数中创建渲染函数观察者时这个参数为真
      // 组件实例的 _watcher 属性的值引用着该组件的渲染函数观察者
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    // deep: 当前观察者实例对象是否是深度观测
    // 平时在使用 Vue 的 watch 选项或者 vm.$watch 函数去观测某个数据时，
    // 可以通过设置 deep 选项的值为 true 来深度观测该数据。
    // user: 用来标识当前观察者实例对象是 开发者定义的 还是 内部定义的
    // 无论是 Vue 的 watch 选项还是 vm.$watch 函数，他们的实现都是通过实例化 Watcher 类完成的
    // sync: 告诉观察者当数据变化时是否同步求值并执行回调
    // before: 可以理解为 Watcher 实例的钩子，当数据变化之后，触发更新之前，
    // 调用在创建渲染函数的观察者实例对象时传递的 before 选项。
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    // cb: 回调
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    // 避免收集重复依赖，且移除无用依赖
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // 检测了 expOrFn 的类型
    // this.getter 函数终将会是一个函数
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    // 求值
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * 求值: 收集依赖
   * 求值的目的有两个
   * 第一个是能够触发访问器属性的 get 拦截器函数
   * 第二个是能够获得被观察目标的值
   */
  get () {
    // 推送当前Watcher实例到Dep.target
    pushTarget(this)
    let value
    // 缓存vm
    const vm = this.vm
    try {
      // 获取value
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        // 递归地读取被观察属性的所有子属性的值
        // 这样被观察属性的所有子属性都将会收集到观察者，从而达到深度观测的目的。
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * 记录自己都订阅过哪些Dep
   */
  addDep (dep: Dep) {
    const id = dep.id
    // newDepIds: 避免在一次求值的过程中收集重复的依赖
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id) // 记录当前watch订阅这个dep
      this.newDeps.push(dep) // 记录自己订阅了哪些dep
      if (!this.depIds.has(id)) {
        // 把自己订阅到dep
        dep.addSub(this)
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    //newDepIds 属性和 newDeps 属性被清空
    // 并且在被清空之前把值分别赋给了 depIds 属性和 deps 属性
    // 这两个属性将会用在下一次求值时避免依赖的重复收集。
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      // 指定同步更新
      this.run()
    } else {
      // 异步更新队列
      queueWatcher(this)
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    if (this.active) {
      const value = this.get()
      // 对比新值 value 和旧值 this.value 是否相等
      // 是对象的话即使值不变(引用不变)也需要执行回调
      // 深度观测也要执行
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          // 意味着这个观察者是开发者定义的，所谓开发者定义的是指那些通过 watch 选项或 $watch 函数定义的观察者
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            // 回调函数在执行的过程中其行为是不可预知, 出现错误给出提示
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }

  /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }

  /**
   * Depend on all deps collected by this watcher.
   */
  depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }

  /**
   * 把Watcher实例从从当前正在观测的状态的依赖列表中移除
   */
  teardown () {
    if (this.active) {
      // 该观察者是否激活状态
      if (!this.vm._isBeingDestroyed) {
        // _isBeingDestroyed一个标识，为真说明该组件实例已经被销毁了，为假说明该组件还没有被销毁
        // 将当前观察者实例从组件实例对象的 vm._watchers 数组中移除
        remove(this.vm._watchers, this)
      }
      // 当一个属性与一个观察者建立联系之后，属性的 Dep 实例对象会收集到该观察者对象
      let i = this.deps.length
      while (i--) {
        this.deps[i].removeSub(this)
      }
      // 非激活状态
      this.active = false
    }
  }
}
```



```
export const unicodeRegExp = /a-zA-Z\u00B7\u00C0-\u00D6\u00D8-\u00F6\u00F8-\u037D\u037F-\u1FFF\u200C-\u200D\u203F-\u2040\u2070-\u218F\u2C00-\u2FEF\u3001-\uD7FF\uF900-\uFDCF\uFDF0-\uFFFD/
const bailRE = new RegExp(`[^${unicodeRegExp.source}.$_\\d]`)
// path为keypath(属性路径) 处理'a.b.c'(即vm.a.b.c) => a[b[c]]
export function parsePath (path: string): any {
  if (bailRE.test(path)) {
    return
  }
  const segments = path.split('.')
  return function (obj) {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return
      obj = obj[segments[i]]
    }
    return obj
  }
}

```
