## 摘要

我们都知道Vue的响应式是通过Object.defineProperty来进行数据劫持。但是那是针对Object类型可以实现, 如果是数组呢? 通过set/get方式是不行的。

但是Vue作者使用了一个方式来实现Array类型的监测: 拦截器。

### 核心思想

通过创建一个拦截器来覆盖数组本身的原型对象Array.prototype。


### 拦截器

通过查看Vue源码路径vue/src/core/observer/array.js。

```
/**
 * Vue对数组的变化侦测
 * 思想: 通过一个拦截器来覆盖Array.prototype。
 * 拦截器其实就是一个Object, 它的属性与Array.prototype一样。 只是对数组的变异方法进行了处理。
*/

function def (obj, key, val, enumerable) {
    Object.defineProperty(obj, key, {
      value: val,
      enumerable: !!enumerable,
      writable: true,
      configurable: true
    })
}

// 数组原型对象
const arrayProto = Array.prototype
// 拦截器
const arrayMethods = Object.create(arrayProto)

// 变异数组方法：执行后会改变原始数组的方法
const methodsToPatch = [
    'push',
    'pop',
    'shift',
    'unshift',
    'splice',
    'sort',
    'reverse'
]

methodsToPatch.forEach(function (method) {
    // 缓存原始的数组原型上的方法
    const original = arrayProto[method]
    // 对每个数组编译方法进行处理(拦截)
    def(arrayMethods, method, function mutator (...args) {
      // 返回的value还是通过数组原型方法本身执行的结果
      const result = original.apply(this, args)
      // 每个value在被observer()时候都会打上一个__ob__属性
      const ob = this.__ob__
      // 存储调用执行变异数组方法导致数组本身值改变的数组，主要指的是原始数组增加的那部分(需要重新Observer)
      let inserted
      switch (method) {
        case 'push':
        case 'unshift':
          inserted = args
          break
        case 'splice':
          inserted = args.slice(2)
          break
      }
      // 重新Observe新增加的数组元素
      if (inserted) ob.observeArray(inserted)
      // 发送变化通知
      ob.dep.notify()
      return result
    })
})

```

### 关于Vue什么时候对data属性进行Observer

如果熟悉Vue源码的童鞋应该很快能找到Vue的入口文件vue/src/core/instance/index.js。

```
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
// 给原型绑定代理属性$props, $data
// 给Vue原型绑定三个实例方法: vm.$watch，vm.$set，vm.$delete
stateMixin(Vue)
// 给Vue原型绑定事件相关的实例方法: vm.$on, vm.$once ,vm.$off , vm.$emit
eventsMixin(Vue)
// 给Vue原型绑定生命周期相关的实例方法: vm.$forceUpdate, vm.destroy, 以及私有方法_update
lifecycleMixin(Vue)
// 给Vue原型绑定生命周期相关的实例方法: vm.$nextTick, 以及私有方法_render, 以及一堆工具方法
renderMixin(Vue)

export default Vue
```

#### this.__init__()

源码路径: vue/src/core/instance/init.js。

```

export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    // 当前实例
    const vm: Component = this
    // a uid
    // 实例唯一标识
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    // 开发模式, 开启Vue性能检测和支持 performance.mark API 的浏览器上。
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      // 处于组件初始化阶段开始打点
      mark(startTag)
    }

    // a flag to avoid this being observed
    // 标识为一个Vue实例
    vm._isVue = true
    // merge options
    // 把我们传入的optionsMerge到$options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    // 初始化生命周期
    initLifecycle(vm)
    // 初始化事件中心
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    // 初始化State
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }
    // 挂载
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

#### initState()

源码路径:vue/src/core/instance/state.js。

```
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}

```

这个时候你会发现observe出现了。

#### observe

源码路径: vue/src/core/observer/index.js

```
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
   // value已经是一个响应式数据就不再创建Observe实例, 避免重复侦听
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    // 出现目标, 创建一个Observer实例
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

### 使用拦截器的时机

Vue的响应式系统中有个Observe类。源码路径:vue/src/core/observer/index.js。


```
// can we use __proto__?
export const hasProto = '__proto__' in {}

const arrayKeys = Object.getOwnPropertyNames(arrayMethods)

function protoAugment (target, src: Object) {
  /* eslint-disable no-proto */
  target.__proto__ = src
  /* eslint-enable no-proto */
}

function copyAugment (target: Object, src: Object, keys: Array<string>) {
  // target: 需要被Observe的对象
  // src: 数组代理原型对象
  // keys: const arrayKeys = Object.getOwnPropertyNames(arrayMethods)
  // keys: 数组代理原型对象上的几个编译方法名
  // const methodsToPatch = [
  //   'push',
  //   'pop',
  //   'shift',
  //   'unshift',
  //   'splice',
  //   'sort',
  //   'reverse'
  // ]
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i]
    def(target, key, src[key])
  }
}

export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    //
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    // 如果是数组
    if (Array.isArray(value)) {
      if (hasProto) {
        // 如果支持__proto__属性(非标属性, 大多数浏览器支持): 直接把原型指向代理原型对象
        protoAugment(value, arrayMethods)
      } else {
        // 不支持就在数组实例上挂载被加工处理过的同名的变异方法(且不可枚举)来进行原型对象方法拦截
        // 当你访问一个对象的方法时候, 只有当自身不存在时候才会去原型对象上查找
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * 遍历数组每一项来进行侦听变化,即每个元素执行一遍Observer()
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

### 如何收集依赖

Vue里面真正做数据响应式处理的是defineReactive()。
defineReactive方法就是把对象的数据属性转为访问器属性, 即为数据属性设置get/set。

```
function dependArray (value: Array<any>) {
  for (let e, i = 0, l = value.length; i < l; i++) {
    e = value[i]
    e && e.__ob__ && e.__ob__.dep.depend()
    if (Array.isArray(e)) {
      dependArray(e)
    }
  }
}


export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  // dep在访问器属性中闭包使用
  // 每一个数据字段都通过闭包引用着属于自己的 dep 常量
  // 每个字段的Dep对象都被用来收集那些属于对应字段的依赖。
  const dep = new Dep()

  // 获取该字段可能已有的属性描述对象
  const property = Object.getOwnPropertyDescriptor(obj, key)
  // 边界情况处理： 一个不可配置的属性是不能使用也没必要使用 Object.defineProperty 改变其属性定义的。
  if (property && property.configurable === false) {
    return
  }

  // 由于一个对象的属性很可能已经是一个访问器属性了，所以该属性很可能已经存在 get 或 set 方法
  // 如果接下来会使用 Object.defineProperty 函数重新定义属性的 setter/getter
  // 这会导致属性原有的 set 和 get 方法被覆盖，所以要将属性原有的 setter/getter 缓存
  const getter = property && property.get
  const setter = property && property.set
  // 边界情况处理
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }
  // 默认就是深度观测，引用子属性的__ob__
  // 为Vue.set 或 Vue.delete 方法提供触发依赖。
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      // 如果 getter 存在那么直接调用该函数，并以该函数的返回值作为属性的值，保证属性的原有读取操作正常运作
      // 如果 getter 不存在则使用 val 作为属性的值
      const value = getter ? getter.call(obj) : val
      // Dep.target的值是在对Watch实例化时候赋值的
      if (Dep.target) {
        // 开始收集依赖到dep
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            // 调用 dependArray 函数逐个触发数组每个元素的依赖收集
            dependArray(value)
          }
        }
      }
      // 正确地返回属性值。
      return value
    },
    set: function reactiveSetter (newVal) {
      // 获取原来的值
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      // 比较新旧值是否相等, 考虑NaN情况
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      // 如果数据之前有setter, 那么应该继续使用该函数来设置属性的值
      if (setter) {
        setter.call(obj, newVal)
      } else {
        // 赋新值
        val = newVal
      }
      // 由于属性被设置了新的值，那么假如我们为属性设置的新值是一个数组或者纯对象，
      // 那么该数组或纯对象是未被观测的，所以需要对新值进行观测
      childOb = !shallow && observe(newVal)
      // 通知dep中的watcher更新
      dep.notify()
    }
  })
}

```

### 存储数组依赖的列表

我们为什么需要把依赖存在Observer实例上。
即
```
export class Observer {
    constructor (value: any) {
        ...
        this.dep = new Dep()
    }
}
```

首先我们需要在getter里面访问到Observer实例
```
// 即上述的
let childOb = !shallow && observe(val)
...
if (childOb) {
  // 调用Observer实例上dep的depend()方法收集依赖
  childOb.dep.depend()
  if (Array.isArray(value)) {
    // 调用 dependArray 函数逐个触发数组每个元素的依赖收集
    dependArray(value)
  }
}
```
另外我们在前面提到的拦截器中要使用Observer实例。

```
methodsToPatch.forEach(function (method) {
    ...
    // this表示当前被操作的数据
    // 但是__ob__怎么来的?
    const ob = this.__ob__
    ...
    // 重新Observe新增加的数组元素
    if (inserted) ob.observeArray(inserted)
    // 发送变化通知
    ob.dep.notify()
    ...
})
```

思考上述的this.__ob__属性来自哪里?

```
export class Observer {
    constructor () {
        ...
        this.dep = new Dep()
        // 在vue上新增一个不可枚举的__ob__属性, 这个属性的值就是Observer实例
        // 因此我们就可以通过数组数据__ob__获取Observer实例
        // 进而获取__ob__上的dep
        def(value, '__ob__', this)
        ...
    }
}
```

牢记所有的属性一旦被侦测了都会被打上一个__ob__的标记, 即表示是响应式数据。

#### 关于Array注意事项

由于 JavaScript 的限制，Vue 不能检测以下变动的数组：

- 当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue
- 当你修改数组的长度时，例如：vm.items.length = newLength



解决方法见官网文档:

[关于Array注意事项](https://cn.vuejs.org/v2/guide/list.html#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)


