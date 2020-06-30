---
title: Vue 2.60 中, 插槽新增的v-slot
date: 2020-06-30 22:32:36
summary: 官方文档摘要
categories: 原创
tags: 
- Vue
- slot
top: true
cover: true
img:
keywords: 
- Vue
- slot
---
**注意`v-slot`只能添加在`<template>`上(只有[一种例外情况](https://cn.vuejs.org/v2/guide/components-slots.html#%E7%8B%AC%E5%8D%A0%E9%BB%98%E8%AE%A4%E6%8F%92%E6%A7%BD%E7%9A%84%E7%BC%A9%E5%86%99%E8%AF%AD%E6%B3%95))，这一点和已经废弃的[`slot`attribute](https://cn.vuejs.org/v2/guide/components-slots.html#%E5%BA%9F%E5%BC%83%E4%BA%86%E7%9A%84%E8%AF%AD%E6%B3%95)不同。**

[作用域插槽](https://cn.vuejs.org/v2/guide/components-slots.html#%E4%BD%9C%E7%94%A8%E5%9F%9F%E6%8F%92%E6%A7%BD "作用域插槽")
------------------------------------------------------------------------------------------------------------------

> 自 2.6.0 起有所更新。已废弃的使用`slot-scope`attribute 的语法在[这里](https://cn.vuejs.org/v2/guide/components-slots.html#%E5%BA%9F%E5%BC%83%E4%BA%86%E7%9A%84%E8%AF%AD%E6%B3%95)。

有时让插槽内容能够访问子组件中才有的数据是很有用的。例如，设想一个带有如下模板的`<current-user>`组件：

```html
<span>
  <slot>{{ user.lastName }}</slot>
</span>
```

我们可能想换掉备用内容，用名而非姓来显示。如下：

```html
<current-user>
  {{ user.firstName }}
</current-user>
```

然而上述代码不会正常工作，因为只有`<current-user>`组件可以访问到`user`而我们提供的内容是在父级渲染的。

为了让`user`在父级的插槽内容中可用，我们可以将`user`作为`<slot>`元素的一个 attribute 绑定上去：

```html
<span>
  <slot v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```

绑定在`<slot>`元素上的 attribute 被称为**插槽 prop**。现在在父级作用域中，我们可以使用带值的`v-slot`来定义我们提供的插槽 prop 的名字：

```html
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```

在这个例子中，我们选择将包含所有插槽 prop 的对象命名为`slotProps`，但你也可以使用任意你喜欢的名字。
