---
title: D3.event 在经过Babel转译之后出现的问题
date: 2020-06-02 14:07:28
img:
top: true
keywords: d3.event
categories: 原创
tags: 
- D3.js
- JavaScript
summary: 使用 Babel, Webpack 或者其他的 ES6 转 ES5 的打包工具，要注意 d3.event 的值在事件中的变化！
---

## d3.event

**官方文档说明**

如果你使用 Babel, Webpack 或者其他的 ES6 转 ES5 的打包工具，要注意 d3.event 的值在事件中的变化！导入的 d3.event 必须是 live binding(动态绑定) 的，因此你需要将打包配置设置为引入 D3 的 ES6 模块而不是生成的 UMD；并不是所有的打包工具都识别 jsnext:main。也要注意与 window.event 的冲突。

## 解决方法
文档说的动态绑定就是在事件处理函数里面手动引入d3.event, 而不是在全局引入, 如下:
``` javascript
    function dragged() {
        const { event } = require("d3-selection");//动态引入event
        self.view
          .select(".output")
          .attr("transform", `translate(0, ${event.y - self.y0 + self.viewY})`);
    }
```
