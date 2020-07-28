---
title: 封装Vue组件的原则及技巧
date: 2020-04-30 22:39:42
summary: 封装组件，除了对于API的熟练度之外，更多地是考察JavaScript基础。Vue入门十分轻松，但是要写好优雅的Vue代码，也是一份不小的学问。
categories: 转载
tags: 
- Vue
- 组件封装
top: true
cover: true
img:
keywords: 
- Vue
- 组件封装
---

Vue的组件系统
--------

Vue组件的API主要包含三部分：prop、event、slot

*   [props](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcomponents-props.html)表示组件接收的参数，最好用对象的写法，这样可以针对每个属性设置类型、默认值或自定义校验属性的值，此外还可以通过type、validator等方式对输入进行验证
*   [slot](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcomponents-slots.html)可以给组件动态插入一些内容或组件，是实现高阶组件的重要途径；当需要多个插槽时，可以使用具名slot
*   [event](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcomponents.html%23%25E7%259B%2591%25E5%2590%25AC%25E5%25AD%2590%25E7%25BB%2584%25E4%25BB%25B6%25E4%25BA%258B%25E4%25BB%25B6)是子组件向父组件传递消息的重要途径

### 单向数据流

参考:[单向数据流-官方文档](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcomponents-props.html%23%25E5%258D%2595%25E5%2590%2591%25E6%2595%25B0%25E6%258D%25AE%25E6%25B5%2581)。

> 父级 prop 的更新会向下流动到子组件中，但是反过来则不行

单向数据流是Vue组件一个非常明显的特征，不应该在子组件中直接修改props的值

*   如果传递的prop仅仅用作展示，不涉及修改，则在模板中直接使用即可
*   如果需要对prop的值进行转化然后展示，则应该使用computed计算属性
*   如果prop的值用作初始化，应该定义一个子组件的data属性并将prop作为其初始值

从源码`/src/core/vdom/create-component.js`和`/src/core/vdom/helpers/extract-props.js`里可以看见，在处理props的取值时，首先从

```javascript
function extractPropsFromVNodeData(){  
    const res = {}  const { attrs, props } = data  // 执行浅拷贝  
    checkProp(res, props, key, altKey, true) || checkProp(res, attrs, key, altKey,  false)   
    return res
} 
```

在子组件修改props，却不会修改父组件，这是因为`extractPropsFromVNodeData`中是通过**浅复制**将attrs传递给props的。

浅复制意味着在子组件中对**对象和数组**的props进行修改还是会影响父组件，这就违背了单向数据流的设计。因此需要避免这种情况出现。

### 组件之间的通信

这里可以参考:[vue组件通信全揭秘](https://juejin.im/post/5bd97e7c6fb9a022852a71cf)，写的比较全面

*   父子组件的关系可以总结为**prop 向下传递，事件event向上传递**
*   祖先组件和后代组件（跨多代）的数据传递，可以使用[provide和inject](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fapi%2F%23provide-inject)来实现

此外，如果需要跨组件或者兄弟组件之间的通信，可以通过eventBus或者vuex等方式来实现。

“绕开”单向数据流
---------

考虑下面场景：父组件将数据通过prop形式传递给子组件，子组件进行相关操作并修改数据，需要修改父组件的prop值（一个典型的例子是：购物车的商品数量counter组件）。

根据组件单向数据流和和事件通信机制，需要由子组件通过事件通知父组件，并在父组件中修改原始的prop数据，完成状态的更新。在子组件中修改父组件的数据的场景在业务中也是比较常见的，那么有什么办法可以“绕开”单向数据流的限制呢?

### 状态提升

可以参考React的状态提升，直接通过props将父元素的数据处理逻辑传入子组件，子组件只做数据展示和事件挂载即可

```html
<template>
    <div class="counter">
        <div class="counter_btn" @click="onMinus">-</div>
        <div class="counter_val">{{value}}</div>
        <div class="counter_btn" @click="onPlus">+</div>
    </div>
</template>
 
<script>
    export default {
        props: {
            value: {
                type: Number,
                default: 0
            },
            onMinus: Function,
            onPlus: Function
        },
    };
</script>
```

然后在调用时传入事件处理函数

```html
<template>
    <div>
        <counter :value="counter2Val" :on-minus="minusVal" :on-plus="plusVal"></counter>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                counter2Val: 0,
            }
        },
        methods: {
            minusVal(){
                this.counter2Val--
            },
            plusVal(){
                this.counter2Val++
            }
        }
    }
</script>

```

很明显，由于在每个父组件中都需要实现`on-minus`和`on-plus`，因此状态提升并没有从根本上解决问题。

### v-model语法糖

Vue内置了`v-model`指令，v-model 是一个语法糖，可以拆解为 props: value 和 events: input。就是说组件只要提供一个名为 value 的 prop，以及名为 input 的自定义事件，满足这两个条件，使用者就能在自定义组件上使用`v-model`

```html

<template>
  <div>
    <button @click="changeValue(-1)">-1</button>
    <span>{{currentVal}}</span>
    <button @click="changeValue(1)">+1</button>
  </div>
</template>
 
<script>
export default {
  props: {
    value: {
      type: Number // 定义value属性
    }
  },
  data() {
    return {
      currentVal: this.value
    };
  },
  methods: {
    changeVal(val) {
      this.currentVal += parseInt(val);
      this.$emit("input", this.currentVal); // 定义input事件
    }
  }
};
</script>
```

然后调用的时候只需要传入v-model指令即可

```javascript
<counter v-model="counerVal"/>
```

使用v-model，可以很方便地在子组件中同步父组件的数据。在2.2之后的版本中，可以定制`v-model`指令的prop和event名称，参考[model配置项](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fapi%2F%23model)

```javascript
export default {    model: {        prop: 'value',        event: 'input'    },    // ... }
```

获得组件实例的引用
---------

在开发组件中，获取组件实例是一个非常有用的方法。组件可以通过`$refs`、`$parents`、`$children`等方式获得vm实例引用

*   `$refs`在组件（或者dom上）增加ref属性即可
    
*   `$parents`获取子组件挂载的父组件节点
    
*   `$children`，获取组件的所有子节点
    

这些接口返回的都是vnode，可以通过`vnode.componentInstance`获得对应的组件实例，然后直接调用组件的方法或访问数据。虽然这种方式多多少少有些违背组件的设计理念，增加了组件之间的耦合成本，但代码实现会更加简洁。

### 表单验证组件

通常情况下，表单验证是表单提交前一个十分常见的应用场景。那么，如何把表单验证的功能封装在组件内部呢？

下面是一个表单组件的示例，展示了通过获得组件的引用来实现表单验证功能。

首先定义组件的使用方式，

*   `xm-form`接收`model`和`rule`两个prop
    *   `model`表示表单绑定的数据对象，最后表单提交的就是这个对象
    *   `rule`表示验证规则策略，表单验证可以使用[async-validator](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fyiminghe%2Fasync-validator)插件
*   `xm-form-item`接收的`prop`属性，对应form组件的model和rule的某个key值，根据该key从model上取表单数据，从rule上取验证规则

下面是使用示例代码

```html
<template>
    <div class="page">
        <xm-form :model="form" :rule="rule" ref="baseForm">
            <xm-form-item label="姓名" prop="name">
                <input v-model="form.name"/>
            </xm-form-item>
            <xm-form-item label="邮箱" prop="email">
                <input v-model="form.email"/>
            </xm-form-item>
            <xm-form-item>
                <button @click="submit">提交</button>
            </xm-form-item>
        </xm-form>
    </div>
</template>
 
<script>
    import xmForm from "../src/form/form"
    import xmFormItem from "../src/form/form-item"
 
    export default {
        components: {
            xmForm,
            xmFormItem,
        },
        data() {
            return {
                form: {
                    name: "",
                    email: ""
                },
                rule: {
                    name: [
                        {required: true, message: '用户名不能为空', trigger: 'blur'}
                    ],
                    email: [
                        {required: true, message: '邮箱不能为空', trigger: 'blur'},
                        {type: 'email', message: '邮箱格式不正确', trigger: 'blur'}
                    ],
                }
            }
        },
        methods: {
            submit() {
                // 调用form组件的validate方法
                this.$refs.baseForm.validate().then(res => {
                    console.log(res)
                }).catch(e => {
                    console.log(e)
                })
            }
        }
    }
</script>

```

接下来让我们实现form-item组件，其主要作用是放置表单元素，及展示错误信息

```html
<template>
    <label class="form-item">
        <div class="form-item_label">{{label}}</div>
        <div class="form-item_mn">
            <slot></slot>
        </div>
        <div class="form-item_error" v-if="errorMsg">{{errorMsg}}</div>
    </label>
</template>
<script>
    export default {
        name: "form-item",
        props: {
            label: String,
            prop: String
        },
        data() {
            return {
                errorMsg: ""
            }
        },
        methods: {
            showError(msg) {
                this.errorMsg = msg
            }
        }
    }
</script>
```

然后让我们来实现form组件

*   通过`calcFormItems`获取每个`xm-form-item`的引用，保存在formItems中
*   暴露validate接口，内部调用AsyncValidator，并根据结果遍历formItems中每个表单元素的`prop`属性，处理对应的error信息

```html

<template>
    <div class="form">
        <slot></slot>
    </div>
</template>
 
<script>
    import AsyncValidator from 'async-validator';
 
    export default {
        name: "xm-form",
        props: {
            model: {
                type: Object
            },
            rule: {
                type: Object,
                default: {}
            }
        },
        data() {
            return {
                formItems: []
            }
        },
        mounted() {
            this.calcFormItems()
        },
        updated() {
            this.calcFormItems()
        },
        methods: {
            calcFormItems() {
                // 获取form-item的引用
                if (this.$slots.default) {
                    let children = this.$slots.default.filter(vnode => {
                        return vnode.tag &&
                            vnode.componentOptions && vnode.componentOptions.Ctor.options.name === 'form-item'
                    }).map(({componentInstance}) => componentInstance)
 
                    if (!(children.length === this.formItems.length && children.every((pane, index) => pane === this.formItems[index]))) {
                        this.formItems = children
                    }
                }
            },
            validate() {
                let validator = new AsyncValidator(this.rule);
 
                let isSuccess = true
 
                let findErrorByProp = (errors, prop) => {
                    return errors.find((error) => {
                        return error.field === prop
                    }) || ""
                }
 
                validator.validate(this.model, (errors, fields) => {
                    this.formItems.forEach(formItem => {
                        let prop = formItem.prop
                        let error = findErrorByProp(errors || [], prop)
                        if (error) {
                            isSuccess = false
                        }
 
                        formItem.showError(error && error.message || "")
                    })
                });
 
                return Promise.resolve(isSuccess)
            }
        }
    }
</script>
```

这样我们就完成了一个通用的表单验证组件。从这个例子中可以看出获取组件引用，在组件开发中是一个非常有用的方法。

封装API组件
-------

一些组件如提示框、弹出框等，更适合单独的API调用方式，如

```javascript
import MessageBox from '@/components/MessageBox.vue'MessageBox.toast('hello)
```

如何实现制这种不需要手动嵌入模板里面的组件呢？原来，除了在通过在模板中嵌入组件到children挂载组件，Vue还为组件提供了手动挂载的方法[$mount](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fapi%2Findex.html%23vm-mount)

```javascript
let component = new MessageBox().$mount()document.getElementById('app').appendChild(component.$el)
```

通过这种方式，我们就是可以封装API形式调用组件，下面是一个alert消息提示的接口封装

### 消息弹窗组件

一个消息组件就是在页面指定绘制展示提示消息的组件，下面是简单实现

```html
<template>
    <div class="alert">
        <div class="alert-main" v-for="item in notices" :key="item.name">
            <div class="alert-content">{{ item.content }}</div>
        </div>
    </div>
</template>
 
<script>
    let seed = 0;
 
    function getUuid() {
        return 'alert_' + (seed++);
    }
 
    export default {
        data() {
            return {
                notices: []
            }
        },
        methods: {
            add(notice) {
                const name = getUuid();
 
                let _notice = Object.assign({
                    name: name
                }, notice);
 
                this.notices.push(_notice);
 
                // 定时移除，单位：秒
                const duration = notice.duration;
                setTimeout(() => {
                    this.remove(name);
                }, duration * 1000);
            },
            remove(name) {
                const notices = this.notices;
 
                for (let i = 0; i < notices.length; i++) {
                    if (notices[i].name === name) {
                        this.notices.splice(i, 1);
                        break;
                    }
                }
            }
        }
    }
</script>

```

下面来实现消息组件挂载到页面的逻辑，并对外暴露展示消息的接口

```javascript
import Vue from 'vue';
 
// 具体的组件
import Alert from './alert.vue';
Alert.newInstance = properties => {
    const props = properties || {};
	// 实例化一个组件，然后挂载到body上
    const Instance = new Vue({
        data: props,
        render (h) {
            return h(Alert, {
                props: props
            });
        }
    });
    const component = Instance.$mount();
    document.body.appendChild(component.$el);
	// 通过闭包维护alert组件的引用
    const alert = Instance.$children[0];
    return {
        // Alert组件对外暴露的两个方法
        add (noticeProps) {
            alert.add(noticeProps);
        },
        remove (name) {
            alert.remove(name);
        }
    }
};
 
// 提示单例
let messageInstance;
function getMessageInstance () {
    messageInstance = messageInstance || Alert.newInstance();
    return messageInstance;
}
function notice({ duration = 1.5, content = '' }) {
    // 等待接口调用的时候再实例化组件，避免进入页面就直接挂载到body上
    let instance = getMessageInstance();
    instance.add({
        content: content,
        duration: duration
    });
}
 
// 对外暴露的方法
export default {
    info (options) {
        return notice(options);
    }
}

```

然后就可以使用API的方式来调用弹窗组件了

```javascript
import alert from './alert.js'
// 直接使用
alert.info({content: '消息提示', duration: 2})
// 或者挂载到Vue原型上
Vue.prototype.$Alert = alert
// 然后在组件中使用
this.$Alert.info({content: '消息提示', duration: 2})

```

高阶组件
----

高阶组件可以看做是函数式编程中的**组合**。可以把高阶组件看做是一个函数，他接收一个组件作为参数，并返回一个功能增强的组件。

高阶组件是一个接替Mixin实现抽象组件公共功能的方法，不会因为组件的使用而污染DOM（添加并不想要的div标签等）、可以包裹任意的单一子元素等等

在React中高阶组件是比较常用的组件封装形式，在Vue中如何实现高阶组件呢？

在组件的render函数中，只需要返回一个vNode数据类型即可，如果在render函数中提前做一些处理，并返回`this.$slots.default[0]`对应的vnode，就可以实现高阶组件。

### 内置的keep-alive

Vue内置了一个高阶组件`keep-alive`，查看[源码](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fblob%2Fdev%2Fsrc%2Fcore%2Fcomponents%2Fkeep-alive.js)可以发现其实现原理，就是通过维护一个cache，并在render函数中根据key返回缓存的vnode，来实现组件的持久化。

### throttle

节流是web开发中处理事件比较常见的需求。常见的场景有及时搜索框避免频繁触发搜索接口、表单按钮防止在短暂时间误重复提交等

首先来看看Throttle组件的使用方式，接收两个props

*   `time`表示节流的时间间隔
*   `events`表示需要处理的事件名，多个事件用逗号分隔

在下面的例子中，通过Throttle组件来控制其内部button的点击事件，此时连续点击多次，触发clickBtn的次数要比点击的次数小（节流函数通过一个定时器进行处理）。

```html
 <template>
    <div>
        <Throttle :time="1000" events="click">
            <button @click="clickBtn">click {{count}}</button>
        </Throttle>
    </div>
</template>

```

下面是具体实现，实现高阶组件的主要功能是在render函数中对当前插槽中的vnode进行处理

```javascript
const throttle = function (fn, wait = 50, ctx) {
    let timer
    let lastCall = 0
    return function (...params) {
        const now = new Date().getTime()
        if (now - lastCall < wait) return
        lastCall = now
        fn.apply(ctx, params)
    }
}
 
export default {
    name: 'throttle',
    abstract: true,
    props: {
        time: Number,
        events: String,
    },
    created() {
        this.eventKeys = this.events.split(',')
        this.originMap = {}
        this.throttledMap = {}
    },
    // render函数直接返回slot的vnode，避免外层添加包裹元素
    render(h) {
        const vnode = this.$slots.default[0]
        this.eventKeys.forEach((key) => {
            const target = vnode.data.on[key]
            if (target === this.originMap[key] && this.throttledMap[key]) {
                vnode.data.on[key] = this.throttledMap[key]
            } else if (target) {
                // 将原本的事件处理函数替换成throttle节流后的处理函数
                this.originMap[key] = target
                this.throttledMap[key] = throttle(target, this.time, vnode)
                vnode.data.on[key] = this.throttledMap[key]
            }
        })
        return vnode
    },
}

```

我们还可以进一步封装，通过debounce函数来实现Debounce组件，可见高阶组件的作用，就是为了增强某个组件而存在的。关于高阶组件的其他应用，可以参考[HOC(高阶组件)在vue中的应用](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fcoolriver%2Fcoolriver-site%2Fblob%2Fmaster%2Fmarkdown%2Fvue-mixin-hoc.md)。

小结
--

本文整理了几种实现Vue组件的技巧

*   以counter计数器组件为例，展示了通过v-model语法糖同步父子组件的方式
*   以表单验证组件为例，展示了通过获取子组件的实例来封装组件的方法
*   以全局弹窗组件为例，展示了手动mount挂载组件封装API组件的方式
*   以throttle节流组件为例，展示了在vue中一种实现高阶组件的方式

在了解Vue的API之后，理解上面的概念都比较轻松，封装组件，除了对于API的熟练度之外，更多地是考察JavaScript基础。Vue入门十分轻松，但是要写好优雅的Vue代码，也是一份不小的学问。

