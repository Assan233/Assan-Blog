---
title: CSS 笔记 
date: 2020-06-02 17:22:07
summary: CSS 笔记
categories: 笔记
tags: 
- CSS
- CSS 3
top:
cover:
img:
keywords:
- CSS
- CSS 3
---

## 1. :before 和 :after 伪元素注意事项

+ 利用伪元素:before以及:after生成的元素盒子的父元素是利用伪元素这个属性的那个元素盒子。
+ 由伪元素生成的内容并没有脱离文档流。
+ 由伪元素生成的元素盒子它是行内元素，而对于行内元素来说，你懂的，width以及height属性并不能使用。width以及height只适用于块元素以及替换元+ 素。
+ 如果你不给由伪元素生成的元素盒子应用content属性的话，那么这个生成的元素盒子将没有尺寸，毕竟padding以及margin的默认值都是0。
+ 定位参考点的问题，:before伪元素生成的元素盒子的定位参考点是利用伪元素这个属性的那个元素盒子；:after伪元素生成的元素盒子的定位参考点是+ :before伪元素生成的元素盒子。
+ 在补充一下稍微搭边的注意点：如果某个inline元素的position属性的值是absolute或者fixed的话，那么这个inline元素会变成block元素；如果某+ 个inline元素设置了float属性的话，那么这个inline元素会变成block元素。给元素设置了transform属性的话，那么会使得元素具有类似+ position:relative的效果。
   
## 2.  消除编辑器换行所带来的空格问题
  可通过给元素设置 font-size: 0的css样式, 可以消除编辑器换行所带来的空格问题
  
## 3. flex布局的相关属性
**容器上的属性**
+ **flex-direction**: row/column/row-reverse/column-reverse
+ **flex-wrap**: nowrap/wrap/wrap-reverse
+ **flex-flow**:  flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。
+ **justify-content**: flex-start/ flex-end/center/space-between/space-around/space-evently
+ **align-items**: flex-start/flex-end/center/baseline
+ **align-content**: flex-start/flex-end/center/space-between/space-around;

**项目的属性**
+ **order**: Number -order属性定义项目的排列顺序。数值越小，排列越靠前，默认为0;
+ **flex-grow**: Number -flex-grow属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。
+ **flex-shrink**: Number -flex-shrink属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。
+ **flex-basis**: flex-basis属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空+ 间。它的默认值为auto，即项目的本来大小。
+ **flex**: auto(1,1,auto)/ none (0, 0, auto) -flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个+ 属性可选。
+ **align-self**: auto/flex-start/flex-end/center/baseline/stretch.  -align-self属性允许单个项目有与其他项目不一样的对齐方式，可+ 覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。