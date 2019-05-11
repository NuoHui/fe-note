## 什么是BFC

BFC(块格式化上下文): 是Web页面可视化渲染CSS的一部分, 是布局过程中生成块级盒子的区域。也是浮动元素与其他元素的交互限定区域。

简单理解就是具备BFC特性的元素, 就像被一个容器所包裹, 容器内的元素在布局上不会影响外面的元素。

## BFC常见应用

### 解决普通文档流块元素的外边距折叠问题

```
 <style>
    * {
        margin: 0;
        padding: 0;
    }
    .demo div {
        width: 40px;
        height: 40px;
    }
    .demo1 {
        margin: 10px 0;
        background: pink;
    }
    .demo2 {
        margin: 20px 0;
        background: blue;
    }
</style>
<div class="demo">
    <div class="demo1"></div>
    <div class="demo2"></div>
</div>
```

![块元素外边距折叠问题](https://user-gold-cdn.xitu.io/2019/2/21/1690bb2a83eb38c9?w=2504&h=1004&f=png&s=209518)

可见两个块元素外边距为20px。

我们可以使用BFC来解决这个问题,只需要把两个元素置于不同的BFC中进行隔离。

```
<style>
    * {
        margin: 0;
        padding: 0;
    }
    .demo {
        overflow: hidden;
    }
    .demo div {
        width: 40px;
        height: 40px;
    }
    .demo1 {
        margin: 10px 0;
        background: pink;
    }
    .demo2 {
        margin: 20px 0;
        background: blue;
    }
</style>

<div class="demo">
    <div class="demo1"></div>
</div>
<div class="demo">
    <div class="demo2"></div>
</div>
```

![BFC](https://user-gold-cdn.xitu.io/2019/2/21/1690bb63f3eebe99?w=2518&h=972&f=png&s=188623)


### BFC清除浮动

demo演示:
```
<style>
    * {
        margin: 0;
        padding: 0;
    }
    .demo {
        border: 1px solid pink;
    }
    .demo p {
        float: left;
        width: 100px;
        height: 100px;
        background: blue;
    }
</style>

<div class="demo">
    <p></p>
</div>
```
可见容器元素内子元素浮动,脱离文档流,容器元素高度只有2px。

![BFC清除浮动](https://user-gold-cdn.xitu.io/2019/2/21/1690bb99839f6de1?w=2516&h=966&f=png&s=182465)

解决方法:

```
.demo {
    border: 1px solid pink;
    overflow: hidden;
}
```


![BFC清除浮动](https://user-gold-cdn.xitu.io/2019/2/21/1690bbb333737669?w=2514&h=1004&f=png&s=185229)


### 阻止普通文档流元素被浮动元素覆盖

demo演示:
```
<style>
    * {
        margin: 0;
        padding: 0;
    }
    .demo1 {
        width: 100px;
        height: 100px;
        float: left;
        background: pink
    }
    .demo2 {
        width: 200px;
        height: 200px;
        background: blue;
    }
</style>

<div class="demo">
    <div class="demo1">我是一个左侧浮动元素</div>
    <div class="demo2">我是一个没有设置浮动, 也没有触发BFC的元素</div>
</div>
```

demo2部分区域被浮动元素demo1覆盖, 但是文字没有覆盖, 即文字环绕效果。
![元素被浮动元素覆盖](https://user-gold-cdn.xitu.io/2019/2/21/1690bbfccb00357d?w=2522&h=980&f=png&s=240810)

解决办法就是触发demo2的BFC。

```
.demo2 {
    width: 200px;
    height: 200px;
    background: blue;
    overflow: hidden;
}
```


![BFC解决元素被浮动元素覆盖问题](https://user-gold-cdn.xitu.io/2019/2/21/1690bc19fc381ff8?w=2558&h=472&f=png&s=141117)

### 自适应两栏布局


demo演示:

```
<style>
* {
    margin: 0;
    padding: 0;
}
.container {
}
.float {
    width: 200px;
    height: 100px;
    float: left;
    background: red;
    opacity: 0.3;
}

.main {
    background: green;
    height: 100px;
    overflow: hidden;
}
</style>

<div class="container">
    <div class="float">
        浮动元素
    </div>
    <div class="main">
        自适应
    </div>
</div>
```


![BFC实现两栏布局](https://user-gold-cdn.xitu.io/2019/2/21/1690bcadaf036964?w=2238&h=532&f=png&s=104520)

## 如何触发BFC


```
1. 根元素或包含根元素的元素
2. 浮动元素（元素的 float 不是 none）
3. 绝对定位元素（元素的 position 为 absolute 或 fixed）
4. 行内块元素（元素的 display 为 inline-block）
5. 表格单元格（元素的 display为 table-cell，HTML表格单元格默认为该值）
6. 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
7. 匿名表格单元格元素（元素的 display为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 inline-table）
8. overflow 值不为 visible 的块元素
9. display 值为 flow-root 的元素
10. contain 值为 layout、content或 strict 的元素
11. 弹性元素（display为 flex 或 inline-flex元素的直接子元素）
12. 网格元素（display为 grid 或 inline-grid 元素的直接子元素）
13. 多列容器（元素的 column-count 或 column-width 不为 auto，包括 column-count 为 1）
14. column-span 为 all 的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中（标准变更，Chrome bug）。

```
