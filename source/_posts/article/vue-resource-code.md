---
title: Vue(v2.6.11)万行源码生啃
date: 2020-07-11 15:58:13
summary: 源码阅读可能会迟到，但是一定不会缺席！
categories: 转载
tags: Vue
top: true
cover: true
img:
keywords: Vue
---
前言
--

源码阅读可能会迟到，但是一定不会缺席！

众所周知，以下代码就是 vue 的一种直接上手方式。通过 cdn 可以在线打开 vue.js。一个文件，一万行源码，是万千开发者赖以生存的利器，它究竟做了什么？让人品味。

```javascript
<html>
<head></head>
<body>
    <div id="app">
        {{ message }}
    </div>
</body>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    var app = new Vue({
        el: '#app',
        data: {
            message: 'See Vue again!'
        },
    })
</script>
</html>

```

源码cdn地址：[cdn.jsdelivr.net/npm/vue/dis…](https://cdn.jsdelivr.net/npm/vue/dist/vue.js)，当下版本：v2.6.11。

本瓜选择生啃的原因是，可以更自主地选择代码段分轻重来阅读，一方面测试自己的掌握程度，一方面追求更直观的源码阅读。

当然你也可以选择在 [github.com/vuejs/vue/t…](https://github.com/vuejs/vue/tree/dev/src) 分模块的阅读，也可以看各路大神的归类整理。

其实由于本次任务量并不算小，为了能坚持下来，本瓜将源码尽量按 500 行作为一个模块来形成一个 md 文件记录（[分解版本共 24 篇感兴趣可移步](https://tuaran.github.io/views/2020/studyVuesc1.html)），结合注释、自己的理解、以及附上对应查询链接来逐行细读源码，**此篇为合并版本**。

**目的：自我梳理，分享交流。**

最佳阅读方式推荐：先点赞👍再阅读📖，靴靴靴靴😁

正文
--

### 第 1 行至第 10 行

// init

```javascript
(
    function (global, factory) {
        typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() : typeof define === 'function' && define.amd ? define(factory) : (global = global || self, global.Vue = factory());
    }(
        this,
        function () {
            'use strict';
            //...核心代码...
        }
    )
);
// 变形
if (typeof exports === 'object' && typeof module !== 'undefined') { // 检查 CommonJS
    module.exports = factory()
} else {
    if (typeof define === 'function' && define.amd) { // AMD 异步模块定义 检查JavaScript依赖管理库 require.js 的存在 [link](https://stackoverflow.com/questions/30953589/what-is-typeof-define-function-defineamd-used-for)
        define(factory)
    } else {
        (global = global || self, global.Vue = factory());
    }
}
// 等价于
window.Vue=factory() 
// factory 是个匿名函数,该匿名函数并没自执行 设计参数 window，并传入window对象。不污染全局变量，也不会被别的代码污染

```

### 第 11 行至第 111 行

// 工具代码

```javascript
var emptyObject = Object.freeze({});// 冻结的对象无法再更改 [link](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)

```

// 接下来是一些封装用来判断基本类型、引用类型、类型转换的方法

*   isUndef//判断未定义
    
*   isDef// 判断已定义
    
*   isTrue// 判断为 true
    
*   isFalse// 判断为 false
    
*   isPrimitive// 判断为原始类型
    
*   isObject// 判断为 obj
    
*   toRawType // 切割引用类型得到后面的基本类型，例如：\[object RegExp\] 得到的就是 RegExp
    
*   isPlainObject// 判断纯粹的对象："纯粹的对象"，就是通过 { }、new Object()、Object.create(null) 创建的对象
    
*   isRegExp// 判断原生引用类型
    
*   isValidArrayIndex// 检查val是否是一个有效的数组索引，其实就是验证是否是一个非无穷大的正整数
    
*   isPromise// 判断是否是 Promise
    
*   toString// 类型转成 String
    
*   toNumber// 类型转成 Number
    

### 第 113 行至第 354 行

*   makeMap// makeMap 方法将字符串切割，放到map中，用于校验其中的某个字符串是否存在（区分大小写）于map中 e.g.

```javascript
var isBuiltInTag = makeMap('slot,component', true);// 是否为内置标签
isBuiltInTag('slot'); //true
isBuiltInTag('slot1'); //undefined
var isReservedAttribute = makeMap('key,ref,slot,slot-scope,is');// 是否为保留属性

```

*   remove// 数组移除元素方法
    
*   hasOwn// 判断对象是否含有某个属性
    
*   cached// ※高级函数 cached函数，输入参数为函数，返回值为函数。同时使用了闭包，其会将该传入的函数的运行结果缓存，创建一个cache对象用于缓存运行fn的运行结果。[link](http://u-to-world.com:8081/?p=287)
    

```javascript
function cached(fn) {
    var cache = Object.create(null);// 创建一个空对象
    return (function cachedFn(str) {// 获取缓存对象str属性的值，如果该值存在，直接返回，不存在调用一次fn，然后将结果存放到缓存对象中
        var hit = cache[str];
        return hit || (cache[str] = fn(str))
    })
}

```

*   camelize// 驼峰化一个连字符连接的字符串
    
*   capitalize// 对一个字符串首字母大写
    
*   hyphenateRE// 用字符号连接一个驼峰的字符串
    
*   polyfillBind// ※高级函数 参考[link](http://u-to-world.com:8081/?p=287)
    
*   Function.prototype.bind() // [link1](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind##Polyfill)[link2](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
    
*   toArray// 将[像数组的](https://github.com/lessfish/underscore-analysis/issues/14)转为真数组
    
*   extend// 将多个属性插入目标的对象
    
*   toObject// 将对象数组合并为单个对象。
    

e.g.

```javascript
console.log(toObject(["bilibli"]))
//{0: "b", 1: "i", 2: "l", 3: "i", 4: "b", 5: "l", 6: "i", encodeHTML: ƒ}

```

*   no// 任何情况都返回false
    
*   identity // 返回自身
    
*   genStaticKeys// 从编译器模块生成包含静态键的字符串。TODO:demo
    
*   looseEqual//※高级函数 对对象的浅相等进行判断
    

//有赞、头条面试题

```javascript
function looseEqual(a, b) {
    if (a === b) return true
    const isObjectA = isObject(a)
    const isObjectB = isObject(b)
    if(isObjectA && isObjectB) {
        try {
            const isArrayA = Array.isArray(a)
            const isArrayB = Array.isArray(b)
            if(isArrayA && isArrayB) {
                return a.length === b.length && a.every((e, i) => {
                    return looseEqual(e, b[i])
                })
            }else if(!isArrayA && !isArrayB) {
                const keysA = Object.keys(a)
                const keysB = Object.keys(b)
                return keysA.length === keysB.length && keysA.every(function (key) {
                    return looseEqual(a[key], b[key])
                })
            }else {
                return false
            }
        } catch(e) {
            return false
        }
    }else if(!isObjectA && !isObjectB) {
        return String(a) === String(b)
    }else {
        return false
    }
}

```

*   looseIndexOf// 返回索引，如果没找到返回-1，否则执行looseEqual()
*   once// 确保函数只被调用一次，用到闭包

### 阶段小结

*   cached
*   polyfillBind
*   looseEqual

这三个函数要重点细品！主要的点是：闭包、类型判断，函数之间的互相调用。也即是这部分工具函数的精华！

### 第 356 行 至 第 612 行

// 定义常量和配置

*   SSR\_ATTR// 服务端渲染
*   ASSET\_TYPES// 全局函数 component、directive、filter
*   LIFECYCLE\_HOOKS// 生命周期，无需多言
*   config // 全局配置 [link](https://www.cnblogs.com/greatdesert/p/11011015.html)
*   unicodeRegExp//用于解析html标记、组件名称和属性pat的unicode字母
*   isReserved// 检查变量的开头是 $ 或 \_
*   def// 在一个对象上定义一个属性的构造函数，其中 !!enumerable 强制转换 boolean
*   parsePath// 解析一个简单路径 TODO:
*   userAgent// 浏览器识别
*   inBrowser
*   \_isServer//检测 vue的服务器渲染是否存在, 而且避免webpack去填充process
*   isNative //这里判断 函数是否是系统函数, 比如 Function Object ExpReg window document 等等, 这些函数应该使用c/c++实现的。这样可以区分 Symbol是系统函数, 还是用户自定义了一个Symbol
*   hasSymbol//这里使用了ES6的Reflect方法, 使用这个对象的目的是, 为了保证访问的是系统的原型方法, ownKeys 保证key的输出顺序, 先数组 后字符串
*   \_Set// 设置一个Set

[link](https://www.cnblogs.com/dhsz/p/7064913.html)

### 第 616 行至第 706 行

//设置warn,tip等全局变量 TODO:

*   warn
*   tip
*   generateComponentTrace// 生成组件跟踪路径（组件数规则）
*   formatComponentName// 格式化组件名

### 第 710 行至第 763 行

**Vue核心：数据监听最重要之一的 Dep**

```javascript
// Dep是订阅者Watcher对应的数据依赖
var Dep = function Dep () {
  //每个Dep都有唯一的ID
  this.id = uid++;
  //subs用于存放依赖
  this.subs = [];
};

//向subs数组添加依赖
Dep.prototype.addSub = function addSub (sub) {
  this.subs.push(sub);
};
//移除依赖
Dep.prototype.removeSub = function removeSub (sub) {
  remove(this.subs, sub);
};
//设置某个Watcher的依赖
//这里添加了Dep.target是否存在的判断，目的是判断是不是Watcher的构造函数调用
//也就是说判断他是Watcher的this.get调用的，而不是普通调用
Dep.prototype.depend = function depend () {
  if (Dep.target) {
    Dep.target.addDep(this);
  }
};

Dep.prototype.notify = function notify () {
  var subs = this.subs.slice();
  //通知所有绑定 Watcher。调用watcher的update()
  for (var i = 0, l = subs.length; i &lt; l; i++) {
    subs[i].update();
  }
};

```

强烈推荐阅读：[link](https://www.cnblogs.com/datiangou/p/10144883.html)

Dep 相当于把 Observe 监听到的信号做一个收集（collect dependencies），然后通过dep.notify()再通知到对应 Watcher ，从而进行视图更新。

### 第 767 行至第 900 行

**Vue核心：视图更新最重要的 VNode（ Virtual DOM）**

*   VNode
*   createEmptyVNode
*   createTextVNode
*   cloneVNode

把你的 template 模板 描述成 VNode，然后一系列操作之后通过 VNode 形成真实DOM进行挂载

更新的时候对比旧的VNode和新的VNode，只更新有变化的那一部分，提高视图更新速度。

e.g.

```javascript
<div class="parent" style="height:0" href="2222">
    111111
</div>

//转成Vnode
{    

    tag: 'div',    

    data: {        

        attrs:{href:"2222"}

        staticClass: "parent",        

        staticStyle: {            

            height: "0"

        }
    },    

    children: [{        

        tag: undefined,        

        text: "111111"

    }]
}

```

强烈推荐阅读：[link](https://cloud.tencent.com/developer/article/1479295)

*   methodsToPatch

将数组的基本操作方法拓展，实现响应式，视图更新。

因为：对于对象的修改是可以直接触发响应式的，但是对数组直接赋值，是无法触发的，但是用到这里经过改造的方法。我们可以明显的看到 ob.dep.notify() 这一核心。

### 阶段小结

这一 part 最重要的，毋庸置疑是：Dep 和 VNode，需重点突破！！！

### 第 904 行至第 1073 行

**Vue核心：数据监听最重要之一的 Observer**

*   核心的核心！Observer（发布者） => Dep（订阅器） => Watcher（订阅者）

类比一个生活场景：报社将各种时下热点的新闻收集，然后制成各类报刊，发送到每家门口的邮箱里，订阅报刊人们看到了新闻，对新闻作出评论。

在这个场景里，报社==发布者，新闻==数据，邮箱==订阅器，订阅报刊的人==订阅者，对新闻评论==视图更新

*   Observer//Observer的调用过程：initState()-->observe(data)-->new Observer()

```javascript
var Observer = function Observer (value) {
  this.value = value;
  this.dep = new Dep();
  this.vmCount = 0;
  def(value, '__ob__', this);
  if (Array.isArray(value)) {
    if (hasProto) {
      protoAugment(value, arrayMethods);
    } else {
      copyAugment(value, arrayMethods, arrayKeys);
    }
    this.observeArray(value);
  } else {
    this.walk(value);
  }
};

```

*   ※※ defineReactive 函数，定义一个响应式对象，给对象动态添加 getter 和 setter ，用于依赖收集和派发更新。

```javascript
function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()// 1. 为属性创建一个发布者

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get // 依赖收集
  const setter = property && property.set // 派发更新
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  let childOb = !shallow && observe(val)// 2. 获取属性值的__ob__属性
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()// 3. 添加 Dep
        if (childOb) {
          childOb.dep.depend()//4. 也为属性值添加同样的 Dep 
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}

```

第 4 步非常重要。为对象的属性添加 dep.depend(),达到监听对象（引用的值）属性的目的

### 重点备注

Vue对数组的处理跟对象还是有挺大的不同，length是数组的一个很重要的属性，无论数组增加元素或者删除元素（通过splice，push等方法操作）length的值必定会更新，为什么不直接操作监听length呢？而需要拦截splice，push等方法进行数组的状态更新？

原因是：在数组length属性上用defineProperty拦截的时候，会报错。

```javascript
Uncaught TypeError: Cannot redefine property: length

```

再用Object.getOwnPropertyDescriptor(arr, 'length')查看一下：//（Object.getOwnPropertyDescriptor用于返回defineProperty.descriptor）

{ configurable: false enumerable: false value: 0 writable: true } configurable为false，且MDN上也说重定义数组的length属性在不同浏览器上表现也是不一致的，所以还是老老实实拦截splice，push等方法，或者使用ES6的Proxy。

### 第 1075 行至第 1153 行

*   set //在对象上设置一个属性。如果是新的属性就会触发更改通知（旧属性也会触发更新通知，因为第一个添加的时候已经监听了，之后自动触发，不再手动触发）
*   del //删除一个属性，如果必要触发通知
*   dependArray // 收集数组的依赖 [link](http://qjzd.net:3000/topic/57cd26dad703dbd15b10c707)

### 第 1157 行至第 1568 行

// 配置选项合并策略

```javascript
ar strats = config.optionMergeStrategies;

```

*   mergeData
*   strats.data
*   mergeDataOrFn
*   mergeHook
*   mergeAssets
*   strats.watch
*   strats.computed
*   defaultStrat
*   checkComponents
*   validateComponentName
*   normalizeProps
*   normalizeInject
*   normalizeDirectives
*   assertObjectType
*   mergeOptions

这一部分代码写的就是父子组件配置项的合并策略，包括：默认的合并策略、钩子函数的合并策略、filters/props、data合并策略，且包括标准的组件名、props写法有一个统一化规范要求。

一图以蔽之

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="733"></svg>)

强烈推荐阅读：[link](https://segmentfault.com/a/1190000014738314#item-1)

### 阶段小结

这一部分最重要的就是 Observer（观察者） ，这也是 Vue 核心中的核心！其次是 mergeOptions（组件配置项的合并策略），但是通常在用的过程中，就已经了解到了大部分的策略规则。

### 第 1570 行至第 1754 行

*   resolveAsset// resolveAsset 全局注册组件用到

e.g.

我们的调用 resolveAsset(context.![options, 'components', tag)，即拿 vm.](https://juejin.im/equation?tex=options%2C%20'components'%2C%20tag)%EF%BC%8C%E5%8D%B3%E6%8B%BF%20vm.)options.components\[tag\]，这样我们就可以在 resolveAsset 的时候拿到这个组件的构造函数，并作为 createComponent 的钩子的参数。

*   validateProp// prop的格式校验

校验prop：

1.  prop为Boolean类型时做特殊处理
2.  prop的值为空时，获取默认值，并创建观察者对象
3.  prop验证

*   getPropDefaultValue// 获取默认 prop 值

获取 prop 的默认值 && 创建观察者对象

1.  @param {\*} vm vm 实例
2.  @param {\*} prop 定义选项
3.  @param {\*} vmkey prop 的 key

// 在非生产环境下（除去 Weex 的某种情况），将对prop进行验证，包括验证required、type和自定义验证函数。

*   assertProp //验证 prop Assert whether a prop is valid.

```javascript
case 1: 验证 required 属性
   case 1.1: prop 定义时是 required，但是调用组件时没有传递该值（警告）
   case 1.2: prop 定义时是非 required 的，且 value === null || value === undefined（符合要求，返回）
case 2: 验证 type 属性-- value 的类型必须是 type 数组里的其中之一
case 3: 验证自定义验证函数

```

*   assertType

```javascript
`assertType`函数，验证`prop`的值符合指定的`type`类型，分为三类：
  - 第一类：通过`typeof`判断的类型，如`String`、`Number`、`Boolean`、`Function`、`Symbol`
  - 第二类：通过`Object.prototype.toString`判断`Object`/`Array`
  - 第三类：通过`instanceof`判断自定义的引用类型

```

### 第 1756 行至第 1823 行

// 辅助函数：检测内置类型

*   getType
*   isSameType
*   getTypeIndex
*   getInvalidTypeMessage
*   styleValue
*   isExplicable
*   isBoolean

### 第 1827 行至第 1901 行

// 辅助函数：处理错误、错误打印

*   handleError
*   invokeWithErrorHandling
*   globalHandleError
*   logError

### 第 1905 行至第 2007 行

*   flushCallbacks// flushCallbacks 挨个同步执行callbacks中回调
*   MutationObserver
*   nextTick// 把传入的 cb 回调函数用 try-catch 包裹后放在一个匿名函数中推入callbacks数组中，这么做是因为防止单个 cb 如果执行错误不至于让整个JS线程挂掉，每个 cb 都包裹是防止这些回调函数如果执行错误不会相互影响，比如前一个抛错了后一个仍然可以执行。

**精髓中的精髓 —— nextTick**

这里有一段很重要的注释

```javascript
// Here we have async deferring wrappers using microtasks.
// In 2.5 we used (macro) tasks (in combination with microtasks).
// However, it has subtle problems when state is changed right before repaint
// (e.g. #6813, out-in transitions).
// Also, using (macro) tasks in event handler would cause some weird behaviors
// that cannot be circumvented (e.g. #7109, #7153, #7546, #7834, #8109).
// So we now use microtasks everywhere, again.
// A major drawback of this tradeoff is that there are some scenarios
// where microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690, which have workarounds)
// or even between bubbling of the same event (#6566).

在vue2.5之前的版本中，nextTick基本上基于 micro task 来实现的，但是在某些情况下 micro task 具有太高的优先级，并且可能在连续顺序事件之间（例如＃4521，＃6690）或者甚至在同一事件的事件冒泡过程中之间触发（＃6566）。但是如果全部都改成 macro task，对一些有重绘和动画的场景也会有性能影响，如 issue #6813。vue2.5之后版本提供的解决办法是默认使用 micro task，但在需要时（例如在v-on附加的事件处理程序中）强制使用 macro task。

```

什么意思呢？分析下面这段代码。

```javascript
<span id='name' ref='name'>{{ name }}</span>
<button @click='change'>change name</button>

methods: {
      change() {
          this.$nextTick(() => console.log('setter前：' + this.$refs.name.innerHTML))
          this.name = ' vue3 '
          console.log('同步方式：' + this.$refs.name.innerHTML)
          setTimeout(() => this.console("setTimeout方式：" + this.$refs.name.innerHTML))
          this.$nextTick(() => console.log('setter后：' + this.$refs.name.innerHTML))
          this.$nextTick().then(() => console.log('Promise方式：' + this.$refs.name.innerHTML))
      }
  }
//同步方式：vue2
//setter前：vue2
//setter后： vue3 
//Promise方式： vue3 
//setTimeout方式： vue3 

```

1.  同步方式： 当把data中的name修改之后，此时会触发name的 setter 中的 dep.notify 通知依赖本data的render watcher去 update，update 会把 flushSchedulerQueue 函数传递给 nextTick，render watcher在 flushSchedulerQueue 函数运行时 watcher.run 再走 diff -> patch 那一套重渲染 re-render 视图，这个过程中会重新依赖收集，这个过程是异步的；所以当我们直接修改了name之后打印，这时异步的改动还没有被 patch 到视图上，所以获取视图上的DOM元素还是原来的内容。
2.  setter前： setter前为什么还打印原来的是原来内容呢，是因为 nextTick 在被调用的时候把回调挨个push进callbacks数组，之后执行的时候也是 for 循环出来挨个执行，所以是类似于队列这样一个概念，先入先出；在修改name之后，触发把render watcher填入 schedulerQueue 队列并把他的执行函数 flushSchedulerQueue 传递给 nextTick ，此时callbacks队列中已经有了 setter前函数 了，因为这个 cb 是在 setter前函数 之后被push进callbacks队列的，那么先入先出的执行callbacks中回调的时候先执行 setter前函数，这时并未执行render watcher的 watcher.run，所以打印DOM元素仍然是原来的内容。
3.  setter后： setter后这时已经执行完 flushSchedulerQueue，这时render watcher已经把改动 patch 到视图上，所以此时获取DOM是改过之后的内容。
4.  Promise方式： 相当于 Promise.then 的方式执行这个函数，此时DOM已经更改。
5.  setTimeout方式： 最后执行macro task的任务，此时DOM已经更改。

备注：前文提过，在依赖收集原理的响应式化方法 defineReactive 中的 setter 访问器中有派发更新 dep.notify() 方法，这个方法会挨个通知在 dep 的 subs 中收集的订阅自己变动的 watchers 执行 update。

强烈推荐阅读：[link](https://zhuanlan.zhihu.com/p/55423103)

### 0 行 至 2000 行小结

0 至 2000 行主要的内容是：

1.  工具代码
2.  数据监听：Obeserve,Dep
3.  Vnode
4.  nextTick

### 第 2011 行至第 2232 行

*   perf// performance
*   initProxy// 代理对象是es6的新特性，它主要用来自定义对象一些基本操作（如查找，赋值，枚举等）。[link](https://juejin.im/post/5b11db686fb9a01e5b10eae7)

//proxy是一个强大的特性，为我们提供了很多"元编程"能力。

```javascript
const handler = {
    get: function(obj, prop) {
        return prop in obj ? obj[prop] : 37;
    }
};

const p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b);      // 1, undefined
console.log('c' in p, p.c); // false, 37

```

[link](https://juejin.im/post/5a5227ce6fb9a01c927e85c4)

*   traverse// 遍历：\_traverse 深度遍历，用于

traverse 对一个对象做深层递归遍历，因为遍历过程中就是对一个子对象的访问，会触发它们的 getter 过程，这样就可以收集到依赖，也就是订阅它们变化的 watcher，且遍历过程中会把子响应式对象通过它们的 dep id 记录到 seenObjects，避免以后重复访问。

*   normalizeEvent// normalizeEvents是针对v-model的处理,例如在IE下不支持change事件，只能用input事件代替。
*   createFnInvoker// 在初始构建实例时，旧节点是不存在的,此时会调用createFnInvoker函数对事件回调函数做一层封装，由于单个事件的回调可以有多个，因此createFnInvoker的作用是对单个，多个回调事件统一封装处理，返回一个当事件触发时真正执行的匿名函数。
*   updateListeners// updateListeners的逻辑也很简单，它会遍历on事件对新节点事件绑定注册事件，对旧节点移除事件监听，它即要处理原生DOM事件的添加和移除，也要处理自定义事件的添加和移除，

### 阶段小结

[Vue 的事件机制](https://juejin.im/post/5d5a5dbd6fb9a06acc0084dd#heading-7)

### 第 2236 行至第 2422 行

*   mergeVNodeHook// 重点 合并 VNode

// 把 hook 函数合并到 def.data.hook\[hookey\] 中，生成新的 invoker，createFnInvoker 方法

// vnode 原本定义了 init、prepatch、insert、destroy 四个钩子函数，而 mergeVNodeHook 函数就是把一些新的钩子函数合并进来，例如在 transition 过程中合并的 insert 钩子函数，就会合并到组件 vnode 的 insert 钩子函数中，这样当组件插入后，就会执行我们定义的 enterHook 了。

*   extractPropsFromVNodeData// 抽取相应的从父组件上的prop
*   checkProp// 校验 Prop

```javascript
    // The template compiler attempts to minimize the need for normalization by
    // statically analyzing the template at compile time.
    // 模板编译器尝试用最小的需求去规范：在编译时，静态分析模板

    // For plain HTML markup, normalization can be completely skipped because the
    // generated render function is guaranteed to return Array<VNode>. There are
    // two cases where extra normalization is needed:
    // 对于纯 HTML 标签，可跳过标准化，因为生成渲染函数一定会会返回 Vnode Array.有两种情况，需要额外去规范

    // 1. When the children contains components - because a functional component
    // may return an Array instead of a single root. In this case, just a simple
    // normalization is needed - if any child is an Array, we flatten the whole
    // thing with Array.prototype.concat. It is guaranteed to be only 1-level deep
    // because functional components already normalize their own children.
    // 当子级包含组件时-因为功能组件可能会返回Array而不是单个根。在这种情况下，需要规范化-如果任何子级是Array，我们将整个具有Array.prototype.concat的东西。保证只有1级深度，因为功能组件已经规范了自己的子代。

    // 2. When the children contains constructs that always generated nested Arrays,
    // e.g. <template>, <slot>, v-for, or when the children is provided by user
    // with hand-written render functions / JSX. In such cases a full normalization
    // is needed to cater to all possible types of children values.
    // 当子级包含始终生成嵌套数组的构造时，例如<template>，<slot>，v-for或用户提供子代时,具有手写的渲染功能/ JSX。在这种情况下，完全归一化,才能满足所有可能类型的子代值。

```

Q:这一段话说的是什么意思呢？

A：归一化操作其实就是将多维的数组，合并转换成一个一维的数组。在 Vue 中归一化分为三个级别，

1.  不需要进行归一化
    
2.  只需要简单的归一化处理，将数组打平一层
    
3.  完全归一化，将一个 N 层的 children 完全打平为一维数组
    

利用递归来处理的，同时处理了一些边界情况。

### 第 2426 行至第 2490 行

*   initProvide
*   initInjections
*   resolveInject

### 第 2497 行至第 2958 行

*   resolveSlots// Runtime helper for resolving raw children VNodes into a slot object.
*   isWhitespace
*   normalizeScopedSlots
*   normalizeScopedSlot
*   proxyNormalSlot
*   renderList// Runtime helper for rendering v-for lists.
*   renderSlot// Runtime helper for rendering `<slot>`
*   resolveFilter// Runtime helper for resolving filters
*   checkKeyCodes// Runtime helper for checking keyCodes from config.
*   bindObjectProps// Runtime helper for merging v-bind="object" into a VNode's data.
*   renderStatic// Runtime helper for rendering static trees.
*   markOnce// Runtime helper for v-once.

这一部分讲的是辅助程序 —— Vue 的各类渲染方法，从字面意思中可以知道一些方法的用途，这些方法用在Vue生成的渲染函数中。

*   installRenderHelpers// installRenderHelpers 用于执行以上。

### 第 2962 行至第 3515 行

*   FunctionalRenderContext// 创建一个包含渲染要素的函数
*   createFunctionalComponent

函数式组件的实现

```javascript
  Ctor,                                       //Ctro:组件的构造对象(Vue.extend()里的那个Sub函数)
  propsData,                                  //propsData:父组件传递过来的数据(还未验证)
  data,                                       //data:组件的数据
  contextVm,                                  //contextVm:Vue实例 
  children                                    //children:引用该组件时定义的子节点

```

// createFunctionalComponent 最后会执行我们的 render 函数

特注：Vue 组件是 Vue 的核心之一

组件分为：异步组件和函数式组件

这里就是**函数式组件相关**

> Vue提供了一种可以让组件变为无状态、无实例的函数化组件。从原理上说，一般子组件都会经过实例化的过程，而单纯的函数组件并没有这个过程，它可以简单理解为一个中间层，只处理数据，不创建实例，也是由于这个行为，它的渲染开销会低很多。实际的应用场景是，当我们需要在多个组件中选择一个来代为渲染，或者在将children,props,data等数据传递给子组件前进行数据处理时，我们都可以用函数式组件来完成，它本质上也是对组件的一个外部包装。

函数式组件会在组件的对象定义中，将functional属性设置为true，这个属性是区别普通组件和函数式组件的关键。同样的在遇到子组件占位符时，会进入createComponent进行子组件Vnode的创建。\*\*由于functional属性的存在，代码会进入函数式组件的分支中，并返回createFunctionalComponent调用的结果。\*\*注意，执行完createFunctionalComponent后，后续创建子Vnode的逻辑不会执行，这也是之后在创建真实节点过程中不会有子Vnode去实例化子组件的原因。(无实例)

[官方说明](https://cn.vuejs.org/v2/guide/components-dynamic-async.html)

*   cloneAndMarkFunctionalResult
*   mergeProps
*   componentVNodeHooks
*   createComponent // createComponent 方法创建一个组件的 VNode。这 createComponent 是创建子组件的关键

// 创建组件的 VNode 时，若组件是函数式组件，则其 VNode 的创建过程将与普通组件有所区别。

*   createComponentInstanceForVnode // [link](https://github.com/HcySunYang/vue-design/issues/199)

推荐阅读：[link](https://juejin.im/post/5ca2437b51882543b7019f38#heading-7)

*   installComponentHooks // installComponentHooks就是把 componentVNodeHooks的钩子函数合并到data.hook中，，在合并过程中，如果某个时机的钩子已经存在data.hook中，那么通过执行mergeHook函数做合并勾子。
    
*   mergeHook$1
    
*   transformModel
    
*   createElement// 创建元素
    
*   \_createElement
    
*   applyNS
    
*   registerDeepBindings
    
*   initRender // 初识渲染
    

[link](https://zhuanlan.zhihu.com/p/79538534)

### 阶段小结

这一部分主要是围绕 Vue 的组件的创建。Vue 将页面划分成各类的组件，组件思想是 Vue 的精髓之一。

### 第 3517 行至第 3894 行

*   renderMixin // 引入视图渲染混合函数
    
*   ensureCtor
    
*   createAsyncPlaceholder
    
*   resolveAsyncComponent
    
*   isAsyncPlaceholder
    
*   getFirstComponentChild
    
*   initEvents// 初始化事件
    
*   add
    
*   remove$1
    
*   createOnceHandler
    
*   updateComponentListeners
    
*   eventsMixin // 挂载事件响应相关方法
    

### 第 3898 行至第 4227 行

*   setActiveInstance
*   initLifecycle
*   lifecycleMixin// 挂载生命周期相关方法
*   mountComponent
*   updateChildComponent
*   isInInactiveTree
*   activateChildComponent
*   deactivateChildComponent
*   callHook

> 几乎所有JS框架或插件的编写都有一个类似的模式，即向全局输出一个类或者说构造函数，通过创建实例来使用这个类的公开方法，或者使用类的静态全局方法辅助实现功能。相信精通Jquery或编写过Jquery插件的开发者会对这个模式非常熟悉。Vue.js也如出一辙，只是一开始接触这个框架的时候对它所能实现的功能的感叹盖过了它也不过是一个内容较为丰富和精致的大型类的本质。

[link](https://juejin.im/post/5c62218e6fb9a049e063cf5e)

### 阶段小结

这里要对 js 的继承有一个深刻的理解。 [link](https://juejin.im/entry/58dfbe0361ff4b006b166388)

1.  类继承

```javascript
function Animal(){
    this.live=true;
}
function Dog(name){
    this.name=name
}
Dog.prototype=new Animal()
var dog1=new Dog("wangcai")
console.log(dog1)// Dog {name: "wangcai"}
console.log(dog1.live)// true

```

2.  构造继承

```javascript
function Animal(name,color){
    this.name=name;
    this.color=color;}
function Dog(){
    Animal.apply(this,arguments)
}
var dog1=new Dog("wangcai","balck")
console.log(dog1)// Dog {name: "wangcai", color: "balck"}

```

3.  组合继承（类继承 + 构造继承）

```javascript
function Animal(name,color){
    this.name=name;
    this.color=color;
    this.live=true;
}
function Dog(){
    Animal.apply(this, arguments);   
}
Dog.prototype=new Animal()
var dog1=new Dog("wangcai","black")
console.log(dog1)// Dog {name: "wangcai", color: "black", live: true}

```

4.  寄生组合式继承
5.  extend继承

Vue 同 Jquery 一样，本质也是一个大型的类库。

// 定义Vue构造函数，形参options

```javascript
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // ...  
  this._init(options)
}

```

// 功能函数

```javascript
// 引入初始化混合函数
import { initMixin } from './init'
// 引入状态混合函数
import { stateMixin } from './state'
// 引入视图渲染混合函数
import { renderMixin } from './render'
// 引入事件混合函数
import { eventsMixin } from './events'
// 引入生命周期混合函数
import { lifecycleMixin } from './lifecycle'
// 引入warn控制台错误提示函数
import { warn } from '../util/index'
...

// 挂载初始化方法
initMixin(Vue)
// 挂载状态处理相关方法
stateMixin(Vue)
// 挂载事件响应相关方法
eventsMixin(Vue)
// 挂载生命周期相关方法
lifecycleMixin(Vue)
// 挂载视图渲染方法
renderMixin(Vue)

```

### 第 4231 行至第 4406 行

*   resetSchedulerState // 重置状态
*   flushSchedulerQueue// 据变化最终会把flushSchedulerQueue传入到nextTick中执行flushSchedulerQueue函数会遍历执行watcher.run()方法，watcher.run()方法最终会完成视图更新

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="633" height="418"></svg>)

vue中dom的更像并不是实时的，当数据改变后，vue会把渲染watcher添加到异步队列，异步执行，同步代码执行完成后再统一修改dom。

*   callUpdatedHooks
*   queueActivatedComponent
*   callActivatedHooks
*   queueWatcher

[link](https://cloud.tencent.com/developer/article/1356678)

### 第 4412 行至第 4614 行

*   Watcher// !important 重中之重的重点

这一 part 在 Watcher 的原型链上定义了get、addDep、cleanupDeps、update、run、evaluate、depend、teardown 方法，即 Watcher 的具体实现的一些方法，比如新增依赖、清除、更新试图等。

每个Vue组件都有一个对应的watcher，这个watcher将会在组件render的时候收集组件所依赖的数据，并在依赖有更新的时候，触发组件重新渲染。

### 第 4618 行至第 5071 行

```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }
    // 如果是Vue的实例，则不需要被observe
    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    // 第一步： options参数的处理
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      // mergeOptions接下来我们会详细讲哦~
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    // 第二步： renderProxy
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    // 第三步： vm的生命周期相关变量初始化
    initLifecycle(vm)
    
    // 第四步： vm的事件监听初始化
    initEvents(vm)
    // 第五步： vm的编译render初始化
    initRender(vm)
    // 第六步： vm的beforeCreate生命钩子的回调
    callHook(vm, 'beforeCreate')
    // 第七步： vm在data/props初始化之前要进行绑定
    initInjections(vm) // resolve injections before data/props
    
    // 第八步： vm的sate状态初始化
    initState(vm)
    // 第九步： vm在data/props之后要进行提供
    initProvide(vm) // resolve provide after data/props
    // 第十步： vm的created生命钩子的回调
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }
    // 第十一步：render & mount
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}

```

主要是为我们的Vue原型上定义一个方法\_init。然后当我们执行new Vue(options) 的时候，会调用这个方法。而这个\_init方法的实现，便是我们需要关注的地方。 前面定义vm实例都挺好理解的，主要我们来看一下mergeOptions这个方法，其实Vue在实例化的过程中，会在代码运行后增加很多新的东西进去。我们把我们传入的这个对象叫options，实例中我们可以通过vm.$options访问到。

[link](https://juejin.im/post/5bb471c0f265da0afb335931)

### 0 至 5000 行 总结

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="796"></svg>)

从 0 至 5000 行我们可以清晰看到 Vue 模板编译的轮廓了。

*   笔者将这一部分出现的关键词进行按顺序罗列：

1.  function (global, factory)
2.  工具函数
3.  Dep
4.  Observe
5.  VNode
6.  nextTick
7.  事件机制
8.  Render
9.  components
10.  Watcher

我们可以总结：Vue 的核心就是 VDOM ！对 DOM 对象的操作调整为操作 VNode 对象，采用 diff 算法比较差异，一次 patch。

render 的流程是:

1.  Vue使用HTML的Parser将HTML模板解析为AST
2.  function render(){}
3.  Virtual DOM
4.  watcher将会在组件render的时候收集组件所依赖的数据，并在依赖有更新的时候，触发组件重新渲染

推荐阅读：[link](https://gershonv.github.io/2018/07/04/vue-render/)

### 第 5073 行至第 5446 行

```javascript
// 定义 Vue 构造函数
function Vue (options) {
      if (!(this instanceof Vue)
      ) {
        warn('Vue is a constructor and should be called with the `new` keyword');
      }
      this._init(options);
    }

// 将 Vue 作为参数传递给导入的五个方法
initMixin(Vue);// 初始化 Mixin
stateMixin(Vue);// 状态 Mixin
eventsMixin(Vue);// 事件 Mixin
lifecycleMixin(Vue);// 生命周期 Mixin
renderMixin(Vue);// 渲染 Mixin

```

这一部分就是初始化函数的调用。

```javascript
// 
Object.defineProperty(Vue.prototype, '$isServer', {
      get: isServerRendering
    });

```

为什么这么写？

Object.defineProperty能保护引入的库不被重新赋值，如果你尝试重写，程序会抛出“TypeError: Cannot assign to read only property”的错误。

[link-【译】Vue框架引入JS库的正确姿势](https://wufenfen.github.io/2017/04/24/%E3%80%90%E8%AF%91%E3%80%91Vue%E6%A1%86%E6%9E%B6%E5%BC%95%E5%85%A5JS%E5%BA%93%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF/)

```javascript
// 版本
Vue.version = '2.6.11';

```

### 阶段小结

这一部分是 Vue index.js 的内容,包括 Vue 的整个挂在过程

1.  先进入 initMixin(Vue),在prototype上挂载

```javascript
Vue.prototype._init = function (options) {} 

```

2.  进入 stateMixin(Vue),在prototype上挂载 Vue.prototype.$data

```javascript
Vue.prototype.$props 
Vue.prototype.$set = set 
Vue.prototype.$delete = del 
Vue.prototype.$watch = function(){} 

```

3.  进入eventsMixin(Vue),在prototype上挂载

```javascript
Vue.prototype.$on 
Vue.prototype.$once 
Vue.prototype.$off 
Vue.prototype.$emit

```

4.  进入lifecycleMixin(Vue),在prototype上挂载

```javascript
Vue.prototype._update 
Vue.prototype.$forceUpdate 
Vue.prototype.$destroy

```

5.  最后进入renderMixin(Vue),在prototype上挂载 Vue.prototype.$nextTick

```javascript
Vue.prototype._render 
Vue.prototype._o = markOnce 
Vue.prototype._n = toNumber 
Vue.prototype._s = toString 
Vue.prototype._l = renderList 
Vue.prototype._t = renderSlot
Vue.prototype._q = looseEqual 
Vue.prototype._i = looseIndexOf 
Vue.prototype._m = renderStatic 
Vue.prototype._f = resolveFilter 
Vue.prototype._k = checkKeyCodes 
Vue.prototype._b = bindObjectProps 
Vue.prototype._v = createTextVNode 
Vue.prototype._e = createEmptyVNode 
Vue.prototype._u = resolveScopedSlots 
Vue.prototype._g = bindObjectListeners

```

> mergeOptions使用策略模式合并传入的options和Vue.options合并后的代码结构, 可以看到通过合并策略components,directives,filters继承了全局的, 这就是为什么全局注册的可以在任何地方使用,因为每个实例都继承了全局的, 所以都能找到。

推荐阅读：

[link](https://juejin.im/post/5ab07a63f265da2389258b12#heading-16)

[link](https://juejin.im/post/5bb471c0f265da0afb335931#heading-0)

new 一个 Vue 对象发生了什么：

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="817"></svg>)

### 第 5452 行至第 5655 行

```javascript
// these are reserved for web because they are directly compiled away
// during template compilation

// 这些是为web保留的，因为它们是直接编译掉的
// 在模板编译期间

```

*   isBooleanAttr
*   genClassForVnode// class 转码获取vonde 中的staticClass 静态class 和class动态class转义成真实dom需要的class格式。然后返回class字符串
*   mergeClassData// mergeClassData
*   renderClass// 渲染calss 这里获取到已经转码的calss
*   stringifyClass// 转码 class，把数组格式，对象格式的calss 全部转化成 字符串格式
*   stringifyArray// 数组字符串变成字符串，然后用空格 隔开 拼接 起来变成字符串
*   stringifyObject// 对象字符串变成字符串，然后用空格 隔开 拼接 起来变成字符串
*   namespaceMap
*   isHTMLTag
*   isSVG// 判断svg 标签
*   isUnknownElement// 检查dom 节点的tag标签 类型 是否是VPre 标签 或者是判断是否是浏览器自带原有的标签
*   isTextInputType // //匹配'text,number,password,search,email,tel,url'

这一 part 没有特别要说的，主要是对 class 的转码、合并和其他二次封装的工具函数。实际上我们在 Vue 源码很多地方看到了这样的封装，在平常的开发中，我们也得要求自己封装基本的函数。如果能形成自己习惯用的函数的库，会方便很多，且对自己能力也是一个提升。

### 第 5659 行至第 5792 行

*   createElement // 创建元素，实例化 VNode
*   createElementNS
*   createTextNode
*   createComment
*   insertBefore
*   removeChild
*   appendChild
*   parentNode
*   nextSibling
*   tagName
*   setTextContent
*   setStyleScope
*   nodeOps

```javascript
// nodeOps:
    createElement: createElement$1, //创建一个真实的dom
    createElementNS: createElementNS, //创建一个真实的dom svg方式
    createTextNode: createTextNode, // 创建文本节点
    createComment: createComment,  // 创建一个注释节点
    insertBefore: insertBefore,  //插入节点 在xxx  dom 前面插入一个节点
    removeChild: removeChild,   //删除子节点
    appendChild: appendChild,  //添加子节点 尾部
    parentNode: parentNode,  //获取父亲子节点dom
    nextSibling: nextSibling,     //获取下一个兄弟节点
    tagName: tagName,   //获取dom标签名称
    setTextContent: setTextContent, //  //设置dom 文本
    setStyleScope: setStyleScope  //设置组建样式的作用域

```

*   ref
*   registerRef // 注册ref或者删除ref。比如标签上面设置了ref='abc' 那么该函数就是为this.$refs.abc 注册ref 把真实的dom存进去

### 阶段小结

这里的重点想必就是 “ref” 了

在绝大多数情况下，我们最好不要触达另一个组件实例内部或手动操作 DOM 元素。不过也确实在一些情况下做这些事情是合适的。ref 为我们提供了解决途径。

ref属性不是一个标准的HTML属性，只是Vue中的一个属性。

### 第 5794 行至第 6006 行

Virtual DOM !

没错，这里就是 虚拟 dom 生成的源码相关。

*   sameVnode
*   sameInputType
*   createKeyToOldIdx
*   createPatchFunction // !important:patch 把 vonde 渲染成真实的 dom
*   emptyNodeAt
*   createRmCb
*   removeNode
*   isUnknownElement$$1
*   createElm // 创造 dom 节点
*   createComponent // 创建组件，并且判断它是否实例化过
*   initComponent

> createElement方法接收一个tag参数，在内部会去判断tag标签的类型，从而去决定是创建一个普通的VNode还是一个组件类VNode；

createComponent 的实现，在渲染一个组件的时候的 3 个关键逻辑：

1.  构造子类构造函数，
2.  安装组件钩子函数
3.  实例化 vnode。createComponent 后返回的是组件 vnode，它也一样走到 vm.\_update 方法

我们传入的 vnode 是组件渲染的 vnode，也就是我们之前说的 vm.\_vnode，如果组件的根节点是个普通元素，那么 vm.\_vnode 也是普通的 vnode，这里 createComponent(vnode, insertedVnodeQueue, parentElm, refElm) 的返回值是 false。接下来的过程就系列一的步骤一样了，先创建一个父节点占位符，然后再遍历所有子 VNode 递归调用 createElm，在遍历的过程中，如果遇到子 VNode 是一个组件的 VNode，则重复过程，这样通过一个递归的方式就可以完整地构建了整个组件树。

> initComponent 初始化组建，如果没有tag标签则去更新真实dom的属性，如果有tag标签，则注册或者删除ref 然后为insertedVnodeQueue.push(vnode);

[参考link](https://www.cnblogs.com/hao123456/p/10616356.html)

### 第 6008 行至第 6252 行

*   reactivateComponent
*   insert
*   createChildren
*   isPatchable
*   invokeCreateHooks
*   setScope
*   addVnodes // 添加 Vnodes
*   invokeDestroyHook
*   removeVnodes // 移除 Vnodes
*   removeAndInvokeRemoveHook
*   updateChildren // 在patchVnode中提到，如果新老节点都有子节点，但是不相同的时候就会调用 updateChildren，这个函数通过diff算法尽可能的复用先前的DOM节点。

// diff 算法就在这里辣！[详解link](https://juejin.im/post/5affd01551882542c83301da)

```javascript
function updateChildren(parentElm, oldCh, newCh, insertedVnodeQueue) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, elmToMove, refElm 
    
    while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (isUndef(oldStartVnode)) {
            oldStartVnode = oldCh[++oldStartIdx]
        } else if (isUndef(oldEndVnode)) {
            oldEndVnode = oldCh[--oldEndIdx]
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
            patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
            oldStartVnode = oldCh[++oldStartIdx]
            newStartVnode = newCh[++newStartIdx]
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
            oldEndVnode = oldCh[--oldEndIdx]
            newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldStartVnode, newEndVnode)) {
            patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
            canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
            oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldEndVnode, newStartVnode)) {
            patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
            canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
            oldEndVnode = oldCh[--oldEndIdx]
            newStartVnode = newCh[++newStartIdx]
        } else {
            if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
            idxInOld = isDef(newStartVnode.key) ? oldKeyToIdx[newStartVnode.key] : null
            if (isUndef(idxInOld)) {
                createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
                newStartVnode = newCh[++newStartIdx]
            } else {
                elmToMove = oldCh[idxInOld]
                if (sameVnode(elmToMove, newStartVnode)) {
                    patchVnode(elmToMove, newStartVnode, insertedVnodeQueue)
                    oldCh[idxInOld] = undefined
                    canMove && nodeOps.insertBefore(parentElm, newStartVnode.elm, oldStartVnode.elm)
                    newStartVnode = newCh[++newStartIdx]
                } else {
                    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
                    newStartVnode = newCh[++newStartIdx]
                }
            }
        }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }
}

```

*   checkDuplicateKeys
*   findIdxInOld

reactivateComponent 承接上文 createComponent

### 第 6259 行至第 6561 行

*   patchVnode // 如果符合sameVnode，就不会渲染vnode重新创建DOM节点，而是在原有的DOM节点上进行修补，尽可能复用原有的DOM节点。
*   invokeInsertHook
*   isRenderedModule
*   hydrate
*   assertNodeMatch
*   patch // !important: patch的本质是将新旧vnode进行比较，创建、删除或者更新DOM节点/组件实例

### 阶段小结

Vue 的核心思想：组件化。

这一部分是关于构建组件树，形成虚拟 dom ，以及非常重要的 patch 方法。

再来亿遍：

1.  原因：当修改某条数据的时候，这时候js会将整个DOM Tree进行替换，这种操作是相当消耗性能的。所以在Vue中引入了Vnode的概念：Vnode是对真实DOM节点的模拟，可以对Vnode Tree进行增加节点、删除节点和修改节点操作。这些过程都只需要操作VNode Tree，不需要操作真实的DOM，大大的提升了性能。修改之后使用diff算法计算出修改的最小单位，在将这些小单位的视图进行更新。
    
2.  原理：data中定义了一个变量a，并且模板中也使用了它，那么这里生成的Watcher就会加入到a的订阅者列表中。当a发生改变时，对应的订阅者收到变动信息，这时候就会触发Watcher的update方法，实际update最后调用的就是在这里声明的updateComponent。 当数据发生改变时会触发回调函数updateComponent，updateComponent是对patch过程的封装。patch的本质是将新旧vnode进行比较，创建、删除或者更新DOM节点/组件实例。
    

联系前后QA

Q：vue.js 同时多个赋值是一次性渲染还是多次渲染DOM？

A：官网已给出答案：[cn.vuejs.org/v2/guide/re…](https://cn.vuejs.org/v2/guide/reactivity.html)

> 可能你还没有注意到，Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。

> 例如，当你设置 vm.someData = 'new value'，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新。多数情况我们不需要关心这个过程，但是如果你想基于更新后的 DOM 状态来做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员使用“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们必须要这么做。为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 Vue.nextTick(callback)。这样回调函数将在 DOM 更新完成后被调用。

这样是不是有种前后连贯起来的感觉，原来 nextTick 是这样子的。

*   [参考link1](https://juejin.im/post/5ddb91e451882572fe6edf69#heading-4)
*   [参考link2](https://juejin.im/post/5e2804e1e51d453c9e155f08#heading-4)

### 第 6566 行至第 7069 行

*   directives // 官网：[cn.vuejs.org/v2/guide/cu…](https://cn.vuejs.org/v2/guide/custom-directive.html)
    
*   updateDirectives // 更新指令
    
*   \_update
    
*   normalizeDirectives // 统一directives的格式
    
*   getRawDirName // 返回指令名称 或者属性name名称+修饰符
    
*   callHook$1 //触发指令钩子函数
    
*   updateAttrs // 更新属性
    
*   setAttr // 设置属性
    
*   baseSetAttr
    
*   updateClass // 更新样式
    
*   klass
    
*   parseFilters // 处理value 解析成正确的value，把过滤器 转换成 vue 虚拟dom的解析方法函数 比如把过滤器 ' ab | c | d' 转换成 \_f("d")(\_f("c")(ab))
    
*   wrapFilter // 转换过滤器格式
    
*   baseWarn // 基础警告
    
*   pluckModuleFunction //循环过滤数组或者对象的值，根据key循环 过滤对象或者数组\[key\]值，如果不存在则丢弃，如果有相同多个的key值，返回多个值的数组
    
*   addProp //在虚拟dom中添加prop属性
    
*   addAttr //添加attrs属性
    
*   addRawAttr //添加原始attr(在预转换中使用)
    
*   addDirective //为虚拟dom 添加一个 指令directives属性 对象
    
*   addHandler // 为虚拟dom添加events 事件对象属性
    

前面围绕“指令”和“过滤器”的一些基础工具函数。

后面围绕为虚拟 dom 添加属性、事件等具体实现函数。

### 第 7071 行至第 7298 行

*   getRawBindingAttr
*   getBindingAttr // 获取 :属性 或者v-bind:属性，或者获取属性 移除传进来的属性name，并且返回获取到 属性的值
*   getAndRemoveAttr // 移除传进来的属性name，并且返回获取到 属性的值
*   getAndRemoveAttrByRegex
*   rangeSetItem
*   genComponentModel // 为虚拟dom添加model属性

```javascript
    /*
    * Parse a v-model expression into a base path and a final key segment.
    * Handles both dot-path and possible square brackets.
    * 将 v-model 表达式解析为基路径和最后一个键段。
    * 处理点路径和可能的方括号。
    */

```

*   parseModel //转义字符串对象拆分字符串对象 把后一位key分离出来

// 如果数据是object.info.name的情况下 则返回是 {exp: "object.info",key: "name"} // 如果数据是object\[info\]\[name\]的情况下 则返回是 {exp: "object\[info\]",key: "name"}

*   next
*   eof
*   parseBracket //检测 匹配\[\] 一对这样的=括号
*   parseString // 循环匹配一对''或者""符号

这一部分包括：原生指令 v-bind 和为虚拟 dom 添加 model 属性，以及格式校验工具函数。

### 第 7300 行至第 7473 行

*   model
*   genCheckboxModel // 为input type="checkbox" 虚拟dom添加 change 函数 ，根据v-model是否是数组，调用change函数，调用 set 去更新 checked选中数据的值
*   genRadioModel // 为虚拟dom inpu标签 type === 'radio' 添加change 事件 更新值
*   genSelect // 为虚拟dom添加change 函数 ，change 函数调用 set 去更新 select 选中数据的值
*   genDefaultModel // 如果虚拟dom标签是 'input' 类型不是checkbox，radio 或者是'textarea' 标签的时候，获取真实的dom的value值调用 change或者input方法执行set方法更新数据

[参考link](https://www.cnblogs.com/hao123456/p/10616356.html)

### 阶段小结

*   v-bind、v-model

区别：

1.  v-bind 用来绑定数据和属性以及表达式，缩写为'：'
2.  v-model 使用在表单中，实现双向数据绑定的，在表单元素外使用不起作用

Q：你知道v-model的原理吗？说说看

A: v-model本质上是语法糖，即利用v-model绑定数据，其实就是既绑定了数据，又添加了一个input事件监听 [link](https://github.com/haizlin/fe-interview/issues/560)

*   自定义指令钩子函数

一个指令定义对象可以提供如下几个钩子函数 (均为可选)：

```javascript
1. bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
2. inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
3. update：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。
4. componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
5. unbind：只调用一次，指令与元素解绑时调用。

```

*   指令钩子函数会被传入以下参数：

```javascript
1. el：指令所绑定的元素，可以用来直接操作 DOM 。
2. binding：一个对象，包含以下属性：
     name：指令名，不包括 v- 前缀。
     value：指令的绑定值，例如：v-my-directive="1 + 1" 中，绑定值为 2。
     oldValue：指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
     expression：字符串形式的指令表达式。例如 v-my-directive="1 + 1" 中，表达式为 "1 + 1"。
     arg：传给指令的参数，可选。例如 v-my-directive:foo 中，参数为 "foo"。
     modifiers：一个包含修饰符的对象。例如：v-my-directive.foo.bar 中，修饰符对象为 { foo: true, bar: true }。
3. vnode：Vue 编译生成的虚拟节点。移步 VNode API 来了解更多详情。
4. oldVnode：上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。

```

除了 el 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 dataset 来进行。

[【译】vue 自定义指令的魅力](https://juejin.im/post/59ffbcc151882554b836ee21)

### 第 7473 行至第 7697 行

*   normalizeEvents // 为事件 多添加 change 或者input 事件加进去
*   createOnceHandler$1
*   add$1 // 为真实的dom添加事件
*   remove$2
*   updateDOMListeners // 更新dom事件
*   updateDOMProps // 更新真实dom的props属性
*   shouldUpdateValue // 判断是否需要更新value
*   isNotInFocusAndDirty
*   isDirtyWithModifiers // 判断脏数据修改 [脏数据概念](https://blog.csdn.net/LVXIANGAN/article/details/85329630)

### 第 7699 行至第 7797 行

*   domProps
*   parseStyleText // 把style 字符串 转换成对象
*   normalizeStyleData // 在同一个vnode上合并静态和动态样式数据
*   normalizeStyleBinding // 将可能的数组/字符串值规范化为对象
*   getStyle

```javascript
    /**
    * parent component style should be after child's
    * so that parent component's style could override it
    * 父组件样式应该在子组件样式之后
    * 这样父组件的样式就可以覆盖它
    * 循环子组件和组件的样式，把它全部合并到一个样式对象中返回 样式对象 如{width:100px,height:200px} 返回该字符串。
    */

```

*   setProp // 设置 prop

### 第 7799 行至第 7995 行

*   normalize // 给css加前缀。解决浏览器兼用性问题，加前缀
*   updateStyle // 将vonde虚拟dom的css 转义成并且渲染到真实dom的csszhong
*   addClass // 为真实dom 元素添加class类
*   removeClass // 删除真实dom的css类名
*   resolveTransition // 解析vonde中的transition的name属性获取到一个css过度对象类
*   autoCssTransition // 通过 name 属性获取过渡 CSS 类名 比如标签上面定义name是 fade css就要定义 .fade-enter-active,.fade-leave-active，.fade-enter,.fade-leave-to 这样的class
*   nextFrame // 下一帧

### 第 7997 行至第 8093 行

*   addTransitionClass // 获取 真实dom addTransitionClass 记录calss类
*   removeTransitionClass // 删除vonde的class类和删除真实dom的class类
*   whenTransitionEnds // 获取动画的信息，执行动画。
*   getTransitionInfo // 获取transition，或者animation 动画的类型，动画个数，动画执行时间

这一部分关于：对真实 dom 的操作，包括样式的增删、事件的增删、动画类等。

回过头再理一下宏观上的东西，再来亿遍-虚拟DOM：模板 → 渲染函数 → 虚拟DOM树 → 真实DOM

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1237" height="225"></svg>)

那么这一部分则处在“虚拟DOM树 → 真实DOM”这个阶段

### 第 8093 行至第 8518 行

*   getTimeout

```javascript
// Old versions of Chromium (below 61.0.3163.100) formats floating pointer numbers
// in a locale-dependent way, using a comma instead of a dot.
// If comma is not replaced with a dot, the input will be rounded down (i.e. acting
// as a floor function) causing unexpected behaviors

// 根据本地的依赖方式，Chromium 的旧版本（低于61.0.3163.100）格式化浮点数字，使用逗号而不是点。如果逗号未用点代替，则输入将被四舍五入而导致意外行为

```

*   toMs // [update toMs function. fix #4894](https://github.com/vuejs/vue/pull/8495/files)
*   enter

```javascript
// activeInstance will always be the <transition> component managing this
// transition. One edge case to check is when the <transition> is placed
// as the root node of a child component. In that case we need to check
// <transition>'s parent for appear check.

// activeInstance 将一直作为<transition>的组件来管理 transition。要检查的一种边缘情况：<transition> 作为子组件的根节点时。在这种情况下，我们需要检查 <transition> 的父项的展现。

```

*   leave // 离开动画
    
*   performLeave
    
*   checkDuration // only used in dev mode : 检测 val 必需是数字
    
*   isValidDuration
    
*   getHookArgumentsLength // 检测钩子函数 fns 的长度
    
*   \_enter
    
*   createPatchFunction // path 把vonde 渲染成真实的dom：创建虚拟 dom - 函数体在 5845 行
    
*   directive // 生命指令：包括 插入 和 组件更新
    

> 更新指令 比较 oldVnode 和 vnode，根据oldVnode和vnode的情况 触发指令钩子函数bind，update，inserted，insert，componentUpdated，unbind钩子函数

此节前部分是 transition 动画相关工具函数，后部分关于虚拟 Dom patch、指令的更新。

### 第 8520 行至第 8584 行

*   setSelected // 设置选择 - 指令更新的工具函数
*   actuallySetSelected // 实际选择，在 setSelected() 里调用
*   hasNoMatchingOption // 没有匹配项 - 指令组件更新工具函数
*   getValue // 获取 option.value
*   onCompositionStart // 组成开始 - 指令插入工具函数
*   onCompositionEnd // 组成结束-指令插入工具函数：防止无故触发输入事件
*   trigger // 触发事件

### 第 8592 行至第 8728 行

// 定义在组件根内部递归搜索可能存在的 transition

*   locateNode
*   show // 控制 el 的 display 属性
*   platformDirectives // 平台指令
*   transitionProps // 过渡Props对象

```javascript
    // in case the child is also an abstract component, e.g. <keep-alive>
    // we want to recursively retrieve the real component to be rendered
    // 如果子对象也是抽象组件，例如<keep-alive>
    // 我们要递归地检索要渲染的实际组件

```

*   getRealChild
*   extractTransitionData // 提取 TransitionData
*   placeholder // 占位提示
*   hasParentTransition // 判断是否有 ParentTransition
*   isSameChild // 判断子对象是否相同

### 第 8730 行至第 9020 行

*   Transition // !important

前部分以及此部分大部分围绕 Transition 这个关键对象。即迎合官网 “过渡 & 动画” 这一节，是我们需要关注的重点！

> Vue 在插入、更新或者移除 DOM 时，提供多种不同方式的应用过渡效果。包括以下工具：
> 
> *   在 CSS 过渡和动画中自动应用 class
> *   可以配合使用第三方 CSS 动画库，如 Animate.css
> *   在过渡钩子函数中使用 JavaScript 直接操作 DOM
> *   可以配合使用第三方 JavaScript 动画库，如 Velocity.js
> 
> 在这里，我们只会讲到进入、离开和列表的过渡，你也可以看下一节的[管理过渡状态](https://cn.vuejs.org/v2/guide/transitioning-state.html)。

vue - transition 里面大有东西，这里有一篇[“细谈”](https://juejin.im/post/5cf411d8e51d4550a629b222)推荐阅读。

*   props
*   TransitionGroup // TransitionGroup
*   callPendingCbs // Pending 回调
*   recordPosition // 记录位置
*   applyTranslation // 应用动画 - TransitionGroup.updated 调用

```javascript

// we divide the work into three loops to avoid mixing DOM reads and writes
// in each iteration - which helps prevent layout thrashing.

//我们将工作分为三个 loops，以避免将 DOM 读取和写入混合在一起
//在每次迭代中-有助于防止布局冲撞。

```

*   platformComponents // 平台组件

```javascript
// 安装平台运行时指令和组件
extend(Vue.options.directives, platformDirectives);
extend(Vue.options.components, platformComponents);

```

Q: vue自带的内置组件有什么？

A: Vue中内置的组件有以下几种：

1.  component

component组件：有两个属性---is inline-template

渲染一个‘元组件’为动态组件，按照'is'特性的值来渲染成那个组件

2.  transition

transition组件：为组件的载入和切换提供动画效果，具有非常强的可定制性，支持16个属性和12个事件

3.  transition-group

transition-group：作为多个元素/组件的过渡效果

4.  keep-alive

keep-alive：包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们

5.  slot

slot：作为组件模板之中的内容分发插槽，slot元素自身将被替换

### 第 9024 行至第 9207 行

// install platform specific utils // 安装平台特定的工具

*   Vue.config.x

```javascript
Vue.config.mustUseProp = mustUseProp;
Vue.config.isReservedTag = isReservedTag;
Vue.config.isReservedAttr = isReservedAttr;
Vue.config.getTagNamespace = getTagNamespace;
Vue.config.isUnknownElement = isUnknownElement;

```

*   Vue.prototype.$mount // public mount method 安装方法 实例方法挂载 vm

```javascript
// public mount method
Vue.prototype.$mount = function (
    el, // 真实dom 或者是 string
    hydrating //新的虚拟dom vonde
) {
    el = el && inBrowser ? query(el) : undefined;
    return mountComponent(this, el, hydrating)
};

```

**devtools global hook** // 开发环境全局 hook Tip

*   buildRegex // 构建的正则匹配
*   parseText // 匹配view 指令，并且把他转换成 虚拟dom vonde 需要渲染的函数,比如指令{{name}}转换成 \_s(name)
*   transformNode // 获取 class 属性和:class或者v-bind的动态属性值，并且转化成字符串 添加到staticClass和classBinding 属性中
*   genData // 初始化扩展指令 baseDirectives，on,bind,cloak方法，dataGenFns 获取到一个数组，数组中有两个函数 genData（转换 class） 和 genData$1（转换 style）,
*   transformNode$1 // transformNode$1 获取 style属性和:style或者v-bind的动态属性值，并且转化成字符串 添加到staticStyle和styleBinding属性中
*   genData$1 // 参见 genData
*   style$1 // 包含 staticKeys、transformNode、genData 属性

### 第 9211 行至第 9537 行

*   he
*   isUnaryTag // 工具函数
*   canBeLeftOpenTag // 工具函数
*   isNonPhrasingTag // 工具函数 **Regular Expressions**
*   parseHTML // 解析成 HTML !important

parseHTML 这个函数实现大概两百多行，是一个比较大的函数体了。

parseHTML 中的方法用于处理HTML开始和结束标签。

parseHTML 方法的整体逻辑是用正则判断各种情况，进行不同的处理。其中调用到了 options 中的自定义方法。

options 中的自定义方法用于处理AST语法树，最终返回出整个AST语法树对象。

贴一下源码，有兴趣可自行感受一二。附一篇详解[Vue.js HTML解析细节学习](https://segmentfault.com/a/1190000013592119)

```javascript
function parseHTML(html, options) {
    var stack = [];
    var expectHTML = options.expectHTML;
    var isUnaryTag$$1 = options.isUnaryTag || no;
    var canBeLeftOpenTag$$1 = options.canBeLeftOpenTag || no;
    var index = 0;
    var last, lastTag;
    while (html) {
        last = html;
        // 确保我们不在像脚本/样式这样的纯文本内容元素中
        if (!lastTag || !isPlainTextElement(lastTag)) {
            var textEnd = html.indexOf('<');
            if (textEnd === 0) {
                // Comment:
                if (comment.test(html)) {
                    var commentEnd = html.indexOf('-->');

                    if (commentEnd >= 0) {
                        if (options.shouldKeepComment) {
                            options.comment(html.substring(4, commentEnd), index, index + commentEnd + 3);
                        }
                        advance(commentEnd + 3);
                        continue
                    }
                }

                // http://en.wikipedia.org/wiki/Conditional_comment#Downlevel-revealed_conditional_comment
                if (conditionalComment.test(html)) {
                    var conditionalEnd = html.indexOf(']>');

                    if (conditionalEnd >= 0) {
                        advance(conditionalEnd + 2);
                        continue
                    }
                }

                // Doctype:
                // 匹配 html 的头文件
                var doctypeMatch = html.match(doctype);
                if (doctypeMatch) {
                    advance(doctypeMatch[0].length);
                    continue
                }

                // End tag:
                var endTagMatch = html.match(endTag);
                if (endTagMatch) {
                    var curIndex = index;
                    advance(endTagMatch[0].length);
                    parseEndTag(endTagMatch[1], curIndex, index);
                    continue
                }

                // Start tag:
                // 解析开始标记
                var startTagMatch = parseStartTag();
                if (startTagMatch) {
                    handleStartTag(startTagMatch);
                    if (shouldIgnoreFirstNewline(startTagMatch.tagName, html)) {
                        advance(1);
                    }
                    continue
                }
            }

            var text = (void 0),
                rest = (void 0),
                next = (void 0);
            if (textEnd >= 0) {
                rest = html.slice(textEnd);
                while (
                    !endTag.test(rest) &&
                    !startTagOpen.test(rest) &&
                    !comment.test(rest) &&
                    !conditionalComment.test(rest)
                ) {
                    // < in plain text, be forgiving and treat it as text
                    next = rest.indexOf('<', 1);
                    if (next < 0) {
                        break
                    }
                    textEnd += next;
                    rest = html.slice(textEnd);
                }
                text = html.substring(0, textEnd);
            }

            if (textEnd < 0) {
                text = html;
            }

            if (text) {
                advance(text.length);
            }

            if (options.chars && text) {
                options.chars(text, index - text.length, index);
            }
        } else {
            //  处理是script,style,textarea
            var endTagLength = 0;
            var stackedTag = lastTag.toLowerCase();
            var reStackedTag = reCache[stackedTag] || (reCache[stackedTag] = new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i'));
            var rest$1 = html.replace(reStackedTag, function (all, text, endTag) {
                endTagLength = endTag.length;
                if (!isPlainTextElement(stackedTag) && stackedTag !== 'noscript') {
                    text = text
                        .replace(/<!\--([\s\S]*?)-->/g, '$1') // #7298
                        .replace(/<!\[CDATA\[([\s\S]*?)]]>/g, '$1');
                }
                if (shouldIgnoreFirstNewline(stackedTag, text)) {
                    text = text.slice(1);
                }
                if (options.chars) {
                    options.chars(text);
                }
                return ''
            });
            index += html.length - rest$1.length;
            html = rest$1;
            parseEndTag(stackedTag, index - endTagLength, index);
        }

        if (html === last) {
            options.chars && options.chars(html);
            if (!stack.length && options.warn) {
                options.warn(("Mal-formatted tag at end of template: \"" + html + "\""), {
                    start: index + html.length
                });
            }
            break
        }
    }

    // Clean up any remaining tags
    parseEndTag();

    function advance(n) {
        index += n;
        html = html.substring(n);
    }

    function parseStartTag() {
        var start = html.match(startTagOpen);
        if (start) {
            var match = {
                tagName: start[1],
                attrs: [],
                start: index
            };
            advance(start[0].length);
            var end, attr;
            while (!(end = html.match(startTagClose)) && (attr = html.match(dynamicArgAttribute) || html.match(attribute))) {
                attr.start = index;
                advance(attr[0].length);
                attr.end = index;
                match.attrs.push(attr);
            }
            if (end) {
                match.unarySlash = end[1];
                advance(end[0].length);
                match.end = index;
                return match
            }
        }
    }

    function handleStartTag(match) {
        var tagName = match.tagName;
        var unarySlash = match.unarySlash;

        if (expectHTML) {
            if (lastTag === 'p' && isNonPhrasingTag(tagName)) {
                parseEndTag(lastTag);
            }
            if (canBeLeftOpenTag$$1(tagName) && lastTag === tagName) {
                parseEndTag(tagName);
            }
        }

        var unary = isUnaryTag$$1(tagName) || !!unarySlash;

        var l = match.attrs.length;
        var attrs = new Array(l);
        for (var i = 0; i < l; i++) {
            var args = match.attrs[i];
            var value = args[3] || args[4] || args[5] || '';
            var shouldDecodeNewlines = tagName === 'a' && args[1] === 'href' ?
                options.shouldDecodeNewlinesForHref :
                options.shouldDecodeNewlines;
            attrs[i] = {
                name: args[1],
                value: decodeAttr(value, shouldDecodeNewlines)
            };
            if (options.outputSourceRange) {
                attrs[i].start = args.start + args[0].match(/^\s*/).length;
                attrs[i].end = args.end;
            }
        }

        if (!unary) {
            stack.push({
                tag: tagName,
                lowerCasedTag: tagName.toLowerCase(),
                attrs: attrs,
                start: match.start,
                end: match.end
            });
            lastTag = tagName;
        }

        if (options.start) {
            options.start(tagName, attrs, unary, match.start, match.end);
        }
    }

    function parseEndTag(tagName, start, end) {
        var pos, lowerCasedTagName;
        if (start == null) {
            start = index;
        }
        if (end == null) {
            end = index;
        }

        // Find the closest opened tag of the same type
        if (tagName) {
            lowerCasedTagName = tagName.toLowerCase();
            for (pos = stack.length - 1; pos >= 0; pos--) {
                if (stack[pos].lowerCasedTag === lowerCasedTagName) {
                    break
                }
            }
        } else {
            // If no tag name is provided, clean shop
            pos = 0;
        }

        if (pos >= 0) {
            // Close all the open elements, up the stack
            for (var i = stack.length - 1; i >= pos; i--) {
                if (i > pos || !tagName &&
                    options.warn
                ) {
                    options.warn(
                        ("tag <" + (stack[i].tag) + "> has no matching end tag."), {
                            start: stack[i].start,
                            end: stack[i].end
                        }
                    );
                }
                if (options.end) {
                    options.end(stack[i].tag, start, end);
                }
            }

            // Remove the open elements from the stack
            stack.length = pos;
            lastTag = pos && stack[pos - 1].tag;
        } else if (lowerCasedTagName === 'br') {
            if (options.start) {
                options.start(tagName, [], true, start, end);
            }
        } else if (lowerCasedTagName === 'p') {
            if (options.start) {
                options.start(tagName, [], false, start, end);
            }
            if (options.end) {
                options.end(tagName, start, end);
            }
        }
    }
}

```

### 第 9541 行至第 9914 行

**Regular Expressions** // 相关正则

*   createASTElement // Convert HTML string to AST.
*   parse // !important

parse 函数从 9593 行至 9914 行，共三百多行。核心吗？当然核心！

引自 wikipedia：

> 在计算机科学和语言学中，语法分析（英语：syntactic analysis，也叫 parsing）是根据某种给定的形式文法对由单词序列（如英语单词序列）构成的输入文本进行分析并确定其语法结构的一种过程。
> 
> 语法分析器（parser）通常是作为编译器或解释器的组件出现的，它的作用是进行语法检查、并构建由输入的单词组成的数据结构（一般是语法分析树、抽象语法树等层次化的数据结构）。语法分析器通常使用一个独立的词法分析器从输入字符流中分离出一个个的“单词”，并将单词流作为其输入。实际开发中，语法分析器可以手工编写，也可以使用工具（半）自动生成。

**parse 的整体流程实际上就是先处理了一些传入的options，然后执行了parseHTML 函数，传入了template，options和相关钩子。**

具体实现这里盗一个图：

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="660"></svg>)

parse中的语法分析可以看[这一篇这一节](https://juejin.im/post/5d09a4fef265da1b6b1cd96b#heading-13)

1.  start
2.  char
3.  comment
4.  end

parse、optimize、codegen的核心思想解读可以看[这一篇这一节](https://juejin.im/post/5cfc6ad9e51d4558936aa04d#heading-6)

这里实现的细节还真不少！

### 阶段小结（重点）

噫嘘唏！来到第 20 篇的小结！来个图镇一下先！

还记得官方这样的一句话吗？

> 下图展示了实例的生命周期。你不需要立马弄明白所有的东西，不过随着你的不断学习和使用，它的参考价值会越来越高。

看了这么多，我们再回头看看注释版。

![注释版](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="548" height="1280"></svg>)

[link](http://www.shangdixinxi.com/detail-1105609.html)

上图值得一提的是：**Has "template" option?** 这个逻辑的细化

> 碰到是否有 template 选项时，会询问是否要对 template 进行编译：即模板通过编译生成 AST，再由 AST 生成 Vue 的渲染函数，渲染函数结合数据生成 Virtual DOM 树，对 Virtual DOM 进行 diff 和 patch 后生成新的UI。

如图（此图前文也有提到，见 0 至 5000 行总结）：

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="796"></svg>)

将 Vue 的源码的“数据监听”、“虚拟 DOM”、“Render 函数”、“组件编译”、结合好，则算是融会贯通了！

**一图胜万言**

![](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="696" height="772"></svg>)

好好把上面的三张图看懂，便能做到“成竹在胸”，走遍天下的 VUE 原理面试都不用慌了。框架就在这里，细化的东西就需要多多记忆了！

### 第 9916 行至第 10435 行

🙌 到 1w 行了，自我庆祝一下！

*   processRawAttrs // parse 方法里用到的工具函数 用于将特性保存到AST对象的attrs属性上
*   processElement// parse 方法工具函数 元素填充

```javascript
export function processElement (
  element: ASTElement,
  options: CompilerOptions
) {
  processKey(element)

  // determine whether this is a plain element after
  // removing structural attributes
  element.plain = (
    !element.key &&
    !element.scopedSlots &&
    !element.attrsList.length
  )

  processRef(element)
  processSlotContent(element)
  processSlotOutlet(element)
  processComponent(element)
  for (let i = 0; i < transforms.length; i++) {
    element = transforms[i](element, options) || element
  }
  processAttrs(element)
  return element
}

```

可以看到主要函数包括：processKey、processRef、processSlotContent、processSlotOutlet、processComponent、processAttrs 和最后遍历执行的transforms。

processElement完成的slotTarget的赋值，这里则是将所有的slot创建的astElement以对象的形式赋值给currentParent的scopedSlots。以便后期组件内部实例话的时候可以方便去使用vm.$$slot。

*   processKey
*   processRef

1.  首先最为简单的是processKey和processRef,在这两个函数处理之前，我们的key属性和ref属性都是保存在astElement上面的attrs和attrsMap，经过这两个函数之后，attrs里面的key和ref会被干掉，变成astElement的直属属性。
    
2.  探讨一下slot的处理方式，我们知道的是，slot的具体位置是在组件中定义的，而需要替换的内容又是组件外面嵌套的代码，Vue对这两块的处理是分开的。
    

先说组件内的属性摘取，主要是slot标签的name属性，这是processSlotOutLet完成的。

*   processFor
*   parseFor
*   processIf
*   processIfConditions
*   findPrevElement
*   addIfCondition
*   processOnce
*   processSlotContent
*   getSlotName
*   processSlotOutlet

```javascript
// handle <slot/> outlets
function processSlotOutlet (el) {
  if (el.tag === 'slot') {
    el.slotName = getBindingAttr(el, 'name') // 就是这一句了。
    if (process.env.NODE_ENV !== 'production' && el.key) {
      warn(
        `\`key\` does not work on <slot> because slots are abstract outlets ` +
        `and can possibly expand into multiple elements. ` +
        `Use the key on a wrapping element instead.`,
        getRawBindingAttr(el, 'key')
      )
    }
  }
}
// 其次是摘取需要替换的内容，也就是 processSlotContent，这是是处理展示在组件内部的slot，但是在这个地方只是简单的将给el添加两个属性作用域插槽的slotScope和 slotTarget，也就是目标slot。

```

*   processComponent // processComponent 并不是处理component，而是摘取动态组件的is属性。 processAttrs是获取所有的属性和动态属性。
*   processAttrs
*   checkInFor
*   parseModifiers
*   makeAttrsMap

这一部分仍是衔接这 parse function 里的具体实现：start、end、comment、chars四大函数。

流程再回顾一下：

一、普通标签处理流程描述

1.  识别开始标签，生成匹配结构match。

const match = { // 匹配startTag的数据结构 tagName: 'div', attrs: \[ { 'id="xxx"','id','=','xxx' }, ... \], start: index, end: xxx }  2. 处理attrs，将数组处理成 {name:'xxx',value:'xxx'} 3. 生成astElement，处理for,if和once的标签。 4. 识别结束标签，将没有闭合标签的元素一起处理。 5. 建立父子关系，最后再对astElement做所有跟Vue 属性相关对处理。slot、component等等。

二、文本或表达式的处理流程描述。

1.  截取符号<之前的字符串，这里一定是所有的匹配规则都没有匹配上，只可能是文本了。
2.  使用chars函数处理该字符串。
3.  判断字符串是否含有delimiters，默认也就是${},有的话创建type为2的节点，否则type为3.

三、注释流程描述

1.  匹配注释符号。
2.  使用comment函数处理。
3.  直接创建type为3的节点。

[参考 link](https://juejin.im/post/5d01b954f265da1bbf69172e)

### 阶段小结

parseHTML() 和 parse() 这两个函数占了很大的篇幅，值得重点去看看。的确也很多细节，一些正则的匹配，字符串的操作等。从宏观上把握从 template 到 vnode 的 parse 流程也无大问题。

### 第 10437 行至第 10605 行

*   isTextTag // function chars() 里的工具函数
*   isForbiddenTag // function parseHTML() 用到的工具函数用于检查元素标签是否合法（不是保留命名）
*   guardIESVGBug // parse start() 中用到的工具函数
*   checkForAliasModel // checkForAliasModel用于检查v-model的参数是否是v-for的迭代对象
*   preTransformNode // preTransformNode 方法对el进行预处理，便于后续对标签上的指令和属性进行处理，然后进行树结构的构建，确定el的root, parent, children等属性。总结下来就是生成树节点，构建树结构(关联树节点)。
*   cloneASTElement // 转换属性，把数组属性转换成对象属性，返回对象 AST元素
*   text // 为虚拟dom添加textContent 属性
*   html // 为虚拟dom添加innerHTML 属性
*   baseOptions

```javascript
var baseOptions = {
  expectHTML: true, //标志 是html
  modules: modules$1, //为虚拟dom添加staticClass，classBinding，staticStyle，styleBinding，for，
                      //alias，iterator1，iterator2，addRawAttr ，type ，key， ref，slotName
                      //或者slotScope或者slot，component或者inlineTemplate ，plain，if ，else，elseif 属性
  directives: directives$1, //根据判断虚拟dom的标签类型是什么？给相应的标签绑定 相应的 v-model 双数据绑定代码函数，
                            //为虚拟dom添加textContent 属性，为虚拟dom添加innerHTML 属性
  isPreTag: isPreTag, // 判断标签是否是 pre
  isUnaryTag: isUnaryTag, // 匹配标签是否是area,base,br,col,embed,frame,hr,img,input,
                          // isindex,keygen, link,meta,param,source,track,wbr
  mustUseProp: mustUseProp,
  canBeLeftOpenTag: canBeLeftOpenTag,// 判断标签是否是 colgroup,dd,dt,li,options,p,td,tfoot,th,thead,tr,source
  isReservedTag: isReservedTag, // 保留标签 判断是不是真的是 html 原有的标签 或者svg标签
  getTagNamespace: getTagNamespace, // 判断 tag 是否是svg或者math 标签
  staticKeys: genStaticKeys(modules$1) // 把数组对象 [{ staticKeys:1},{staticKeys:2},{staticKeys:3}]连接数组对象中的 staticKeys key值，连接成一个字符串 str=‘1,2,3’
};

```

*   genStaticKeysCached

### 第 10607 行至第 10731 行

```javascript
/**
  * Goal of the optimizer: walk the generated template AST tree
  * and detect sub-trees that are purely static, i.e. parts of
  * the DOM that never needs to change.
  *
  * Once we detect these sub-trees, we can:
  *
  * 1. Hoist them into constants, so that we no longer need to
  *    create fresh nodes for them on each re-render;
  * 2. Completely skip them in the patching process.
  */
  // 优化器的目标:遍历生成的模板AST树检测纯静态的子树，即永远不需要更改的DOM。
  // 一旦我们检测到这些子树，我们可以:
  // 1。把它们变成常数，这样我们就不需要了
  // 在每次重新渲染时为它们创建新的节点;
  // 2。在修补过程中完全跳过它们。

```

*   optimize // !important:过 parse 过程后，会输出生成 AST 树，接下来需要对这颗树做优化。即这里的 optimize // 循环递归虚拟node，标记是不是静态节点 // 根据node.static或者 node.once 标记staticRoot的状态
*   genStaticKeys$1
*   markStatic$1 // 标准静态节点
*   markStaticRoots // 标注静态根（重要）
*   isStatic // isBuiltInTag（即tag为component 和slot）的节点不会被标注为静态节点，isPlatformReservedTag（即平台原生标签，web 端如 h1 、div标签等）也不会被标注为静态节点。
*   isDirectChildOfTemplateFor

### 阶段小结

简单来说：整个 optimize 的过程实际上就干 2 件事情，markStatic(root) 标记静态节点 ，markStaticRoots(root, false) 标记静态根节点。

那么被判断为静态根节点的条件是什么？

1.  该节点的所有子孙节点都是静态节点（判断为静态节点要满足 7 个判断，[详见](https://juejin.im/post/5d4e3eb7f265da03c61e411e)）
    
2.  必须存在子节点
    
3.  子节点不能只有一个纯文本节点
    

其实，markStaticRoots()方法针对的都是普通标签节点。表达式节点与纯文本节点都不在考虑范围内。

markStatic()得出的static属性，在该方法中用上了。将每个节点都判断了一遍static属性之后，就可以更快地确定静态根节点：通过判断对应节点是否是静态节点 且 内部有子元素 且 单一子节点的元素类型不是文本类型。

> 只有纯文本子节点时，他是静态节点，但不是静态根节点。静态根节点是 optimize 优化的条件，没有静态根节点，说明这部分不会被优化。

Q：为什么子节点的元素类型是静态文本类型，就会给 optimize 过程加大成本呢？

A：optimize 过程中做这个静态根节点的优化目是：在 patch 过程中，减少不必要的比对过程，加速更新。但是需要以下成本

1.  维护静态模板的存储对象 一开始的时候，所有的静态根节点 都会被解析生成 VNode，并且被存在一个缓存对象中，就在 Vue.proto.\_staticTree 中。 随着静态根节点的增加，这个存储对象也会越来越大，那么占用的内存就会越来越多 势必要减少一些不必要的存储，所有只有纯文本的静态根节点就被排除了
    
2.  多层render函数调用 这个过程涉及到实际操作更新的过程。在实际render 的过程中，针对静态节点的操作也需要调用对应的静态节点渲染函数，做一定的判断逻辑。这里需要一定的消耗。
    

纯文本直接对比即可，不进行 optimize 将会更高效。

[参考link](https://juejin.im/post/5e7595c75188252c1f22524a)

### 第 10733 行至第 10915 行

// KeyboardEvent.keyCode aliases

*   keyCodes // 内置按键
*   keyNames
*   genGuard // genGuard = condition => `if(${condition})return null;`
*   modifierCode //m odifierCode生成内置修饰符的处理
*   genHandlers
*   genHandler // 调用genHandler处理events\[name\]，events\[name\]可能是数组也可能是独立对象，取决于name是否有多个处理函数。
*   genKeyFilter // genKeyFilter用于生成一段过滤的字符串：
*   genFilterCode // 在 genKeyFilter 里被调用
*   on
*   bind$1
*   baseDirectives // CodegenState 里的工具函数

不管是组件还是普通标签，事件处理代码都在genData的过程中，和之前分析原生事件一致，genHandlers用来处理事件对象并拼接成字符串。

### 第 10921 行至第 11460 行

// generate(ast, options)

```javascript
export function generate (
  ast: ASTElement | void,
  options: CompilerOptions
): CodegenResult {
  const state = new CodegenState(options)
  const code = ast ? genElement(ast, state) : '_c("div")'
  return {
    render: `with(this){return ${code}}`,
    staticRenderFns: state.staticRenderFns
  }
}

```

*   CodegenState
*   generate // ！important
*   genElement

```javascript
export function genElement (el: ASTElement, 
state: CodegenState): string {
  if (el.parent) {
    el.pre = el.pre || el.parent.pre
  }

  if (el.staticRoot && !el.staticProcessed) {
    // 如果是一个静态的树， 如 <div id="app">123</div>
    // 生成_m()方法
    // 静态的渲染函数被保存至staticRenderFns属性中
    return genStatic(el, state)
  } else if (el.once && !el.onceProcessed) {
    // v-once 转化为_o()方法
    return genOnce(el, state)
  } else if (el.for && !el.forProcessed) {
    // _l()
    return genFor(el, state)
  } else if (el.if && !el.ifProcessed) {
    // v-if 会转换为表达式
    return genIf(el, state)
  } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
    // 如果是template，处理子节点
    return genChildren(el, state) || 'void 0'
  } else if (el.tag === 'slot') {
    // 如果是插槽，处理slot
    return genSlot(el, state)
  } else {
    // component or element
    let code
    // 如果是组件，处理组件
    if (el.component) {
      code = genComponent(el.component, el, state)
    } else {
      let data
      if (!el.plain || (el.pre && state.maybeComponent(el))) {
        data = genData(el, state)
      }

      const children = el.inlineTemplate ? null : genChildren(el, state, true)
      code = `_c('${el.tag}'${
        data ? `,${data}` : '' // data
      }${
        children ? `,${children}` : '' // children
      })`
    }
    // module transforms
    for (let i = 0; i < state.transforms.length; i++) {
      code = state.transforms[i](el, code)
    }
    return code
  }
}


```

*   genStatic // genStatic会将ast转化为\_m()方法
*   genOnce // 如果v-once在v-for中，那么就会生成\_o()方法， 否则将其视为静态节点
*   genIf // genIf会将v-if转换为表达式，示例如下
*   genIfConditions
*   genFor // v-for会转换为\_l()
*   genData$2
*   genDirectives // genData() 里调用
*   genInlineTemplate // genData() 里调用
*   genScopedSlots // genData() 里调用
*   genScopedSlot
*   genChildren // 处理子节点
*   getNormalizationType // 用于判断是否需要规范化
*   genNode // 处理 Node
*   genText // 处理 Text
*   genComment
*   genSlot // 处理插槽
*   genComponent // 处理组件
*   genProps // 处理 props
*   transformSpecialNewlines

这里面的逻辑、细节太多了，不做赘述，有兴趣了解的童鞋可以去看[推荐阅读](https://juejin.im/post/5ec666fd6fb9a047d112616b)

### 阶段小结

generate方法内部逻辑还是很复杂的，**但仅做了一件事情，就是将ast转化为render函数的字符串，形成一个嵌套结构的方法，模版编译生成的\_c(),\_m(),\_l等等其实都是生成vnode的方法**，在执行vue.$mount方法的时候，会调用vm.\_update(vm.\_render(), hydrating)方法，此时\_render()中方法会执行生成的render()函数，执行后会生成vnode，也就是虚拟dom节点。

### 第 11466 行至第 11965 行

*   prohibitedKeywordRE // 正则校验：禁止关键字
*   unaryOperatorsRE // 正则校验：一元表达式操作
*   stripStringRE // 正则校验：脚本字符串
*   detectErrors // 检测错误工具函数
*   checkNode // 检查 Node
*   checkEvent // 检查 Event
*   checkFor // 检查 For 循环
*   checkIdentifier // 检查 Identifier
*   checkExpression // 检查表达式
*   checkFunctionParameterExpression // 检查函数表达式
*   generateCodeFrame
*   repeat$1
*   createFunction // 构建函数
*   createCompileToFunctionFn // 构建编译函数
*   compile // !important

```javascript
return function createCompiler (baseOptions) {
function compile (
    template,
    options
) {
    var finalOptions = Object.create(baseOptions);
    var errors = [];
    var tips = [];

    var warn = function (msg, range, tip) {
    (tip ? tips : errors).push(msg);
    };

    if (options) {
    if (options.outputSourceRange) {
        // $flow-disable-line
        var leadingSpaceLength = template.match(/^\s*/)[0].length;

        warn = function (msg, range, tip) {
        var data = { msg: msg };
        if (range) {
            if (range.start != null) {
            data.start = range.start + leadingSpaceLength;
            }
            if (range.end != null) {
            data.end = range.end + leadingSpaceLength;
            }
        }
        (tip ? tips : errors).push(data);
        };
    }
    // merge custom modules
    if (options.modules) {
        finalOptions.modules =
        (baseOptions.modules || []).concat(options.modules);
    }
    // merge custom directives
    if (options.directives) {
        finalOptions.directives = extend(
        Object.create(baseOptions.directives || null),
        options.directives
        );
    }
    // copy other options
    for (var key in options) {
        if (key !== 'modules' && key !== 'directives') {
        finalOptions[key] = options[key];
        }
    }
    }

    finalOptions.warn = warn;

    var compiled = baseCompile(template.trim(), finalOptions);
    {
    detectErrors(compiled.ast, warn);
    }
    compiled.errors = errors;
    compiled.tips = tips;
    return compiled
}

```

再看这张图，对于“模板编译”是不是有一种新的感觉了。

![](https://user-gold-cdn.xitu.io/2020/7/6/17323830402969ff?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   compileToFunctions

// 最后的最后

```javascript
    return Vue;

```

哇！历时一个月左右，我终于完成啦！！！

完结撒花🎉🎉🎉！激动 + 释然 + 感恩 + 小满足 + ...... ✿✿ヽ(°▽°)ノ✿

这生啃给我牙齿都啃酸了！！

总结
--

emmm，本来打算再多修补一下，但是看到 vue3 的源码解析已有版本出来啦（扶朕起来，朕还能学），时不我待，Vue3 奥利给，干就完了！

后续仍会完善此文，您的点赞是我最大的动力！也望大家不吝赐教，不吝赞美~

最最最最后，还是那句老话，与君共勉：

> 纸上得来终觉浅 绝知此事要躬行

