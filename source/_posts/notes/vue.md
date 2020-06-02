---
title: Vue 笔记
date: 2020-06-02 16:48:55
summary: Vue 笔记及踩过的坑
categories: 笔记
tags: Vue
top: 
cover: 
img:
keywords: 
- Vue
- JavaScript
---

## 1. vue 中替换数组内容的方法$set
``` javascript
for(let i in self.checked){
	self.$set(self.checked, i, false);
}
```

## 2. vue 通过splice操作数组并更新
splice(index,len,[item])它也可以用来替换/删除/添加数组内某一个或者几个值（该方法会改变原始数组）

* index:数组开始下标       
* len: 替换/删除的长度, 添加的话为0       
* item:替换的值，删除操作的话 item为空

### 删除
``` javascript
//删除起始下标为1，长度为1的一个值(len设置1，如果为0，则数组不变)
var arr = ['a','b','c','d'];
arr.splice(1,1);
console.log(arr);  
//['a','c','d']; 

//删除起始下标为1，长度为2的一个值(len设置2)
var arr2 = ['a','b','c','d']
arr2.splice(1,2);
console.log(arr2); 
//['a','d']
```
 

### 替换
``` javascript
//替换起始下标为1，长度为1的一个值为‘ttt’，len设置的1
var arr = ['a','b','c','d'];
arr.splice(1,1,'ttt');
console.log(arr);        
//['a','ttt','c','d'] 

//替换起始下标为1，长度为2的两个值为‘ttt’，len设置的1
var arr2 = ['a','b','c','d'];
arr2.splice(1,2,'ttt');
console.log(arr2);       
//['a','ttt','d'] 
```
 

### 添加
``` javascript
//在下标为1处添加一项'ttt'
var arr = ['a','b','c','d'];
arr.splice(1,0,'ttt');
console.log(arr);        
//['a','ttt','b','c','d'] 
```

## 3. element UI 清空表单
``` javascript
// this.$options.data().formData 是初始数据
this.formData = this.$options.data().formData;
```

## 4. element v-for数组表单验证的方法
要在每个item设置 :rules和 :prop

### item
``` html
<el-form-item
     label="名称"
     :prop="`labelContent.${index}.chineseName`"
     :rules="formRules.labelContentChineseName"
   >
     <el-input
       v-model="formData.labelContent[index].chineseName"
       :disabled="dialogType === 'edit'"
       placeholder="请输入标签名称"
     ></el-input>
   </el-form-item>
```

### rules
``` javascript
formRules: {
     chineseName: [
       { required: true, message: "请输入标注集名称", trigger: "blur" },
       { validator: validateExist, trigger: "blur" }
     ],
     labelContentChineseName: [
       { required: true, message: "请输入标签名称", trigger: "blur" }
     ],
     labelContentEnglishName: [
       { required: true, message: "请输入标签英文", trigger: "blur" }
     ],
     wordSeparator: [
       { required: true, message: "请选择或输入分隔符", trigger: "blur" }
     ],
     filePath: [{ required: true, message: "请上传文件", trigger: "blur" }],
     englishName: [
       { required: true, message: "请输入标注集英文", trigger: "blur" },
       { validator: validateEng, trigger: "blur" }
     ]
   },
```

## 5. ele使用scrollBar组件隐藏横向滚动条
``` css
.el-scrollbar__wrap{
	overflow-x: hidden;
}
```

## 6. 重用相同组件的不同路由, 无法切换
开发人员经常遇到的情况是，多个路由解析为同一个Vue组件。问题是，Vue出于性能原因，默认情况下共享组件将不会重新渲染，如果你尝试在使用相同组件的路由之间进行切换，则不会发生任何变化。
``` javascript
const routes = [
  {
    path: "/a",
    component: MyComponent
  },
  {
    path: "/b",
    component: MyComponent
  },
];
```
如果你仍然希望重新渲染这些组件，则可以通过在 router-view 组件中提供 :key 属性来实现。
``` html
<template>
    <router-view :key="$route.path"></router-view>
</template>
```
## 7. 把所有Props传到子组件很容易
这是一个非常酷的功能，可让你将所有 props 从父组件传递到子组件。如果你有另一个组件的包装组件，这将特别方便。所以，与其把所有的 props 一个一个传下去，你可以利用这个，把所有的 props 一次传下去：
``` html
<template>
  <childComponent v-bind="$props" />
</template>
```
代替：

``` html
<template>
  <childComponent :prop1="prop1" :prop2="prop2" :prop="prop3" :prop4="prop4" ... />
</template>
```

## 8. 自定义 v-model
默认情况下，v-model 是 @input 事件侦听器和 :value 属性上的语法糖。但是，你可以在你的Vue组件中指定一个模型属性来定义使用什么事件和value属性——非常棒！
``` javascript
export default: {
  model: {
    event: 'change',
    prop: 'checked'  
  }
}
```

## 9. Prop自定义验证器
验证 Props 是 Vue 中的基本做法之一。

你可能已经知道可以将props验证为原始类型，例如字符串，数字甚至对象。你也可以使用自定义验证器——例如，如果你想验证一个字符串列表：
``` javascript
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```

## 10. 计算属性传参

通过闭包实现
``` javascript
:data="closure(item, itemName, blablaParams)"

computed: {
 closure () {
   return function (a, b, c) {
        /** do something */
        return data
    }
 }
}
```