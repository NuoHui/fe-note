## 使用场景

我们在写 Vue.js 时，不论是用 CDN 的方式还是基于Webpack的SPA，都会有一个根节点，并且创建一个根实例，比如：

- CDN

```
<body>
  <div id="app"></div>
</body>
<script>
  const app = new Vue({
    el: '#app'
  });
</script>

```

- Webpack一般在入口文件 main.js 里创建一个实例：

```
import Vue from 'vue';
import App from './app.vue';

new Vue({
  el: '#app',
  render: h => h(App)
});

```

基于Webpack的SPA，它的 html 里一般都只有一个根节点 `<div id="app"></div>`，其余都是通过 JavaScript 完成，也就是许多的 Vue.js 组件（每个页面也是一个组件）。

有了初始化的实例，之后所有的页面，都由 vue-router 帮我们管理，组件也都是用 `import` 导入后局部注册（也有在 main.js 全局注册的），不管哪种方式，组件（或页面）的创建过程我们是无需关心的，只是写好 `.vue` 文件并导入即可。这样的组件使用方式，有几个特点：

1.  所有的内容，都是在 `#app` 节点内渲染的；
2.  组件的模板，是事先定义好的；
3.  由于组件的特性，注册的组件只能在当前位置渲染。

比如你要使用一个组件 `<i-date-picker>`，渲染时，这个自定义标签就会被替换为组件的内容，而且在哪写的自定义标签，就在哪里被替换。换句话说，常规的组件使用方式，只能在规定的地方渲染组件，这在一些特殊场景下就比较局限了，例如：

1.  组件的模板是通过调用接口从服务端获取的，需要动态渲染组件；
2.  实现类似原生 `window.alert()` 的提示框组件，它的位置是在 `<body>` 下，而非 `<div id="app">`，并且不会通过常规的组件自定义标签的形式使用，而是像 JS 调用函数一样使用。

一般来说，在我们访问页面时，组件就已经渲染就位了，对于场景 1，组件的渲染是异步的，甚至预先不知道模板是什么。对于场景 2，其实并不陌生，在 jQuery 时代，通过操作 DOM，很容易就能实现，你可以沿用这种思路，只是这种做法不那么 Vue，既然使用 Vue.js 了，就应该用 Vue 的思路来解决问题。对于这两种场景，Vue.extend 和 vm.$mount 语法就派上用场了。


## 方法

上文我们说到，创建一个 Vue 实例时，都会有一个选项 `el`，来指定实例的根节点，如果不写 `el` 选项，那组件就处于未挂载状态。`Vue.extend` 的作用，就是基于 Vue 构造器，创建一个“子类”，它的参数跟 `new Vue` 的基本一样，但 `data` 要跟组件一样，是个函数，再配合 `$mount` ，就可以让组件渲染，并且挂载到任意指定的节点上，比如 body。

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Vue Demo</title>
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
  <body>
    <script>
      // mock从后台获取template
      const getTemplate = () => {
        return '<div>{{msg}}</div>'
      }
      // 创建一个构造器
      const AlertComponent = Vue.extend({
        template: getTemplate(),
        data () {
          return {
            msg: 'hello vue'
          }
        }
      })
      const alertComponent = new AlertComponent().$mount();
      document.body.appendChild(alertComponent.$el)
    </script>
  </body>
</html>

```

当然也可以直接手动就挂载

```
// 手动直接挂载
new AlertComponent().$mount('#app')
new AlertComponent({ el: '#app' });
```

除了使用extend方法外还可以使用render函数创建Vue实例。

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Vue Demo</title>
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
  <body>
    <script>
      const props = {
        msg: 'hello vue'
      }
      // 创建一个构造器
      const instance = new Vue({
        render (h) {
          return h('div', {
            attrs: {
              class: 'alert'
            },
            domProps: {
              innerHTML: props.msg
            },
            props: props
          })
        }
      })
      const alertComponent = instance.$mount();
      document.body.appendChild(alertComponent.$el)
    </script>
  </body>
</html>

```

需要注意的是，我们是用 `$mount` 手动渲染的组件，如果要销毁，也要用 `$destroy` 来手动销毁实例，必要时，也可以用 `removeChild` 把节点从 DOM 中移除。


## $mount源码分析

### $mount函数执行位置


![new Vue()](https://user-gold-cdn.xitu.io/2019/3/10/169684ecc93f837e?w=2092&h=1196&f=jpeg&s=927379)

_init这个私有方法是在执行initMixin时候绑定到Vue原型上的。


![mount](https://user-gold-cdn.xitu.io/2019/3/10/1696850a6f0dad6d?w=2110&h=1118&f=jpeg&s=958661)

### $mount函数是如如何把组件挂在到指定元素

#### $mount函数定义位置

$mount函数定义位置有两个:

第一个是在src/platforms/web/runtime/index.js


![第一个](https://user-gold-cdn.xitu.io/2019/3/11/16968559a7eb9629?w=1778&h=904&f=jpeg&s=783777)

这里的$mount是一个public mount method。之所以这么说是因为Vue有很多构建版本, 有些版本会依赖此方法进行有些功能定制, 后续会解释。

```
// public mount method
// el: 可以是一个字符串或者Dom元素
// hydrating 是Virtual DOM 的补丁算法参数
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  // 判断el, 以及宿主环境, 然后通过工具函数query重写el。
  el = el && inBrowser ? query(el) : undefined
  // 执行真正的挂载并返回
  return mountComponent(this, el, hydrating)
}
```
src/platforms/web/runtime/index.js 文件是运行时版 Vue 的入口文件,所以这个方法是运行时版本Vue执行的$mount。

关于Vue不同构建版本可以看[Vue对不同构建版本的解释](https://cn.vuejs.org/v2/guide/installation.html#%E5%AF%B9%E4%B8%8D%E5%90%8C%E6%9E%84%E5%BB%BA%E7%89%88%E6%9C%AC%E7%9A%84%E8%A7%A3%E9%87%8A)。

关于这个作者封装的工具函数query也可以学习下:

```
/**
 * Query an element selector if it's not an element already.
 */
export function query (el: string | Element): Element {
  if (typeof el === 'string') {
    const selected = document.querySelector(el)
    if (!selected) {
      // 开发环境下给出错误提示
      process.env.NODE_ENV !== 'production' && warn(
        'Cannot find element: ' + el
      )
      // 没有找到的情况下容错处理
      return document.createElement('div')
    }
    return selected
  } else {
    return el
  }
}
```

第二个定义 $mount 函数的地方是src/platforms/web/entry-runtime-with-compiler.js 文件，这个文件是完整版Vue(运行时+编译器)的入口文件。

关于运行时与编译器不清楚的童鞋可以看官网[运行时 + 编译器 vs. 只包含运行时](https://cn.vuejs.org/v2/guide/installation.html#%E8%BF%90%E8%A1%8C%E6%97%B6-%E7%BC%96%E8%AF%91%E5%99%A8-vs-%E5%8F%AA%E5%8C%85%E5%90%AB%E8%BF%90%E8%A1%8C%E6%97%B6)。


```
// 缓存运行时候定义的公共$mount方法
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  // 通过query方法重写el(挂载点: 组件挂载的占位符)
  el = el && query(el)

  /* istanbul ignore if */
  // 提示不能把body/html作为挂载点, 开发环境下给出错误提示
  // 因为挂载点是会被组件模板自身替换点, 显然body/html不能被替换
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }
  // $options是在new Vue(options)时候_init方法内执行.
  // $options可以访问到options的所有属性如data, filter, components, directives等
  const options = this.$options
  // resolve template/el and convert to render function

  // 如果包含render函数则执行跳出,直接执行运行时版本的$mount方法
  if (!options.render) {
    // 没有render函数时候优先考虑template属性
    let template = options.template
    if (template) {
      // template存在且template的类型是字符串
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          // template是ID
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        // template 的类型是元素节点,则使用该元素的 innerHTML 作为模板
        template = template.innerHTML
      } else {
        // 若 template既不是字符串又不是元素节点，那么在开发环境会提示开发者传递的 template 选项无效
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      // 如果template选项不存在，那么使用el元素的outerHTML 作为模板内容
      template = getOuterHTML(el)
    }
    // template: 存储着最终用来生成渲染函数的字符串
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }
      // 获取转换后的render函数与staticRenderFns,并挂在$options上
      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      // 用来统计编译器性能, config是全局配置对象
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  // 调用之前说的公共mount方法
  // 重写$mount方法是为了添加模板编译的功能
  return mount.call(this, el, hydrating)
}
```

关于idToTemplate方法: 通过query获取该ID获取DOM并把该元素的innerHTML 作为模板

```
const idToTemplate = cached(id => {
  const el = query(id)
  return el && el.innerHTML
})

```
getOuterHTML方法:
```
/**
 * Get outerHTML of elements, taking care
 * of SVG elements in IE as well.
 */
function getOuterHTML (el: Element): string {
  if (el.outerHTML) {
    return el.outerHTML
  } else {
    // fix IE9-11 中 SVG 标签元素是没有 innerHTML 和 outerHTML 这两个属性
    const container = document.createElement('div')
    container.appendChild(el.cloneNode(true))
    return container.innerHTML
  }
}
```

关于compileToFunctions函数, 在src/platforms/web/entry-runtime-with-compiler.js中可以看到会挂载到Vue上作为一个全局方法。


![Vue.compile( template )](https://user-gold-cdn.xitu.io/2019/3/11/169687db19baea52?w=2048&h=1084&f=jpeg&s=747414)



### mountComponent方法: 真正执行绑定组件

mountComponent函数中是出现在src/core/instance/lifecycle.js。

```
export function mountComponent (
  vm: Component, // 组件实例vm
  el: ?Element, // 挂载点
  hydrating?: boolean
): Component {
  // 在组件实例对象上添加$el属性
  // $el的值是组件模板根元素的引用
  vm.$el = el
  if (!vm.$options.render) {
    // 渲染函数不存在, 这时将会创建一个空的vnode对象
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  // 触发 beforeMount 生命周期钩子
  callHook(vm, 'beforeMount')

  // vm._render 函数的作用是调用 vm.$options.render 函数并返回生成的虚拟节点(vnode)。template => render => vnode

  // vm._update 函数的作用是把 vm._render 函数生成的虚拟节点渲染成真正的 DOM。 vnode => real dom node

  let updateComponent // 把渲染函数生成的虚拟DOM渲染成真正的DOM
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  // 创建一个Render函数的观察者, 关于watcher后续再讲述.
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```
