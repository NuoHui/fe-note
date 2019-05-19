# 九宫格布局

分析布局特点:

```
1. 九宫格的容器是宽高相等的容器
2. 每个小格子也是宽高相等
```

实现方式也很多, 比如flex, grid, float等, 这里只举一个例子

## 使用flex布局


```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>CSS实现自shiying九宫格(flex布局)</title>
  <style>
    * {
      margin: 0;
      padding: 0;
    }

    li {
      list-style: none;
    }
    /* 实现宽高相等 */
    .square {
      position: relative;
      padding-bottom: 100%; /* padding百分比是相对父元素宽度计算的 */
    }
    /* 撑满父容器 */
    .square-inner {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
    .flex {
      display: flex;
      flex-wrap: wrap;
    }
    .square-inner li {
      width: calc(98% / 3);
      height: calc(98% / 3);
      margin-bottom: 1%;
      margin-right: 1%;
      text-align: center;
      font-size: 50px;
      background: gray;
      color: white;
    }
    .square-inner li:nth-child(3n) {
      margin-right: 0;
    }
    .square-inner li:nth-child(n+7) {
      margin-bottom: 0;
    }
  </style>
</head>

<body>
  <section class="square">
    <ul class="square-inner flex">
      <li>1</li>
      <li>2</li>
      <li>3</li>
      <li>4</li>
      <li>5</li>
      <li>6</li>
      <li>7</li>
      <li>8</li>
      <li>9</li>
    </ul>
  </section>
</body>

</html>

```

[效果图](https://codepen.io/nuohui/pen/rgzVar)
