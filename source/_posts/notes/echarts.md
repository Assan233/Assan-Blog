---
title: Echarts 笔记
date: 2020-06-02 17:43:55
summary: Echarts 笔记
categories: 笔记
tags: 
- Echarts
- 数据可视化
top:
cover:
img:
keywords: Echarts
---

## 1. 进度条底色实现
### series[i]-bar.barGap 
[ default: 30% ]
不同系列的柱间距离，为百分比（如 '30%'，表示柱子宽度的 30%）。

如果想要两个系列的柱子重叠，可以设置 barGap 为 '-100%'。这在用柱子做背景的时候有用。

在同一坐标系上，此属性会被多个 'bar' 系列共享。此属性应设置于此坐标系中最后一个 'bar' 系列上才会生效，并且是对此坐标系中所有 'bar' 系列生效。
