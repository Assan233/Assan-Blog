---
title: CSS 垂直居中的七种方法
date: 2019-08-19 19:10:21
summary: CSS 垂直居中的七种方法
categories: 转载
tags: CSS
top: true
img:
keywords:
---

> 本文转自微信公众号--前端达人

# 开篇

我之所以整理这类专题的手册，就是CSS相关的内容实在太零散，同时又夹杂着相关的兼容问题。遇到问题时，我们有时过度依赖搜索引擎进行求证解决，解决完也没做认真的归纳和总结。再次遇到此类问题时，我们有可能还不会，这就是我归纳这个手册的目的，我会把日常工作中经常会用到的高频CSS相关方法归纳到这个手册里（有的内容可能来源其它作者），欢迎你持续的订阅和关注。

今天我们一起来梳理下CSS垂直居中的几种方法，我们在布局一个页面时，通常都会用到水平居中和垂直居中，处理水平居中很好处理，不外乎就是设定margin:0 auto;或是text-align:center;,就可以轻松解决掉水平居中的问题，但一直以来最麻烦对齐问题就是「垂直居中」，以下将介绍七种单纯利用CSS垂直居中的方式，其实一点也不难(当然跟水平居中比起来难了一点)，只需要理解背后的原理就可以轻松应用。

本篇文章阅读时间预计5分钟。

## 1. 设定行高( line-height )

设定行高是垂直居中最简单的方式，适用于「单行」的「行内元素」 ( inline、inline-block )，例如单行的标题，或是已经设为inline-block属性的div，若将line-height设成和高度一样的数值，则内容的行内元素就会被垂直置中，因为是行高，所以会在行内元素的上下都加上行高的1/2，所以就垂直置中了！不过由此就可以看出，为什么必须要单行的行内元素，因为如果多行，第二行与第一行的间距会变超大，就不是我们所期望的效果了。CSS范例：外层div0，内容redbox，让redbox水平垂直置中。

``` css
.div0{
    width:200px;
    height:150px;
    border:1px solid #000;
    line-height:150px;
    text-align:center;
}
.redbox{
    display:inline-block;
    width:30px;
    height:30px;
    background:#c00;
}

```

![clipboard.png](/img/bVbwsdG)

## 2. 添加伪元素( ::before、::after )

刚刚第一种方法，虽然是最简单的方法(适用于单行标题)，不过就是只能单行，所以我们如果要让多行的元素也可以垂直居中，就须要使用伪元素的方式。在此之前，先解释一下CSS里头vertical-align这个属性，这个属性虽然是垂直置中，不过却是指在元素内的所有元素垂直位置互相置中，并不是相对于外框的高度垂直居中。(下面的CSS会造成这种样子的垂直居中)


![clipboard.png](/img/bVbwsel)


``` css
.div0{
    width:200px;
    height:150px;
    border:1px solid #000;
    text-align:center;
}
.redbox{
    width:30px;
    height:30px;
    background:#c00;
    display:inline-block;
    vertical-align:middle;
}
.greenbox{
    width:30px;
    height:60px;
    background:#0c0;
    display:inline-block;
    vertical-align:middle;
}
.bluebox{
    width:30px;
    height:40px;
    background:#00f;
    display:inline-block;
    vertical-align:middle;
}
```
因此，如果有一个方块变成了高度100%，那么其他的方块就会真正的垂直居中。


![clipboard.png](/img/bVbwsew)


``` css
.greenbox{
    width:30px;
    height:100%;
    background:#0c0;
    display:inline-block;
    vertical-align:middle;
}
```
但是我们总不能每次要垂直居中，都要添加一个奇怪的div在里头吧！所以我们就要把脑筋动到「伪元素」身上，利用::before和::after添加div进到框框内，让这个「伪」div的高度100%,就可以轻松地让其他的div都居中。不过不过不过！div记得要把display设为inline-block，毕竟 vertical-align:middle 是针对行内元素，div本身是block，所以必须要做更改！


![clipboard.png](/img/bVbwseJ)


``` css
.div0::before{
    content:'';
    width:0;
    height:100%;
    display:inline-block;
    position:relative;
    vertical-align:middle;
    background:#f00;
}
```

## 3. calc 动态计算

看到这边或许会有疑问，如果今天我的div必须是block，我该怎么让它垂直居中呢？这时候就必须用到CSS特有的calc动态计算的能力，我们只要让要居中的div的top属性，与上方的距离是「50%的外框高度- 50%的div高度」，就可以做到垂直居中，至于为什么不用margin-top，因为margin抓到的是水平高度，必须要用top才会正确。

![clipboard.png](/img/bVbwseQ)

``` css
.div0{
    width:200px;
    height:150px;
    border:1px solid #000;
}
.redbox{
    position:relative;
    width:30px;
    height:30px;
    background:#c00;
    float:left;
    top:calc(50% - 15px);
    margin-left:calc(50% - 45px);
}
.greenbox{
    position:relative;
    width:30px;
    height:80px;
    background:#0c0;
    float:left;
    top:calc(50% - 40px);
}
.bluebox{
    position:relative;
    width:30px;
    height:40px;
    background:#00f;
    float:left;
    top:calc(50% - 20px);
}
```

## 4. 使用表格或假装表格

或许有些人会发现，在表格这个HTML里，要实现垂直置中是相当容易的，只需要下一行vertical-align:middle就可以，为什么呢？最主要的原因就在于table的display是table，而td的display是table-cell，所以我们除了直接使用表格之外，也可以将要垂直置中元素的父元素的display改为table-cell，就可以轻松实现，不过修改display有时候也会造成其他样式属性的连动影响，需要小心使用。

**HTML：**

``` html
<table>
    <tr>
        <td>
            <div>表格垂直居中</div>
        </td>
    </tr>
</table>
<div class="like-table">
    <div>假的表格垂直居中</div>
</div>
```

**CSS：**

``` css
.like-table{
    display:table-cell;
}
td,
.like-table{
    width:150px;
    height:100px;
    border:1px solid #000;
    vertical-align: middle;
}
td div,
.like-table div{
    width:100px;
    height:50px;
    margin:0 auto;
    color:#fff;
    font-size:12px;
    line-height: 50px;
    text-align: center;
    background:#c00;
}
.like-table div{
    background:#069;
}
```

![clipboard.png](/img/bVbwseY)

## 5. transform

transform是CSS3的新属性，主要用于元素的变形、旋转和位移，利用transform里头的translateY (改变垂直的位移，如果使用百分比为单位，则是以元素本身的长宽为基准)，搭配元素本身的top属性，就可以做出垂直居中的效果，需要注意的地方是，子元素必须要加上position:relative，不然就会没有效果喔。

``` css
.use-transform{
    width:200px;
    height:200px;
    border:1px solid #000;
}
.use-transform div{
    position: relative;
    width:100px;
    height:50px;
    top:50%;
    transform:translateY(-50%);
    background:#095;
}
```

![clipboard.png](/img/bVbwsfq)




## 6. 绝对定位

绝对定位就是CSS里头的position:absolute，利用绝对位置来指定，但垂直置中的做法又和我们正统的绝对位置不太相同，是要将上下左右的数值都设为0，再搭配一个margin:auto，就可以办到垂直置中，不过要特别注意的是，设定绝对定位的子元素，其父元素的position必须要指定为relative喔！而且绝对定位的元素是会互相覆盖的，所以如果内容元素较多，可能就会有些问题。

``` css
.use-absolute{
    position: relative;
    width:200px;
    height:150px;
    border:1px solid #000;
}
.use-absolute div{
    position: absolute;
    width:100px;
    height:50px;
    top:0;
    right:0;
    bottom:0;
    left:0;
    margin:auto;
    background:#f60;
}
```

![clipboard.png](/img/bVbwsft)

## 7. 使用Flexbox

Flexbox可谓是我们在移动端用的最多的布局方法，因为大部分现代手机浏览器都支持这个方法了。Flexbox，使用align-items或align-content的属性，轻轻松松就可以做到垂直居中的效果喔！

``` css
.use-flexbox{
    display:flex;
    align-items:center;
    justify-content:center;
    width:200px;
    height:150px;
    border:1px solid #000;
}
.use-flexbox div{
    width:100px;
    height:50px;
    background:#099;
}
```

![clipboard.png](/img/bVbwsfy)




「小福利」——由于flexbox布局的属性众多，如何方便记忆，笔者赠送大家一张图：



![clipboard.png](/img/bVbwsfE)




























