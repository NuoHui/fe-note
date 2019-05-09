## HTTP缓存

Web缓存我们大致可以分为: 数据库缓存, Redis缓存, 代理服务器缓存, CDN缓存, 浏览器缓存。
那么距离前端最近的就是浏览器缓存。

浏览器缓存又包括: HTTP缓存, Cookie, Web Storage, indexDB等, 我们今天主要学习HTTP缓存。

### HTTP缓存术语

```
1. 缓存命中率: 从缓存中获取数据的请求数 / 总HTTP请求数, 缓存命中率越高, 显然效率越高
2. 过期: 超过了缓存有效时间, 通常过期内容我们需要再次向服务器发送请求, 进行缓存验证
3. 验证: 向服务器验证缓存中过期内容是否依然有效, 验证通过, 服务器重新刷新资源的有效时间
4. 失效: 如果服务器资源更新, 那么缓存中内容即为失效, 需要从缓存中移除
```

虽然HTML的meta标签可以控制缓存

```
<META HTTP-EQUIV="Pragma" CONTENT="no-store">
```
但是代理服务器不解析HTML, 因此我们这里谈的主要指的是HTTP头首部控制缓存


### 与HTTP缓存相关的头部字段

- 通用首部字段

<img width="660" alt="屏幕快照 2019-05-10 上午12 46 02" src="https://user-images.githubusercontent.com/42414989/57471109-03d22580-72bd-11e9-9489-e06b3170f0db.png">

> Pragma

当Pragma值为no-cache时候, 表示客户端不要对该资源进行缓存, 每一次都要向服务器发送请求, 此外**Pragma的优先级是高于Cache-Control的**。

> Cache-Control

由于Expires时间是相对服务器而言，无法保证和客户端时间统一”的问题。http1.1新增了 Cache-Control 来定义缓存过期时间。注意：若报文中同时出现了 Expires 和 Cache-Control，则以 Cache-Control 为准。也就是说优先级从高到低分别是 Pragma -> Cache-Control -> Expires。
Cache-Control也是一个通用首部字段，这意味着它能分别在请求报文和响应报文中使用。

```
"Cache-Control" ":" cache-directive
```
作为请求首部时，cache-directive 的可选值有：

<img width="706" alt="屏幕快照 2019-05-10 上午12 58 07" src="https://user-images.githubusercontent.com/42414989/57471842-b22a9a80-72be-11e9-98be-7edb01a18a0d.png">

作为响应首部时，cache-directive 的可选值有：

<img width="690" alt="屏幕快照 2019-05-10 上午12 58 34" src="https://user-images.githubusercontent.com/42414989/57471872-c373a700-72be-11e9-9f02-f687598a0c84.png">

Cache-Control 允许自由组合可选值，例如：
```
Cache-Control: max-age=3600, must-revalidate
```
它意味着该资源是从原服务器上取得的，且其缓存（新鲜度）的有效时间为一小时，在后续一小时内，用户重新访问该资源则无须发送请求。 当然这种组合的方式也会有些限制，比如 no-cache 就不能和 max-age、min-fresh、max-stale 一起搭配使用。
选择 Cache-Control 的策略（摘自 Google Developers）

![33](https://user-images.githubusercontent.com/42414989/57472810-f454db80-72c0-11e9-8942-7e5e833c899a.jpg)


- 请求首部字段

<img width="658" alt="屏幕快照 2019-05-10 上午12 46 59" src="https://user-images.githubusercontent.com/42414989/57471170-2401e480-72bd-11e9-9c00-a613e3bfbf18.png">

> If-Match: ETag-value

告诉服务器如果没有匹配到ETag，或者收到了“*”值而当前并没有该资源实体，则应当返回412(Precondition Failed) 状态码给客户端。否则服务器直接忽略该字段。
需要注意的是，如果资源是走分布式服务器（比如CDN）存储的情况，需要这些服务器上计算ETag唯一值的算法保持一致，才不会导致明明同一个文件，在服务器A和服务器B上生成的ETag却不一样。

> If-None-Match: ETag-value

示例为 If-None-Match: "5d8c72a5edda8d6a:3239" 告诉服务端如果 ETag 没匹配上需要重发资源数据，否则直接回送304 和响应报头即可。 当前各浏览器均是使用的该请求首部来向服务器传递保存的 ETag 值。

> If-Modified-Since: Last-Modified-value

示例为 If-Modified-Since: Thu, 31 Mar 2016 07:07:52 GMT

该请求首部告诉服务器如果客户端传来的最后修改时间与服务器上的一致，则直接回送304 和响应报头即可。
当前各浏览器均是使用的该请求首部来向服务器传递保存的 Last-Modified 值。

> If-Unmodified-Since: Last-Modified-value

该值告诉服务器，若Last-Modified没有匹配上（资源在服务端的最后更新时间改变了），则应当返回412(Precondition Failed) 状态码给客户端。 Last-Modified 存在一定问题，如果在服务器上，一个资源被修改了，但其实际内容根本没发生改变，会因为Last-Modified时间匹配不上而返回了整个实体给客户端（即使客户端缓存里有个一模一样的资源）。

- 响应首部字段

<img width="666" alt="屏幕快照 2019-05-10 上午12 47 43" src="https://user-images.githubusercontent.com/42414989/57471214-3da32c00-72bd-11e9-9150-7588268b37b6.png">

> ETag

为了解决上述Last-Modified可能存在的不准确的问题，Http1.1还推出了 ETag 实体首部字段。 服务器会通过某种算法，给资源计算得出一个唯一标志符（比如md5标志），在把资源响应给客户端的时候，会在实体首部加上“ETag: 唯一标识符”一起返回给客户端。例如：
```
Etag: "5d8c72a5edda8d6a:3239"
```
客户端会保留该 ETag 字段，并在下一次请求时将其一并带过去给服务器。服务器只需要比较客户端传来的ETag跟自己服务器上该资源的ETag是否一致，就能很好地判断资源相对客户端而言是否被修改过了。
如果服务器发现ETag匹配不上，那么直接以常规GET 200回包形式将新的资源（当然也包括了新的ETag）发给客户端；如果ETag是一致的，则直接返回304知会客户端直接使用本地缓存即可。
那么客户端是如何把标记在资源上的 ETag 传回给服务器的呢？请求报文中有两个首部字段可以带上 ETag 值：

- 实体首部字段

<img width="664" alt="屏幕快照 2019-05-10 上午12 48 26" src="https://user-images.githubusercontent.com/42414989/57471265-5ad7fa80-72bd-11e9-9eaa-0df8e6808b42.png">

> Expires


有了Pragma来禁用缓存，自然也需要有个东西来启用缓存和定义缓存时间，对http1.0而言，Expires就是做这件事的首部字段。 Expires的值对应一个GMT（格林尼治时间），比如Mon, 22 Jul 2002 11:12:01 GMT来告诉浏览器资源缓存过期时间，如果还没过该时间点则不发请求, 如

```
Expires: Fri, 11 Jun 2021 11:33:01 GMT
```
如果Pragma头部和Expires头部同时存在，则起作用的会是Pragma，有兴趣的同学可以自己试一下。
需要注意的是，响应报文中Expires所定义的缓存时间是相对服务器上的时间而言的，其定义的是资源“失效时刻”，如果客户端上的时间跟服务器上的时间不一致（特别是用户修改了自己电脑的系统时间），那缓存时间可能就没啥意义了。


>  Last-Modified

服务器将资源传递给客户端时，会将资源最后更改的时间以“Last-Modified: GMT”的形式加在实体首部上一起返回给客户端。

```
Last-Modified: Fri, 22 Jul 2016 01:47:00 GMT
```

客户端会为资源标记上该信息，下次再次请求时，会把该信息附带在请求报文中一并带给服务器去做检查，若传递的时间值与服务器上该资源最终修改时间是一致的，则说明该资源没有被修改过，直接返回304状态码，内容为空，这样就节省了传输数据量 。如果两个时间不一致，则服务器会发回该资源并返回200状态码，和第一次请求时类似。这样保证不向客户端重复发出资源，也保证当服务器有变化时，客户端能够得到最新的资源。一个304响应比一个静态资源通常小得多，这样就节省了网络带宽。


### 缓存头部对比

<img width="847" alt="屏幕快照 2019-05-10 上午1 07 28" src="https://user-images.githubusercontent.com/42414989/57472434-02eec300-72c0-11e9-927e-e6509906ac43.png">

### HTTP缓存匹配流程

HTTP 头信息控制缓存: 大致分为两种：强缓存和协商缓存。强缓存如果命中缓存不需要和服务器端发生交互，而协商缓存不管是否命中都要和服务器端发生交互，强制缓存的优先级高于协商缓存。

![22](https://user-images.githubusercontent.com/42414989/57472554-48ab8b80-72c0-11e9-963c-98435b7c192b.jpg)


#### 强缓存

可以理解为无须验证的缓存策略。对强缓存来说，响应头中有两个字段 Expires/Cache-Control 来表明规则。

- Expires

Expires 指缓存过期的时间，超过了这个时间点就代表资源过期。有一个问题是由于使用具体时间，如果时间表示出错或者没有转换到正确的时区都可能造成缓存生命周期出错。并且 Expires 是 HTTP/1.0 的标准，现在更倾向于用 HTTP/1.1 中定义的 Cache-Control。两个同时存在时也是 Cache-Control 的优先级更高。

- Cache-Control

Cache-Control 可以由多个字段组合而成。

#### 协商缓存

缓存的资源到期了，并不意味着资源内容发生了改变，如果和服务器上的资源没有差异，实际上没有必要再次请求。客户端和服务器端通过某种验证机制验证当前请求资源是否可以使用缓存。浏览器第一次请求数据之后会将数据和响应头部的缓存标识存储起来。再次请求时会带上存储的头部字段，服务器端验证是否可用。如果返回 304 Not Modified，代表资源没有发生改变可以使用缓存的数据，获取新的过期时间。反之返回 200 就相当于重新请求了一遍资源并替换旧资源。

- Last-modified/If-Modified-Since

Last-modified: 服务器端资源的最后修改时间，响应头部会带上这个标识。第一次请求之后，浏览器记录这个时间，再次请求时，请求头部带上 If-Modified-Since 即为之前记录下的时间。服务器端收到带 If-Modified-Since 的请求后会去和资源的最后修改时间对比。若修改过就返回最新资源，状态码 200，若没有修改过则返回 304。
注意：如果响应头中有 Last-modified 而没有 Expire 或 Cache-Control 时，浏览器会有自己的算法来推算出一个时间缓存该文件多久，不同浏览器得出的时间不一样，所以 Last-modified 要记得配合 Expires/Cache-Control 使用。

<img width="554" alt="屏幕快照 2019-05-10 上午1 12 58" src="https://user-images.githubusercontent.com/42414989/57472731-c7082d80-72c0-11e9-9f56-284e95b7505c.png">


- Etag/If-None-Match

由服务器端上生成的一段 hash 字符串，第一次请求时响应头带上 ETag: abcd，之后的请求中带上 If-None-Match: abcd，服务器检查 ETag，返回 304 或 200。

<img width="455" alt="屏幕快照 2019-05-10 上午1 13 19" src="https://user-images.githubusercontent.com/42414989/57472750-d2f3ef80-72c0-11e9-8298-dffda2cab25e.png">




