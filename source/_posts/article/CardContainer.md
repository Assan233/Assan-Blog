---
title: 卡片列表组件 (CardContainer)
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

CardContainer文档


一、 事件:
pagination:

翻页事件

返回参数: page, pageSize

cardClick:

列表 item 点击事件

返回参数: item

二、 配置:
1. cardInfo

数据结构包含：列表请求后端返回的数据 和 单页数据量(pageSize)，含有表格数据和翻页数据：

 前端组 > 列表组件（TableContainer） > image2020-5-21 19:6:58.png

cardInfo = {
   data: [], //表格数据
   currentPage: 1, //当前页数
   totalPages: 0, //总页数
   totalCounts: 0,  //数据总数
   pageSize: 10	//单页数据量
}


类型: Object

(1) currentPage(Number): 当前页数

(2) totalCounts(Number): 数据总数

(3) totalPages(Number): 总页数

(4) data(Array): 表格数据

注意: 由于 cardInfo 使用了双向数据流, 所以, cardInfo 需要使用.sync修饰符, 组件内的 cardInfo 会和业务层同步.



 	<card-container
      :card-info.sync="cardInfo"
      :column-config="columnConfig"
      @pagination="fetchTableList"
    >



2. rowConfig

卡片列表 Ele Row Attributes 配置信息,  配置项详见 Ele文档

类型: Object

3. colConfig:

卡片列表 Ele Col Attributes 配置信息,  配置项详见 Ele文档

类型: Object

4.   pageConfig

翻页配置信息

类型: Object

(1)     limit (Number): 单页数量

(2)     pageSizes (Array): 单页数量数组

(3)     layout (String): 需要显示el-pagination组件

(4)     autoScroll (Boolean):是否自动滚动到顶部

(5)     isShow(Boolean): 是否需要显示翻页组件, 默认显示

三、 Slot插槽
插槽名: itemSlot
使用: #itemSlot="scope"

     2. 插槽返回数据 scope:



scope = {
  item: "", //当前卡片数据
  index: "" // 当前卡片索引值
};







