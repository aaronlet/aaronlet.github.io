---
layout:     post
title:      弹性盒模型布局入门
subtitle:   Introduction to elastic box model layout
date:       2019-02-26
author:     Aaron
header-img: img/blog/post-bg-elastic-box.jpg
catalog: true
tags:
    - Css
    - 弹性和模型
    - 函数式编程
    - 开源框架
---
## 弹性盒模型布局入门

可能大家在平时工作中都会用到一些框架比如`Bootstrap`，`iview`，`Element-ui`等，这些UI框架都会有栅格系统，栅格系统已经能很好的满足业务需求，所以可能对css3的弹性布局不是很感冒，有的用就行了，但是若不使用任何框架完成栅格怎么办，岂不是麻爪了，学习一下弹性盒模型，了解其基础用法，以及深入。

### 什么是弹性盒模型

采用Flex布局的元素，称为Flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。

使用弹性盒模型要给容器添加`dispaly:flex;`属性。

![](https://llp0574.github.io/img/flex-2.gif)

对Css熟悉的童鞋应该对盒模型都又一定的了解，弹性盒模型其本质上就是Box-model（盒子模型）的延伸，Box-model（盒子模型）定义了一个元素的盒模型，而Flexbox（弹性盒子）更进一步的去规范了这些盒模型之间彼此的相对关系。

弹性盒模型按照宽高分出了以下8点：

1. 水平的主轴：main axis，垂直的纵轴：cross axis
2. 纵轴的开始位置和边框的交点：cross start
3. 纵轴的结束位置和边框的交点：cross end
4. 主轴的开始位置和边框的交点：main strat
5. 主轴的结束位置和边框的交点：main end
6. 单个项目占据主轴的空间距离：main axis
7. 单个项目占据纵轴的空间距离：cross axis

![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071004.png)

### 容器属性

1. flex-direction
2. flex-wrap
3. flex-flow
4. justify-content
5. align-items
6. align-content

> flex-direction

flex-direction属性决定主轴的方向（即项目的排列方向）。

flex-direction属性共有4个可选值

1. row（默认值）：以main start点为起始点，按照元素顺序**从左到右**依次排列。
2. row-reverse：以main end为起始点，按元素顺序从**右到左依**次排列。
3. column：以cross start为起始点，按元素顺序从**上到下依**次排列。
4. column-reverse：以cross end为起始点，按元素顺序从**下到上依**次排列。

![](https://llp0574.github.io/img/flex-4.gif)

这里有一个很重要的区别：flex-direction:column并不是把块从主轴移到交叉轴上排列，而是让主轴自身从水平变成垂直。

另外 flex-direction 还有两个选项值：row-reverse 和 column-reverse。

![](https://llp0574.github.io/img/flex-5.gif)

> flex-wrap

默认情况下，项目都排在一条线（又称"轴线"）上。flex-wrap属性定义，如果一条轴线排不下，如何换行。

flex-wrap属性共有3个可选值

1. nowrap（默认值）：不换行。
2. wrap：换行，第一行在上方，垂直纵轴以cross start为起始点，水平主轴以main start为起始点。
3. wrap-reverse：换行，第一行在下方，垂直纵轴以cross end为起始点，水平主轴以main start为起始点。

> flex-flow

flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。

```css
.box {
  flex-flow: <flex-direction> || <flex-wrap>;
}
```

这个很好理解，按照对应的可选属性，填写相对应的值即可。

例如：如果想让其子元素以`main end`为起始点并且换行的话。

```css
.box {
    display:flex;
    flex-flow: row-reverse wrap;
}
```

> justify-content

这定义了沿主轴的对齐。当主轴上的所有弹性项目都不灵活，或者灵活但已达到其最大尺寸时，它有助于分配剩余的额外空闲空间。当它们溢出线时，它还对物品的对齐施加一些控制。

justify-content属性共有5个可选值，具体对齐方式与轴的方向有关。下面假设主轴为从左到右。

1. flex-start（默认值）：左对齐
2. flex-end：右对齐
3. center： 居中
4. space-between：两端对齐，项目之间的间隔都相等。
5. space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。
 

![](https://llp0574.github.io/img/flex-6.gif)

> align-items

align-items属性定义项目在交叉轴上如何对齐。

align-items属性共有5个可选值。具体的对齐方式与交叉轴的方向有关，下面假设交叉轴从上到下。

1. flex-start：交叉轴的起点对齐。
2. flex-end：交叉轴的终点对齐。
3. center：交叉轴的中点对齐。
4. baseline: 项目的第一行文字的基线对齐。
5. stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

![](https://llp0574.github.io/img/flex-8.gif)

文字基线

![](http://images2015.cnblogs.com/blog/446475/201604/446475-20160411221120457-1802709993.png)

为了更清楚地阐明主轴和交叉轴的区别，下面我们来把 justify-content 和 align-items 合在一起，看看在 flex-direction 两种值的作用下轴心有什么不一样：

![](https://llp0574.github.io/img/flex-10.gif)

> align-content

align-content属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

align-content属性共有6个可选值。

1. flex-start：与交叉轴的起点对齐。
2. flex-end：与交叉轴的终点对齐。
3. center：与交叉轴的中点对齐。
4. space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
5. space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
6. stretch（默认值）：轴线占满整个交叉轴。

### 元素属性

1. order
2. flex-grow
3. flex-shrink
4. flex-basis
5. flex
6. align-self

> order

order属性定义元素的排列顺序。数值越小，排列越靠前，默认为0，可以使用负值。如果两个元素的值相同，按照元素顺序排列。

```css
.item {
  order: Number <integer>;
}
```

> flex-grow

flex-grow属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。

```css
.item {
  flex-grow: Number;
}
```
如果所有项目的flex-grow属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的flex-grow属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。

> flex-shrink

flex-shrink属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

```css
.item {
  flex-shrink: Number;
}
```

如果所有项目的flex-shrink属性都为1，当空间不足时，都将等比例缩小。如果一个项目的flex-shrink属性为0，其他项目都为1，则空间不足时，前者不缩小。

> flex-basis

flex-basis属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。

```css
.container {
    display:flex;
    margin:50px auto 0px;
    border: 1px solid red;
}
.container div {
    width:100px;
    height:100px;
    background:pink;
    margin:10px;
}
.container div:nth-child(1){
    flex-basis:500px;
}
```

> flex

flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。

```css
.item {
  flex:0 1 auto;
}
```

示例

```css
.container {
    display:flex;
    margin:50px auto 0px;
    border: 1px solid red;
}
.container div {
    width:100px;
    height:100px;
    background:pink;
    margin:10px;
}
.container div:nth-child(1){
    flex:0 1 500px;
    /* flex:1 0 500px; */
}
```

> align-self

align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。

示例

```css
.container {
    display:flex;
    margin:50px auto 0px;
    border: 1px solid red;
    height:500px;
}
.container div {
    width:100px;
    height:100px;
    background:pink;
    margin:10px;
}
.container div:nth-child(1){
    flex:0 1 500px;
    align-self:flex-end;
}
.container div:nth-child(2){
    flex:0 1 500px;
    align-self:center;
}
```

下面将给两个块设置 align-self 属性，其余的使用 align-items: center 和 flex-direction: row，来看看会是什么效果：

![](https://llp0574.github.io/img/flex-11.gif)

文章参考

- [阮一峰老师-Flex布局：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
- [CSS-TRICKS](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)



