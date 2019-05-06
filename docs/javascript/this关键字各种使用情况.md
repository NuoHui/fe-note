this关键字是js中最复杂的机制之一, 因为它是自动被定义在函数作用域中。但是如果我们真正的搞懂了this的工作机制, 其实没有那么复杂。

## 为什么需要使用this?

```
function sayName () {
  console.log(this.name)
}

const xm = {
  name: 'xm'
}

const ml = {
  name: 'ml'
}

sayName.call(xm) // xm
sayName.call(ml) // ml

```
你会发现, 在不同的上下午对象中可以复用同一个函数, 如果不使用this, 我们只能显示指定对象参数。

```
function sayName (ctx) {
  console.log(ctx.name)
}

const xm = {
  name: 'xm'
}

const ml = {
  name: 'ml'
}

sayName(xm) // xm
sayName(ml) // ml

```
可见this会隐式传递一个对象引用, 这样可以使函数的设计可以更简洁优雅。

## 误解之一: this总是指向自身


```
function fo (i) {
  console.log('fo', i)
  console.log(this === window) // true
  this.count++
}

fo.count = 0

for (var i = 0; i < 5; i++) {
  fo(i)
}
console.log(fo.count, count) // 0, NaN

```
针对上述问题我们怎么修改可以获得预期效果?
```
// 回避this的使用
function fo (i) {
  console.log('fo', i)
  console.log(this === window) // true
  fo.count++
}

fo.count = 0

for (var i = 0; i < 5; i++) {
  fo(i)
}
console.log(fo.count) // 5
```

<b>简而言之, this是在运行时候绑定的, 不是由定义时候指定, this的上下文取决于由谁在哪里调用。</b>


## this的绑定规则

> 默认绑定

最常用的函数调用方式就是独立函数调用, 因此我们可以称为默认绑定。
默认绑定this指向window。

```
function a () {
  console.log(this.name)
}
var name = 'a'

a() // 默认不带任何修饰的调用 == 普通调用 输出a
a.call(null) // 输出a
a.call(undefined) // 输出a

```
但是需要注意函数体是严格模式下的默认绑定中this指向undefined。

```
function a () {
  "use strict"
  console.log(this === undefined, this === null) // true, false
  console.log(this.name) // Cannot read property 'name' of undefined
}
var name = 'a'

a()
a.call(null)
a.call(undefined)

```

> 隐式绑定

规则:  有时候我们需要考虑调用位置是否有上下文对象,或者是否被某个对象所包含。

```
// 先定义再添加引用属性

function test () {
  console.log(this.name)
}

var obj = {
  name: 'kobe',
  test: test
}

obj.test() // 'kobe'

```

 ```
// 直接作为对象的方法

var obj = {
  name: 'kobe',
  test: function () {
    console.log(this.name)
  }
}

obj.test() // 'kobe'

```

箭头函数的this是指向它的父级执行环境
```
// 注意箭头函数的this指向
var name = 'global'
var obj = {
  name: 'kobe',
  test: () => {
    console.log(this === window) // true
    console.log(this.name) // global
  }
}

obj.test()

```

```
// 对象属性引用链的调用(只有最顶层与最后一层才会影响调用位置)

function foo () {
  console.log(this.name)
}
var c = {
  name: 'c',
  foo: foo
}
var b = {
  name: 'b',
  c: c
}

var a = {
  name: 'a',
  b: b
}

a.b.c.foo() // c

```

> 隐式丢失

一个常见的this绑定出现的问题就是: 被隐式绑定的函数容易丢失绑定对象, 最后应用默认绑定。

```
function a () {
  console.log(this.name)
}

var obj = {
  name: 'obj',
  a: a
}

var temp = obj.a
var name = 'global'
temp() // this绑定丢失, 应用默认绑定, 指向全局window  => 'global'
obj.a() // 'obj'

```

另外需要注意参数传递也是一种隐式赋值。

```
function a () {
  console.log(this.name)
}

var obj = {
  name: 'obj',
  a: a
}

var name = 'global'

function fx (cb) {
  cb()
}

fx(obj.a) // this绑定丢失, 应用默认绑定, 指向全局window  => 'global'

/**
 * fx(obj.a)  === 等于  var fn = obj.a; fx(fn)
*/

```
如果你把函数传入语言内置的函数, 结果其实也是一样。

```
var obj = {
  name: 'obj',
  a: function () {
    console.log(this.name)
  }
}
var name = 'global'
setTimeout(obj.a, 10) // 传参是隐式赋值

```

> 显示绑定

显示绑定主要通过call(), apply()

call()与apply()方法第一个参数指的是绑定对象,即它们会把对象绑定到this, 然后调用函数时候指向这个this, 因此通过直接指定this的绑定对象方法叫做显示绑定。

```
function test () {
  console.log(this.name)
}

var o = {
  name: 'o'
}
test.call(o) // o

```

如果你传入的第一个参数是原始值, 来作为this绑定对象, 那么该原始值会通过new String(), new Boolean(), new Number()转为对象形式, 这个操作又叫做装箱。


> 硬绑定

显示绑定依然无法解决this绑定丢失问题, 但是我们可以通过一些方法来解决该问题。

- 显示绑定变种实现

```
function test () {
  console.log(this.name)
}

var o = {
  name: 'o',
  test: test
};
// 通过创建一个包裹函数
function bindThis () {
  test.call(o)
}
var name = 'global'
setTimeout(bindThis) // o

```

通过bind()实现硬绑定, 来解决丢失this绑定问题。
bind()会返回一个硬编码的新函数, 并且把参数设置为this的上下文。

```
function test () {
  console.log(this.name)
}

var o = {
  name: 'o',
  test: test
};

var name = 'global'
setTimeout(o.test.bind(o)) // o

```

此外很多第三方库或者JS内置函数都允许传递一个可选参数作为上下文对象。
其作用就是类似bind, 确保你的回调函数中this指向的是该上下文对象。

```
function test (item) {
  console.log(item, this.id)
}

var obj = {
  id: 'id'
};

[1,2,3,4].forEach(test, obj)

```

> new 绑定

我们一般创建一个对象实例通过new Constructor()

new 一个构造函数其实会执行以下过程。
```
1. 创建一个全新的对象
2. 该新对象执行原型链接, 把该对象绑定到this
3. 执行函数中代码
4. 如果没有其他返回, 默认返回该新对象
```

```
function Person () {
  this.name = '2'
}

console.log(new Person().name) // 2

```

> 绑定规则优先级


```
1. 默认绑定优先级最低
```

比较显示绑定与隐式绑定: 显示绑定优先于隐式绑定

```
function test () {
  console.log(this.name)
}

var a = {
  name: 'a',
  test: test
}

var b = {
  name: 'b',
  test: test
}

// 隐式绑定
a.test() // a
b.test() // b

// 显示绑定
a.test.call(b) // b
b.test.call(a) // a

```

比较new绑定与隐式绑定: new绑定优先级高于隐式绑定

```
function test (thing) {
  this.thing = thing
}

var obj1 = {
  test: test
}

var obj2 = {}

// 隐式绑定
obj1.test(2)
console.log(obj1.thing) // 2

// 显示绑定
obj1.test.call(obj2, 3)
console.log(obj2.thing) // 3


// 可见new绑定优先级高于隐式绑定, 如果不是的化 test.thing=2
var test = new obj1.test(4)
console.log(obj1.thing) // 2
console.log(test.thing) // 4

```

那么最后我们比较new绑定跟硬绑定: new绑定优先与硬绑定
注意: new绑定不能跟call, apply一起使用。


```
function test (thing) {
  this.thing = thing
}

var o = {}

var binds = test.bind(o)
binds(3)
// 硬绑定
console.log(o.thing) // 3

var thing = new binds(4)
console.log(thing.thing) // 4
console.log(o.thing) // 3, 么有改变说明 new绑定优先级高

```

总结:

```
1. 如果函数是通过new调用(new绑定), 那么this指向新创建的对象
2. 函数是否通过call, apply, bind等显示绑定,或者硬绑定, 是的化this指向指定的对象
3. 函数是在某个上下文中调用(隐式绑定), 那么this绑定的是那个上下文对象
4. 如果以上都不是, 那么默认是默认绑定, this指向全局对象, 如果函数体是严格模式指向undefined

```


## 特殊情况下的绑定

- 被忽略的this

即就是bind, call, apply时候, 把undefined或者null作为绑定对象。

```
var a = 2
function test () {
  console.log(this.a)
}
test.call(null) // 2
test.call(undefined) // 2

```

```
function test (a, b) {
  console.log(a, b)
}

test.apply(null, [3,4]) // 3,4
var temp = test.bind(null, 2)
temp(4) // 2, 4

```
- 更安全的this


但是很多时候我们不建议把null或者undefined作为空对象的占位符,因为它们默认会指向全局对象, 容易产生副作用,  最好使用Object.create(null)。

```
function test (a, b) {
  console.log(a, b)
}

test.apply(Object.create(null), [3,4]) // 3,4
var temp = test.bind(Object.create(null), 2)
temp(4) // 2, 4

```
- 间接引用

有时候我们可能会无意创建这个函数是的间接引用, 这种情况下, 调用这个函数默认会使用默认规则。

```
// 间接引用最容易在赋值时候发生

var name = 'global'
function test () {
  console.log(this.name)
}

var o = {
  name: 'o',
  test: test
}
var a = {
  name: 'a'
};
o.test();// o
(a.test = o.test)() // global

// 如果是
// a.test = o.test
// a.test() // a
```

## 面试题

```
var name = 'global'

var obj = {
  name: 'obj',
  test: function () {
    (() => {
      var name = '2'
      console.log(this.name)
    })()
  }
}

obj.test() // obj

```

```
function foo () {
  return () => {
    console.log(this.name)
  }
}

var obj1 = {
  name: 'obj1'
}

var obj2 = {
  name: 'obj2'
}

var temp = foo.call(obj1)
temp.call(obj2) // obj1

```
