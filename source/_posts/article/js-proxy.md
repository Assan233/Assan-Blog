---
title: Proxy 方法详解
date: 2020-05-30 22:48:00
summary: MDN文档摘要
categories: 原创
tags: 
- JavaScript
top: true
cover: true
img:
keywords: 
- JavaScript
- Proxy
---
## MDN定义
**Proxy**对象用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）。
术语
--

**handler**

包含陷阱（trap）的占位符对象，可译为处理器对象。

**traps**

提供属性访问的方法。这类似于操作系统中捕获器的概念，一般直译作陷阱。

**target**

被 Proxy 代理虚拟化的对象。它常被作为代理的存储后端。根据目标验证关于对象不可扩展性或不可配置属性的不变量（保持不变的语义）。
```js
const p = new Proxy(target, handler)
```


**简而言之, handler就是拦截器, target就是被proxy代理的对象, 对target进行的操作, 都会被handle拦截.**


## 实例
```js
const handler ={
get:function(obj, prop){
    return obj.prop
},
set:function(obj, prop, value){
    obj.prop= value
    return true// 返回设置状态
}
}
```

#### 完整的`traps`列表示例



```js
/*
  var docCookies = ... get the "docCookies" object here:  
  https://developer.mozilla.org/zh-CN/docs/DOM/document.cookie#A_little_framework.3A_a_complete_cookies_reader.2Fwriter_with_full_unicode_support
*/

var docCookies = new Proxy(docCookies, {
  "get": function (oTarget, sKey) {
    return oTarget[sKey] || oTarget.getItem(sKey) || undefined;
  },
  "set": function (oTarget, sKey, vValue) {
    if (sKey in oTarget) { return false; }
    return oTarget.setItem(sKey, vValue);
  },
  "deleteProperty": function (oTarget, sKey) {
    if (sKey in oTarget) { return false; }
    return oTarget.removeItem(sKey);
  },
  "enumerate": function (oTarget, sKey) {
    return oTarget.keys();
  },
  "ownKeys": function (oTarget, sKey) {
    return oTarget.keys();
  },
  "has": function (oTarget, sKey) {
    return sKey in oTarget || oTarget.hasItem(sKey);
  },
  "defineProperty": function (oTarget, sKey, oDesc) {
    if (oDesc && "value" in oDesc) { oTarget.setItem(sKey, oDesc.value); }
    return oTarget;
  },
  "getOwnPropertyDescriptor": function (oTarget, sKey) {
    var vValue = oTarget.getItem(sKey);
    return vValue ? {
      "value": vValue,
      "writable": true,
      "enumerable": true,
      "configurable": false
    } : undefined;
  },
});

/* Cookies 测试 */

alert(docCookies.my_cookie1 = "First value");
alert(docCookies.getItem("my_cookie1"));

docCookies.setItem("my_cookie1", "Changed value");
alert(docCookies.my_cookie1);
```


