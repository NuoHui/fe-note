### 事件相关实例方法

源码位置: vue-dev/src/core/instance/events.js

与事件相关的方法有:

```
vm.$on, vm.$once, vm.$off, vm.$emit
```

#### initEvents()

这里其实主要是一个对象存储, key=eventName(事件名), value=[事件回调列表]。

```
export function initEvents (vm: Component) {
  // _events是一个对象
  // 格式为 {[event(事件名): [事件回调列表]]}
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
```

至于initEvent()方法是在new Vue()时候, 会执行__init()方法。

```
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    ...
    // 执行各种初始化操作
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
  }
}
```

#### vm.$on()

监听当前实例上的自定义事件。事件可以由vm.$emit触发。回调函数会接收所有传入事件触发函数的额外参数。
使用方法就参见官网[vm.$on()](https://cn.vuejs.org/v2/api/#vm-on)。

源码分析:

```
const hookRE = /^hook:/
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
const vm: Component = this
// 如果event是数组，循环数组, 每一项调用$on
// 主要是使回调能注册到每项事件名指定的事件列表
if (Array.isArray(event)) {
  for (let i = 0, l = event.length; i < l; i++) {
    vm.$on(event[i], fn)
  }
} else {
  // 如果不是数组, 直接往事件列表添加回调
  // 如果事件名不存在默认初始化为空列表
  (vm._events[event] || (vm._events[event] = [])).push(fn)
  // optimize hook:event cost by using a boolean flag marked at registration
  // instead of a hash lookup
  if (hookRE.test(event)) {
    vm._hasHookEvent = true
  }
}
return vm
}
```

#### vm.$off()

用来移除事件监听器, 具体使用方法见官网[vm.off()](https://cn.vuejs.org/v2/api/#vm-off)。

源码分析:

```
Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {
    const vm: Component = this
    // all
    // 么有参数时候默认移除所有的监听器
    // 通过重置_events对象为{}
    if (!arguments.length) {
      vm._events = Object.create(null)
      return vm
    }
    // array of events
    // 如果是数组, 遍历逐个移除对应的eventName的回调
    if (Array.isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$off(event[i], fn)
      }
      return vm
    }
    // specific event
    // cbs: eventName对应的回调
    const cbs = vm._events[event]
    // 如果没有回调默认不处理直接返回
    if (!cbs) {
      return vm
    }
    // 如果没有指定具体的回调, 默认清空对应事件名注册下的所有回调函数
    if (!fn) {
      vm._events[event] = null
      return vm
    }
    // specific handler
    // 走到这里说明有fn, 从事件列表回调数组中遍历匹配, 匹配成功就删除
    let cb
    let i = cbs.length
    // 这里有个小技巧, 从后往前遍历如果匹配中删除监听器, 不会影响前面未遍历到的位置
    while (i--) {
      cb = cbs[i]
      // 如果列表中某一项与fn相同, 或者某一项的fn属性与fn一致, 说明匹配成功
      if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1)
        break
      }
    }
    return vm
  }

```

#### vm.$once()

监听一个自定义事件，但是只触发一次，在第一次触发之后移除监听器。
见官网介绍[vm.$once()](https://cn.vuejs.org/v2/api/#vm-once)。

源码分析:

```
Vue.prototype.$once = function (event: string, fn: Function): Component {
    // fn:是用户指定的回调函数
    const vm: Component = this
    // on: 是我们拦截器函数
    function on () {
      // 执行回调时候通过$off()移除事件回调
      vm.$off(event, on)
      // 当事件触发时候执行指定的回调函数
      fn.apply(vm, arguments)
    }
    // 之所以需要on.fn = fn,是因为我们下面使用$on()时候
    // 往事件回调列表推入的是on函数不是用户指定的函数fn.
    // 这个时候使用$off()移除时候, 有一行代码
    /**
     * if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1)
        break
      }
    */
    on.fn = fn
    // 通过$on()监听自定义事件
    vm.$on(event, on)
    return vm
  }
```

#### vm.$emit()

触发当前实例上的事件。附加参数都会传给监听器回调。
见官网使用[vm.$emit()](https://cn.vuejs.org/v2/api/#vm-emit)。

源码分析:

```
  Vue.prototype.$emit = function (event: string): Component {
    const vm: Component = this
    if (process.env.NODE_ENV !== 'production') {
      const lowerCaseEvent = event.toLowerCase()
      if (lowerCaseEvent !== event && vm._events[lowerCaseEvent]) {
        tip(
          `Event "${lowerCaseEvent}" is emitted in component ` +
          `${formatComponentName(vm)} but the handler is registered for "${event}". ` +
          `Note that HTML attributes are case-insensitive and you cannot use ` +
          `v-on to listen to camelCase events when using in-DOM templates. ` +
          `You should probably use "${hyphenate(event)}" instead of "${event}".`
        )
      }
    }
    // 取出事件名对应的回调函数列表
    let cbs = vm._events[event]
    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs
      // args: 为除了第一个函数外所有参数组成的数组
      const args = toArray(arguments, 1)
      const info = `event handler for "${event}"`
      for (let i = 0, l = cbs.length; i < l; i++) {
        // cbs[i]: 回调函数
        invokeWithErrorHandling(cbs[i], vm, args, vm, info)
      }
    }
    return vm
  }
```

```
export function invokeWithErrorHandling (
  handler: Function,
  context: any,
  args: null | any[],
  vm: any,
  info: string
) {
  let res
  try {
    // 执行事件对应回调函数
    res = args ? handler.apply(context, args) : handler.call(context)
    if (res && !res._isVue && isPromise(res) && !res._handled) {
      //  处理Promise嵌套回调
      res.catch(e => handleError(e, vm, info + ` (Promise/async)`))
      // issue #9511
      // avoid catch triggering multiple times when nested calls
      res._handled = true
    }
  } catch (e) {
    // 如果出错, 执行错误处理
    handleError(e, vm, info)
  }
  return res
}
```
