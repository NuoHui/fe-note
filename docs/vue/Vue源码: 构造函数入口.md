## 构造函数的入口

一步步找到Vue的构造函数入口。

### 执行npm run dev

通过查看package.json文件下的scripts命令。

```
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev"
```

scripts/config.js为打开的对应配置文件, process.env.TARGET为web-full-dev。
在scripts/config.js找到对应的配置对象

```
const builds = {
    // Runtime+compiler development build (Browser)
    'web-full-dev': {
        entry: resolve('web/entry-runtime-with-compiler.js'),
        dest: resolve('dist/vue.js'),
        format: 'umd',
        env: 'development',
        alias: { he: './entity-decoder' },
        banner
    },
}
```
当然主要生成配置对象是这段代码

```
function genConfig (name) {
  // opts为builds里面对应key的基础配置对象
  const opts = builds[name]
  // config是真正要返回的配置对象
  const config = {
    input: opts.entry,
    external: opts.external,
    plugins: [
      flow(),
      alias(Object.assign({}, aliases, opts.alias))
    ].concat(opts.plugins || []),
    output: {
      file: opts.dest,
      format: opts.format,
      banner: opts.banner,
      name: opts.moduleName || 'Vue'
    },
    onwarn: (msg, warn) => {
      if (!/Circular/.test(msg)) {
        warn(msg)
      }
    }
  }

  // built-in vars
  const vars = {
    __WEEX__: !!opts.weex,
    __WEEX_VERSION__: weexVersion,
    __VERSION__: version
  }
  // feature flags
  Object.keys(featureFlags).forEach(key => {
    vars[`process.env.${key}`] = featureFlags[key]
  })
  // build-specific env
  // 根据不同的process.env.NODE_ENV加载不同的打包后版本
  if (opts.env) {
    vars['process.env.NODE_ENV'] = JSON.stringify(opts.env)
  }
  config.plugins.push(replace(vars))

  if (opts.transpile !== false) {
    config.plugins.push(buble())
  }

  Object.defineProperty(config, '_name', {
    enumerable: false,
    value: name
  })

  return config
}

if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}

```

### 找到打包入口文件

根据配置对象的entry字段:

```
entry: resolve('web/entry-runtime-with-compiler.js')

```

以及resolve函数

```
const aliases = require('./alias')
const resolve = p => {
  // web/ weex /server
  const base = p.split('/')[0]
  if (aliases[base]) {
    // 拼接完整的入口文件
    return path.resolve(aliases[base], p.slice(base.length + 1))
  } else {
    return path.resolve(__dirname, '../', p)
  }
}
```
aliases.js文件

```
const path = require('path')

const resolve = p => path.resolve(__dirname, '../', p)

module.exports = {
  vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
  compiler: resolve('src/compiler'),
  core: resolve('src/core'),
  shared: resolve('src/shared'),
  web: resolve('src/platforms/web'),
  weex: resolve('src/platforms/weex'),
  server: resolve('src/server'),
  sfc: resolve('src/sfc')
}
```

找到真正的入口文件为: vue-dev/src/platforms/web/entry-runtime-with-compiler.js。

在entry-runtime-with-compiler.js文件中发现
```
import Vue from './runtime/index'
```
其实这里主要做的是挂载$mount()方法, 可以看我之前写的文章[mount挂载函数](https://juejin.im/post/5c8531995188251bbf2edf82)。

OK回到继续回到我们之前话题, 在vue-dev/src/platforms/web/runtime/index.js下发现这里还不是真正的Vue构造函数

```
import Vue from './instance/index'
```
不过也马上接近了, 继续查找vue-dev/src/core/instance/index.js, 很明显这里才是真正的构造函数。

```
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

// Vue构造函数
function Vue (options) {
  // 提示必须使用new Vue()
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // 执行初始化操作, 一般_前缀方法都是内部方法
  // __init()方法是initMixin里绑定的
  this._init(options)
}

// 在Vue原型上挂载方法
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

### initMixin()

```

export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    // 缓存this
    const vm: Component = this
    // a uid
    vm._uid = uid++

    // 这里只要是开启config.performance进行性能调试时候一些组件埋点
    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    // 标识一个对象是 Vue 实例, 避免再次被observed
    vm._isVue = true
    // merge options
    // options是new Vue(options)配置对象
    // _isComponent是一个内部属性, 用于创建组件
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      // 定义实例属性$options: 用于当前 Vue 实例的初始化选项
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
    // 定义一个内部属性_self
    vm._self = vm
    // 执行各种初始化操作
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }
    // 执行挂载操作
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```
