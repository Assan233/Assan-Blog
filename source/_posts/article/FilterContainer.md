---
title: 高级搜索组件（FilterContainer）
date: 2020-05-01 23:10:26
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

FilterContainer 文档


一、事件:
1. @change: filter-item的change事件触发, 会触发change事件

返回参数:  filterData, value, key, index



2. @filterClear: 点击重置 filterData 按钮时, 触发 filterClear 事件

返回参数:  值为空的filterData



二、配置:
(*)为必填

1. dataJson: 表单元素的配置详情(Array)
注:如果dataJson中有异步获取的内容(如option),建议将dataJson定义为计算属性, 这样依赖更新的时候, dataJson也会自动更新

类型: Array

1) label: 表单元素label

类型: String

2) key(*): 表单元素绑定的key

类型: String

3) type(*): 表单element一级组件名, 共有10种, 如下 表1.

类型: String

4) optionType: 表单元素的子组件类型为button时, 如el-checkbox-button/ el-radio-button

类型: String, 枚举,  [button] 

5) isAdvanced: 是否为高级表单元素

类型: Boolean,  true/是

6) config: 表单元素一级的element UI 配置项,配置项与ele文档相同

类型: Object

7) options: 表单元素二级的elementUI配置项, 如果需要设置option分组, 可以options嵌套options, 如<el-select>的option分组

类型: Array 

8) itemWidth: 自定义表单元素的宽度

类型: Number, 单位px

9) optionConfig: 二级补充配置项, 如果需要自定义el-option、el-radio、el-checkbox的 label、value、name的key, 需要声明key的名称 。如果 options 数组元素为非对象, 则不需要配置optionConfig

类型: Object

labelKey(String): option的label自定义key名

valueKey(String): option的value自定义key名

nameKey(String):  option的name自定义key名

 		optionConfig: {
            labelKey: "",
            valueKey: "",
            nameKey: ""
        }


2.filterData: 表单元素过滤对象
类型: Object

由过滤表单元素的key组成的对象, 使用了双向数据流, 修改 filterData 会同步组件内的表单元素数据, 可以通过修改 filterData 来设置表单元素的默认值.

注意:  由于 filterData 使用了双向数据流, 所以父组件传值时, 需要使用.sync修饰符

	<filter-container
      :data-json="dataJson"
      :title-info="titleInfo"
      :filter-data.sync="filterParams"
      :span="[12, 12]"
      @change="handleFilterChange"
    >



3. titleInfo: 标题的配置项
类型: Object

title: 标题

summary: 说明内容

4. span: 组件栅格占比
类型: Array, [left, right]

5. hasClear: 是否需要重置按钮
类型: Boolean, true/是



表1:

配置项	配置内容
一级	二级








type


 (表单element
 组件名, 共有10种)

el-radio-group	el-radio
el-radio-button
el-checkbox-group	el-checkbox-button
el-checkbox
el-input	
el-input-number	
el-select	el-option 
el-option-group
el-cascader	
el-switch	
el-time-select	
el-time-picker	
el-date-picker	


 三、Slot
1. operateSlot: 

插槽名: operateSlot

筛选区域右侧slot, 一般是操作类按钮, 可通过v-slot: operateSlot ="scope"(缩写为: #operateSlot) , 获取当前filterData

注: 插槽内容需要加上公共过滤样式类 filter-item, 以保持排版风格统一

2. formSlot: 

插槽名: formSlot

筛选表单slot, 可通过v-slot: formSlot ="scope" (缩写为: #formSlot), 获取当前filterData

3. titleSlot:

插槽名: titleSlot

标题slot,  例如标题的刷新按钮, v-slot: titleSlot(缩写为: #titleSlot)

  

四、注意事项
Radio单选
1. 配置radio的options时, 需要注意配置label为radio的绑定值, name为radio的名称(见Ele Radio Attributes文档)

Checkbox 多选
1. 配置Checkbox的options时, 需要注意配置label为Checkbox的绑定值, name为Checkbox的名称(见Ele Checkbox Attributes文档)

Cascader 级联选择器
1. 根据ele文档, options应该在config配置



五、代码范例


<template>
	<filter-container
      :span="[6, 18]"
      :data-json="dataJson"
	  :filter-data.sync="filterData"
      :title-info="titleInfo"
      @change="fetchDataList"
    >
      <template #operateSlot="scope">
        <el-button class="filter-item" type="primary" @click="doAddAlarm()"
          >新增预警</el-button
        >
      </template>
    </filter-container>
</template>
data() {
	titleInfo: {
        title: "预警管理"
    },
	filterData:{
		alarmCategory: ""
	}
},
computed: {
	dataJson() {
      const config = [
        {
          label: "预警类型",
          key: "alarmCategory",
          type: "el-select",
          config: { clearable: true },
          options: this.alarmCategoryList
        }
      ];
      return config;
    }

}
methods: {
fetchDataList(){
	const {alarmCategory} = this,filterData;
	fetchDataListApi(alarmCategory); //请求数据
 
}
 
}











