# 最佳实践

即看到或者用到的关于Vue.js的最佳使用技巧。

> Object.freeze()

我们都知道Vue.js会对data选项里面的数据进行Observer, 以此来达到数据响应式, 但是很明显这个操作是比较浪费性能的, 如果我们明确不需要state不需要是响应式数据(如展示型的长列表, 或者深层次嵌套的大JS对象), 可以使用Object.freeze()来阻止Vue对其进行Observer。

> 善用计算属性的get/set

<img width="520" alt="屏幕快照 2019-05-03 下午9 29 18" src="https://user-images.githubusercontent.com/42414989/57140527-8acd5c80-6dea-11e9-93b2-50260d7bbd37.png">

> 善用watch

```
var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3,
    d: 4,
    e: {
      f: {
        g: 5
      }
    }
  },
  watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 深度 watcher
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: function (val, oldVal) { /* ... */ },
      immediate: true
    },
    e: [
      function handle1 (val, oldVal) { /* ... */ },
      function handle2 (val, oldVal) { /* ... */ }
    ],
    // watch vm.e.f's value: {g: 5}
    'e.f': function (val, oldVal) { /* ... */ }
  }
})
vm.a = 2 // => new: 2, old: 1
```

> 路由ID变了，但组件没变？


<img width="717" alt="屏幕快照 2019-05-03 下午9 32 09" src="https://user-images.githubusercontent.com/42414989/57140691-ec8dc680-6dea-11e9-981b-569808f2fe30.png">


> 如何在每一个路由后面添加query

```
const transitionTo = router.history.transitionTo

router.history.transitionTo = function (location, onComplete, onAbort) {
  const query = {a: 'xx'}
  location = typeof location === 'object'
    ? {...location, query: {...location.query, ...query}}
    : {path: location, query}

  transitionTo.call(router.history, location, onComplete, onAbort)
}
```

> 事件监听器如何无副作用传递参数


```
@click="someMethod" - 没有参数

@click="someMethod(1, 2, 3)" 没有event对象

@click="event => {someMethod(event, 1, 2, 3)}" - 有参数&event

@click="someMethod($event, 1, 2, 3)" - 更优雅

```

> 不建议在业务代码中使用原生的DOM/BOM API

```
- 无法跨平台
- 不符合设计
```
应该改为使用

```
- 指令
- 插件
```

> 避免隐性的父子组件通信

建议使用prop和事件, 避免使用this.$parent, this.$refs等

> 为组件样式设置作用域

可以使用scoped 特性 或者 CSS Modules。

> Vue官方推荐指南

[风格指南](https://cn.vuejs.org/v2/style-guide/)。


## 架构参考

<img width="723" alt="屏幕快照 2019-05-03 下午9 56 18" src="https://user-images.githubusercontent.com/42414989/57142111-59569000-6dee-11e9-8e16-cc4cd11e68af.png">

### 基础设施层

```
init        =>  自动初始化配置文件  => 脚手架
dev       =>  启动dev-server, hot-reload, http-proxy
deploy  =>  编译源码, 静态文件上传CDN, 生成html, 发布上线
```

### api层

```
请求数据
Mock数据
反向校验数据
```
### Util

全局工具函数集

### service层

整合server端返回的数据(如几个api返回的数据), 然后一起发到Store层。

### 全局事件机制 event-bus

一般只用于处理特殊需求, 如业务存在游客模式与用户模式(登陆), 作为用户模式后的提醒

### View层

```
- router(路由)
- styles
--- site
------ common.scss(存储该App公共的基础色调)
- modules(业务层)
- components(组件)
```

### 关于store层

我们选择抽离Api层。ps: 这一层仅供参考。

首先我们之前是直接在页面里面调用后端的api接口, 后面发现这样实现代码耦合度太高, 于是决定抽离出Api层。

```
// 我们是按照业务模块来分层代码, 下面以用户模块为例子

- modules
--- user
----- api.js(所有的user模块下需要的接口)
----- store.js(在actions下面调用api.js下接口)
----- assets(静态资源)
----- index.js(入口文件)
----- user.vue(业务组件)
```

## 工程化

<img width="521" alt="屏幕快照 2019-05-03 下午10 09 36" src="https://user-images.githubusercontent.com/42414989/57142880-34631c80-6df0-11e9-9acd-022b97dd2a00.png">


## 关于静态文件上传CDN

可以借鉴如下思路:

```
cdn-loader + cdn-plugin 自动上传 cdn
```

<img width="736" alt="屏幕快照 2019-05-03 下午10 20 41" src="https://user-images.githubusercontent.com/42414989/57143578-c0c20f00-6df1-11e9-952c-c06acaf91018.png">
<img width="720" alt="屏幕快照 2019-05-03 下午10 20 47" src="https://user-images.githubusercontent.com/42414989/57143579-c0c20f00-6df1-11e9-83f0-c98874bfacb5.png">


```
var loaderUtils = require('loader-utils')
var qcdn = require('@q/qcdn')

module.exports = function(content) {
  this.cacheable && this.cacheable()
  var query = loaderUtils.getOptions(this) || {}

  if (query.disable) {
    var urlLoader = require('url-loader')
    return urlLoader.call(this, content)
  }

  var callback = this.async()
  var ext = loaderUtils.interpolateName(this, '[ext]', {content: content})

  qcdn.content(content, ext)
    .then(function upload(url) {
      callback(null, 'module.exports = ' + JSON.stringify(url))
    })
    .catch(callback)
}

module.exports.raw = true

```

```
var qcdn = require('@q/qcdn')

function CdnAssetsHtmlWebpackPlugin (options) {}

CdnAssetsHtmlWebpackPlugin.prototype.apply = function (compiler) {
  compiler.plugin('compilation', function(compilation) {
    var extMap = {
      script: {
        ext: 'js',
        src: 'src'
      },
      link: {
        ext: 'css',
        src: 'href'
      },
    }

    compilation.plugin('html-webpack-plugin-alter-asset-tags', function(htmlPluginData, callback) {
      console.log('> Static file uploading cdn...')

      var bodys = htmlPluginData.body.map(upload('body'))
      var heads = htmlPluginData.head.map(upload('head'))

      function upload (type) {
        return function (item, i) {
          if (!extMap[item.tagName]) return Promise.resolve()
          var source = compilation.assets[item.attributes[extMap[item.tagName].src].replace(/^(\/)*/g, '')].source()
          return qcdn.content(source, extMap[item.tagName].ext)
            .then(function qcdnDone(url) {
              htmlPluginData[type][i].attributes[extMap[item.tagName].src] = url
              return url
            })
        }
      }

      Promise.all(heads.concat(bodys))
        .then(function (result) {
          console.log('> Static file upload cdn done!')
          callback(null, htmlPluginData)
        })
        .catch(callback)
    })
  })
}

module.exports = CdnAssetsHtmlWebpackPlugin
```


## 关于权限控制

- 路由权限

路由权限前端主要进行三个步骤处理

```
拦截路由
认证权限(权限判断)
跳转Error页
```
首先权限信息我们一般埋在路由的meta信息里面, 如:

```
const router = new Router({
  mode: 'history',
  routes: [
    {
      path: '/category',
      name: 'category',
      component: Category,
      meta: {
        permission: {
             addImpl: true,
             deleteImpl: true,
             editImpl: true,
             itemImpl: true,
             getCategoryParentsImpl: true,
             listImpl: true,
             searchImpl: true
         }
      }
    }
  ]
})
```
然后在路由的钩子里面进行业务判断

```
router.beforeEach(function (to, from, next) {
  if (permission[to.meta.permission] === false) {
    router.push({name: 'error-authorize'})
    return false
  }
  next()
})
```

- api权限

server端会对api权限做验证，所以前端没做权限认证，直接走全局错误处理逻辑

```
// 请求拦截器家token等

// 在响应的拦截器全局处理
axios.interceptors.response.use(function (response) {
  if (response.data.errno !== 0) {
    Bus.$emit('message', {
      message: response.data.msg || '服务器发生错误',
      type: 'error'
    })
    return Promise.reject(response)
  }
  return response
}, function (error) {
  return Promise.reject(error)
})
```

- 按钮显示权限

使用v-if控制是否显示

```
<el-button
  type="danger"
  size="mini"
  @click="remove(scope.row.id)"
  v-if="permission.category.deleteImpl">
  删除
</el-button>

computed: {
  ...mapState({
    permission: state => state.user.permission
  })
}
```
