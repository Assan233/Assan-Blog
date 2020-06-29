---
title: Echarts3.0 中利用 dataset 进行数据管理
date: 2020-06-02 18:57:28
summary: Echarts3.0 中利用 dataset 进行数据管理,并封装图表生成方法
categories: 原创
tags: 
- Echarts
- JavaScript
top:
cover:
img:
keywords:
---

## 效果图


![clipboard.png](/img/bVbt8OB)


## JS

```javascript
let rst = [
            {
                name: 'Matcha Latte',
                data:[
                    {time: '2012',num: 365},
                    {time: '2013',num: 815},
                    {time: '2014',num: 665},
                    {time: '2015',num: 565},
                ]
            },{
                name: 'Milk Tea',
                data:[
                    {time: '2012',num: 265},
                    {time: '2013',num: 615},
                    {time: '2014',num: 465},
                    {time: '2015',num: 965},
                ]
            },{
                name: 'Cheese Cocoa',
                data:[
                    {time: '2012',num: 765},
                    {time: '2013',num: 215},
                    {time: '2014',num: 765},
                    {time: '2015',num: 165},
                ]
            }
        ];
        

        let chartOption = {
            el:'#chart',//放置图表的元素css选择器
            title: '市场饮料销售情况',//图表标题
            unit: '吨',
            dataArr: rst,
            
        }

        
        /**
         * chtOption = {
         *  el:'', //放置图表的元素css选择器
            title: '', //图表大标题
            unit: [], //单位
         * }
         * 
         * */
        function barChart (chtOption){
            let myChart = echarts.init(document.querySelector('#chart'));
            let dataObj = {
                series: [],//系列数据
                xData:[],//x轴数据
                yData:[],//类目数据
                source: [],
                chartType: [],
                
            }
            for(let i in chtOption.dataArr[0].data){
                dataObj.series.push(chtOption.dataArr[0].data[i].time);
            }
            for(let i in chtOption.dataArr){
                let perSeries = [];
                perSeries.push(chtOption.dataArr[i].name);
                for(let j in chtOption.dataArr[i].data){
                    perSeries.push(chtOption.dataArr[i].data[j].num);
                    
                }
                dataObj.xData.push(perSeries);
                dataObj.yData.push(chtOption.dataArr[i].name);
            }
            let dataSeries =  ['name_value'];
            for(let i in dataObj.series){
                dataSeries.push(dataObj.series[i]);
                dataObj.chartType.push({type: 'bar'});
            }
            dataObj.source.push(dataSeries);
            for(let i in dataObj.xData){
                dataObj.source.push(dataObj.xData[i]);
            }
            let option = {
                title: {
                    text: chtOption.title,
                    textAlign: 'left'
                },
                tooltip: {
                    trigger: 'axis',
                    axisPointer: {
                        type: 'shadow'
                    }
                },
                legend: {
                    data: chtOption.series
                },
                grid: {
                    left: '3%',
                    right: '12%',
                    bottom: '3%',
                    top: '10%',
                    containLabel: true
                },
                dataset: {
                    source: dataObj.source
                },
                xAxis: [
                    { gridIndex: 0,name: '单位: ' + chtOption.unit}
                ],
                yAxis: [
                    {type: 'category',gridIndex: 0}
                ],
                series: dataObj.chartType
            };

            myChart.setOption(option);
        }
        
        barChart(chartOption);
```
