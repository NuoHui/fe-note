## 什么是Hybrid

Hybrid故名思义: 混合开发, 即App端+前端混合开发模式,Hybrid App 底层依赖 Native 端的 Web 容器（UIWebview 和 WKWebview），上层使用前端 Html5、CSS、Javascript 做业务开发。

## Hybrid带来哪些好处?

一套好的Hybrid App设计能够即拥有原生App流畅的体验, 又拥有前端H5快速灵活的开发模式, 如跨平台能力, 热更新机制。

Hybrid解决的痛点: 我觉得最主要的就是快速版本迭代。

## Hybrid适合的场景

- 需求频繁变更的
- 对交互体验,性能要求不是特别高的


## Webview

我们都知道Webview是一个App端组件, 我们使用WebView 作为容器直接承载 Web页面。

<img width="306" alt="屏幕快照 2019-05-03 上午9 34 11" src="https://user-images.githubusercontent.com/42414989/57116263-a30b9000-6d86-11e9-9222-d52aab4e1182.png">


这里推荐阅读
- [如何设计一个优雅健壮的Android WebView？（上）](https://juejin.im/post/5a94f9d15188257a63113a74)
- [如何设计一个优雅健壮的Android WebView？（下）](https://juejin.im/post/5a94fb046fb9a0635865a2d6)
- [深入剖析 iOS 与 JS 交互](https://zhuanlan.zhihu.com/p/31368159)





## App接入H5的方式

把H5接入到App中有两种方式:

### 在线H5

这是一种非常常见的部署方式, 我们只需要把H5代码单独部署在服务器, 当App通过Webview渲染H5时候, 我们只需要提供对应H5 URL即可, Webview直接load这个URL。

- 优点

```
1. 具备独立部署/开发/调试/上线能力
2. 资源部署在远程服务器, 不会影响App包体积
3. 接入成本低
```

- 缺点

```
1. 依赖于网络, 首屏渲染慢, 需要进一步优化页面性能
2. 不支持离线访问
```
- 适用业务场景

适用于一些功能性不强，不太需要复杂的功能协议，且不需要离线使用的业务。

### App内置包H5

这是一种本地化嵌入方式, 服务端会把我们打包后的前端代码资源下发到App端，然后App端解压到本地。

<img width="1210" alt="ss" src="https://user-images.githubusercontent.com/42414989/57123354-0cee5e80-6db4-11e9-9117-9d17cf5ed709.png">

当然这里还需要一套版本管理机制, 只当App端本地资源不是最新的才去拉去服务器的资源。

<img width="1047" alt="屏幕快照 2019-05-03 下午3 04 17" src="https://user-images.githubusercontent.com/42414989/57123553-cd744200-6db4-11e9-9220-abfb6a7e3dd2.png">


- 优点

```
1. 首屏加载快, 用户体验好
2. 不依赖网络, 支持离线访问
```

- 缺点

```
1. 开发流畅/更新机制复杂, 需要三方协作
2. 会增加App包的体积
```

## Schema协议

### 什么是Schema协议

scheme是一种页面内跳转协议，是一种非常好的实现机制，通过定义自己的scheme协议，可以非常方便跳转app中的各个页面。

###  Scheme链接格式样式

```
[scheme]://[host]/[path]?[query]
```
在前端使用Schema协议

```
<a href="yc://ycbjie:8888/from?type=yangchong">打开叮咚app</a>
```


### Schem协议使用场景

- 通过小程序，利用Scheme协议打开原生app
- H5页面点击锚点，根据锚点具体跳转路径APP端跳转具体的页面
- APP端收到服务器端下发的PUSH通知栏消息，根据消息的点击跳转路径跳转相关页面
- APP根据URL跳转到另外一个APP指定页面
- 通过短信息中的url短链打开原生app

## JS Bridge设计

App与前端的通信方式。

<img width="978" alt="屏幕快照 2019-05-04 下午9 19 00" src="https://user-images.githubusercontent.com/42414989/57179638-75236a00-6eb2-11e9-89d6-6cbad003c87b.png">


### JS Bridge源码

待补充
