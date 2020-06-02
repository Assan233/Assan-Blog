---
title: javascript 笔记
date: 2020-06-02 16:37:36
summary: javascript 笔记
categories: 笔记
tags: javascript
top: 
cover: 
img:
keywords: javascript
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
	Object.assign(target, resouce) // 将源对象的属性拷贝到目标对象。但是无法对源对象的引用指针进行深拷贝。
```
## 4. let在循环体内的表现
	let声明的变量, 在循环体内每循环一次, 就会创建一个新的作用域,所以可以实现:
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

## 7. 闭包注意点
``` javascript
function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push(function () {
            return I * I;
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