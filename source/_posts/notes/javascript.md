---
title: JavaScript 笔记
date: 2020-06-02 16:37:36
summary: JavaScript 笔记
categories: 笔记
tags: JavaScript
top: 
cover: 
img:
keywords: JavaScript
---

## 1. 触发事件Dom元素获取
``` javascript
    ev = ev || event;
    let srcEle =  ev.currentTarget
```

## 2. 深拷贝的实现
``` javascript
    function deepClone(obj){
        let objClone = Array.isArray(obj)?[]:{};
        if(obj && typeof obj==="object"){
            for(key in obj){
                if(obj.hasOwnProperty(key)){
                    //判断ojb子元素是否为对象，如果是，递归复制
                    if(obj[key]&&typeof obj[key] ==="object"){
                        objClone[key] = deepClone(obj[key]);
                    }else{
                        //如果不是，简单复制
                        objClone[key] = obj[key];
                    }
                }
            }
        }
        return objClone;
    }   
```
## 3. Object.assign() 实现浅拷贝
``` javascript
	Object.assign(target, resource) // 将源对象的属性拷贝到目标对象。但是无法对源对象的引用指针进行深拷贝。
```
## 4. let在循环体内的表现
	let声明的变量, 在循环体内每循环一次, 就会创建一个新的作用域,所以可以依次获取 i:
``` javascript
    for (let i = 0; i < 10 ; i++) {
    	setTimeout(function() {console.log(i); }, 100 * i);
    }
    /*
    0
    1
    2
    3
    4
    5
    6
    7
    8
    9
    */
```

## 5. event对象属性解析
	
   * bubbles	返回布尔值，指示事件是否是起泡事件类型。
   * cancelable	返回布尔值，指示事件是否可拥可取消的默认动作。
   * currentTarget	返回其事件监听器触发该事件的元素。
   * eventPhase	返回事件传播的当前阶段。
   * target	返回触发此事件的元素（事件的目标节点）。
   * timeStamp	返回事件生成的日期和时间。
   * type	返回当前 Event 对象表示的事件的名称。
   * initEvent()	初始化新创建的 Event 对象的属性。
   * preventDefault()	通知浏览器不要执行与事件关联的默认动作。
   * stopPropagation()	不再派发事件。

##  6. 跳出循环的三种方法(break, return, continue)

* break: 是立即结束语句，并跳出语句，结束for循环。
* continue: 不是退出一个循环，而是开始循环的一次新迭代。
* return: 停止函数。
  *forEach等循环语句无法用以上三种中止退出, 但是可以报错中止遍历。*

## 7. 闭包
``` javascript
    function count() {
        var arr = [];
        for (var i=1; i<=3; i++) {
            arr.push(function () {
                return i * i;
            });
        }
        return arr;
    }

    var results = count();
    var f1 = results[0];
    var f2 = results[1];
    var f3 = results[2];
```

在上面的例子中，每次循环，都创建了一个新的函数，然后，把创建的3个函数都添加到一个Array中返回了。
你可能认为调用f1()，f2()和f3()结果应该是1，4，9，但实际结果是：

``` javascript
f1(); // 16
f2(); // 16
f3(); // 16
```

全部都是16！原因就在于返回的函数引用了变量i，但它并非立刻执行。等到3个函数都返回时，它们所引用的变量i已经变成了4，因此最终结果为16。
返回闭包时牢记的一点就是：返回函数不要引用任何循环变量，或者后续会发生变化的变量。
如果一定要引用循环变量怎么办？方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变：

``` javascript
    function count() {
        var arr = [];
        for (var i=1; i<=3; i++) {
            arr.push((function (n) {
                return function () {
                    return n * n;
                }
            })(i));
        }
        return arr;
    }

    var results = count();
    var f1 = results[0];
    var f2 = results[1];
    var f3 = results[2];

    f1(); // 1
    f2(); // 4
    f3(); // 9
```

## 8. 类数组对象(array-like objects)的迭代问题

引用 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Indexed_collections)
>一些 JavaScript 对象, 例如 document.getElementsByTagName() 返回的 NodeList 或者函数内部可用的 arguments 对象，他们表面上看起来，外观和行为像数组，但是不共享他们所有的方法。例如 arguments 对象就提供一个 length 属性，但是不实现 forEach() 方法。

Array的原生(prototype)方法可以用来处理类似数组行为的对象，例如：
```javascript
    Array.prototype.forEach.call(arguments, function(item) {
        console.log(item);
    });
  ```
## 9. Array和Set的对比
一般情况下，在JavaScript中使用数组来存储一组元素，而新的集合对象有这些优势：

+ 数组中用于判断元素是否存在的indexOf 函数效率低下。
+ Set对象允许根据值删除元素，而数组中必须使用基于下标的 splice 方法。
+ 数组的indexOf方法无法找到NaN值。
+ Set对象存储不重复的值，所以不需要手动处理包含重复值的情况。

# 10. Object和Map的比较
一般地，objects会被用于将字符串类型映射到数值。Object允许设置键值对、根据键获取值、删除键、检测某个键是否存在。而Map具有更多的优势。

+ Object的键均为Strings类型，在Map里键可以是任意类型。
+ 必须手动计算Object的尺寸，但是可以很容易地获取使用Map的尺寸。
+ Map的遍历遵循元素的插入顺序。
+ Object有原型，所以映射中有一些缺省的键。（可以用 map = Object.create(null) 回避）。

**这三条提示可以帮你决定用Map还是Object：**

+ 如果键在运行时才能知道，或者所有的键类型相同，所有的值类型相同，那就使用Map。
+ 如果需要将原始值存储为键，则使用Map，因为Object将每个键视为字符串，不管它是一个数字值、布尔值还是任何其他原始值。
+ 如果需要对个别元素进行操作，使用Object。

## 11. Object.prototype.toString.call 判断数据类型
```javascript
    console.log(Object.prototype.toString.call({})) // [object Object]
    console.log(Object.prototype.toString.call(123)) // [object Number]
    console.log(Object.prototype.toString.call('123')) // [object String]
    console.log(Object.prototype.toString.call(undefined)) // [object Undefined]
    console.log(Object.prototype.toString.call(true)) // [object Boolean]
    console.log(Object.prototype.toString.call([])) // [object Array]
    console.log(Object.prototype.toString.call(function(){})) // [object Function]
    console.log(Object.prototype.toString.call(null))  // [object Null]
```

## 12. 动态加载静态资源
由于webpack获取资源依赖的时候, 无法获取动态路径, 所以需要使用 require() 手动请求资源
``` html
 <img :src="require(`./img/${currentEditUnit.url}`)" />
```
