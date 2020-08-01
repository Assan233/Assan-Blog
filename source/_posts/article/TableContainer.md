---
title: 二次封装Element UI表格组件
date: 2020-08-01 23:10:26
summary: 
categories: 原创
tags:
- Element UI
- el-table
- Vue
- JavaScript
top: true
cover: true
img:
keywords:
- Element UI
- el-table
- Vue
- JavaScript
---

# TableContainer 文档

## 支持的功能
1. 完整支持 Element UI Table 组件所有配置项
2. 支持多种单元格内容格式化， 包括 时间格式化、存储单位格式化、标签格式化、单位格式化，还可以通过 配置自定义文本格式化函数，来实现自定义文本格式化的全支持。
3. 支持分页，分页数据自动同步到业务层
4. 表格支持单元格插槽
5. 可以多选（表格带复选框）

## 一、 事件:
注: 支持 elementUI Table 组件所有事件, 返回参数与 elementUI Table Events 文档一致 

### 1. select
当用户手动勾选数据行的 Checkbox 时触发的事件
返回参数: selection, row

### 2. select-all
当用户手动勾选全选 Checkbox 时触发的事件
返回参数: selection

### 3.selection-change
当选择项发生变化时会触发该事件
返回参数: selection

### 4. pagination
翻页事件
返回参数: page, pageSize

## 二、 配置:
### 1. tableInfo
数据结构包含：列表请求后端返回的数据 和 单页数据量(pageSize)，含有表格数据和翻页数据：
``` javascript
tableInfo = {
   data: [], //表格数据
   currentPage: 1, //当前页数
   totalPages: 0, //总页数
   totalCounts: 0,  //数据总数
   pageSize: 10	//单页数据量
}
```
类型: Object
 (1) **currentPage**(Number): 当前页数
 (2) **totalCounts**(Number): 数据总数
 (3) **totalPages**(Number): 总页数
 (4) **data**(Array): 表格数据

**注意: 由于tableInfo使用了双向数据流, 所以, tableInfo 需要使用.sync修饰符, 组件内的 tableInfo 会和业务层同步.**
``` html
 	<table-container
      :table-info.sync="tableInfo"
      :column-config="columnConfig"
      @pagination="fetchTableList"
    >
```

### 2. tableConfig
表格Table Attributes Ele配置信息(不配置data,  data改为外部 tableInfo一起配置)
类型: Object
**注意: 如果需要配置 Table Attributes 的自定义class, 如 header-cell-class-name, 则class 对应的css样式需要使用 /deep/ 样式穿透. 这一点和 elementUI 表现一致.**
``` html
<table-container header-row-class-name = "header-class" >
<style scoped lang="scss">
   /deep/.header-class {
     font-size: 14px;
     color: #333;
   }
</style>
```

### 3. columnConfig
列数据配置信息
类型: Array
 #### (1) config(Object)
 Table-column Ele配置内容(包括prop,label)
 #### (2) status(Object)
单元状态色格式化
**method**(Function): 单元状态色格式化函数, 最终返回  [ info, primary, danger, success, warning]
*注意: 形参确保单元格数据是第一个参数*
**params**(Array):除单元格数据(value), 格式化所需要的参数,
#### (3) isSlot(Boolean)
是否需要使用插槽自定义列数据, 使用prop作为插槽名
#### (4) formate(Object)
列数据格式化内容
+ **type**(String): 格式化类别(5种),枚举   [ time, memory, unit, tag, customize]
+ **defaultValue**(String): 自定义数据为空的默认值, 默认为 ”--”
 *备注:   时间/存储单位格式化已有数据为空默认值,不可修改,此配置无效*
+ **method**(Function): 文本自定义格式化函数(格式化类型为 customize 和 tag 时, 可以自定义格式化文本)
 *注意:  形参确保单元格数据是第一个参数*
+ **params**(Array): 除单元格数据(value), 格式化所需要的参数, 如下表:

| type | params |
| :------ | ------: |
| unit	| 单位格式化,如 个/次| 
| customize	| 自定义格式化参数| 
| time	| 需要转化的时间格式,如 hh-mm-ss| 
| memory	| 内存格式化| 
| tag	| tag自定义文本格式化, 最终返回 数组 或 单文本| 

*注：如果表格数据是多维对象, 可使用 customize 自定义格式化文本, prop 绑定的key名称为该key所在对象单位一级key。*

### 4.   pageConfig
翻页配置信息
类型: Object
+ **limit** (Number): 单页数量
+ **pageSizes** (Array): 单页数量数组
+ **layout** (String): 需要显示el-pagination组件
+ **autoScroll** (Boolean):是否自动滚动到顶部
+ **isShow**(Boolean): 是否需要显示翻页组件, 默认显示

### 5. tableStyle
自定义表格包裹层样式, 可用于弹窗列表消除组件自带padding
类型: Object

### 6. 如果需要调用执行element UI Table 组件方法, 需要获取渲染的表格数据(如 toggleRowSelection(row. true) ), 可通过关键字 tableData 获取组件的表格数据.
```html
 	<table-container
      ref="tableContainerEle"
      :table-info.sync="tableInfo"
      :column-config="columnConfig"
 	></table-container>
console.log(this.$refs.tableContainerEle.tableData)
 ```

### 7.  如果需要获取 TableContainer 组件内的 el-table 组件实例, 可通过关键字 tableEle 获取 el-table 组件实例
```html
 	<table-container
      ref="tableContainerEle"
      :table-info.sync="tableInfo"
      :column-config="columnConfig"
 	></table-container>
console.log(this.$refs.tableContainerEle.tableEle)
 ```

## 三、 Slot插槽
 ## 1. 插槽名: 配置项 prop
      使用: v-slot: [propName]="scope"(缩写为: #[propName]="scope") 
      slot插槽使用prop作为插槽名, 使用插槽的的场景一般是:
        (1)     含有操作列
  *注意: 由于slot插槽 使用prop作为插槽名, 所以操作列记得配置prop*
        (2)     列数据有特殊格式化需求(不含tag ,tag可在fromate处理)
        (3)     列数据有业务操作

  ## 2. 插槽返回数据 scope
```js
scope = {
  td: "", //当前单元格数据
  row: "", //当前行数据
  index: "" // 当前行索引值
};

```
## 四、代码范例
```html
<template>
	<table-container
      v-loading="isTableLoading"
      :table-info.sync="tableInfo"
      :column-config="columnConfig"
      @pagination="fetchDataList"
    >
	// 操作-插槽
      <template #opration="scope">
        <el-button
          v-if="!scope.row.alarmStatus"
          type="primary"
          plain
          size="mini"
          @click="handleRun(scope.row, true)"
          >启 动</el-button
        >
      </template>
    </table-container>
</template>
<script>
export default {
  data() {
	return {
    // table-container配置项
    tableInfo: {
      data: null,
      totalCounts: 0,
      currentPage: 1,
      totalPages: 0,
	    pageSize: 10
    }
}
computed: {
    columnConfig() {
      const config = [
        {
          config: {
            prop: "alarmItem",
            label: "预警项"
          }
        },
        {
          config: {
            prop: "alarmCategory",
            label: "预警类别"
          }
        },
        {
          config: {
            prop: "alarmStatus",
            label: "预警状态"
          },
          status: {
            method: function(val) {
              const rst = val ? "success" : "danger";
              return rst;
            }
          },
          formate: {
            type: "customize",
            method: function(val) {
              const rst = val ? "启动" : "关闭";
              return rst;
            }
          }
        },
        {
          isSlot: true,
          config: {
            label: "操作",
            prop: "opration",
            width: 310
          }
        }
      ];
      return config;
    }
  },
  methods: {
	 // 获取表格数据
    async fetchDataList() {
      const { currentPage, pageSize } = this.tableInfo;
      const params = {
        page: currentPage,
        pageSize: pageSize,
        ...this.filterParams
      };
      const res = await alarmManageApi.query(params);
      const { code, object, msg, errorMsg } = res.data;

      if (code === 0) {
        this.tableInfo = { ...object };
      } else {
        this.$notify({
          title: "错误",
          message: errorMsg || msg,
          type: "error"
        });
      }
    }
  }
}
</script>
```
