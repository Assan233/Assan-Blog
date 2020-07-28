---
title: 实现React-Diff算法
date: 2020-10-12 19:47:55
summary: React 树形Diff算法
categories: 原创
tags: 
- React
- diff
top: true
cover: true
img:
keywords: 
- React
- diff
---

## 实现类似React虚拟Dom的Diff算法, 最后输出dom树差异的层级及key的组合字符串
```javascript
function diff(a, b) {
  let num = 0 // 保存遍历层级
  let str = "" // 输出的差异字符串
  dispatch(a, b)
  function dispatch(a, b) {
    num++
    for (let key in a) {
      const arrOrObj = Array.isArray(a[key]) || Object.prototype.toString.call(a[key]) === "[object Object]"
      if (arrOrObj) {
        dispatch(a[key], b[key])
      } else if (a[key] !== b[key]) {
        str += `${key}${num}-`
      }
    }
  }
  return str
}
```
