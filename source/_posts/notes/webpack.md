---
title: Webpack 笔记
date: 2019-06-16 17:52:20
summary: Webpack 笔记
categories: 笔记
tags: 
- Webpack
- JavaScript
top:
cover:
img:
keywords:
---

##  1. webpack打包和webpack-dev-server所做的事情
1.webpack打包是将所有资源打包到指定的配置路径内,即打包的结果
2.webpack-dev-server是将所有资源加载在内存中, 所以即使删除webpack打包的文件夹,也不会影响由webpack-dev-server加载的页面
3.安装插件时，--save-dev是你开发时候依赖的东西，--save是你发布之后依赖的东西。-g表示全局安装，--save-dev 可以简写为 -D ，--save 可以简写为 -S ，npm install 可以简写为 npm i。

##  2. export 和 export default 的区别
+ export 相当于导出对象变量, 导入使用的时候可以使用解构
+ export default相当于导出一个集合, 导入使用需要使用一个变量名声明(名称不能是集合里的key名称), 导入使用不可以使用解构