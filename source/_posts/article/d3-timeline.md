---
title: D3.js 实现带伸缩时间轴拓扑图
date: 2020-06-30 22:53:44
summary: D3.js 实现带伸缩时间轴拓扑图
categories: 原创
tags: 
- D3.js
img:
keywords: 
- D3.js
- 数据可视化
---
## 效果图:
![d3-timeline.png](http://qbnq7wfi1.bkt.clouddn.com/d3-timeline.png)
基于d3-v5, 依赖dagre-d3, 代码:

## Vue版:
```javascript
<template>
  <div :id="rootEleId" class="topology-timeline">
    <!-- 拓扑图tooltip -->
    <transition name="el-zoom-in-top">
      <div v-show="isTooltipVisiable" class="tooltip">
        <div class="time">{{ tipTime }}</div>
        <div class="content">
          <span class="circle"></span>
          <span class="text">{{ tipText }}</span>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import * as d3 from "d3";
import dagreD3 from "dagre-d3";
import UUID from "uuid";
export default {
  name: "TopologyTimeline",
  components: {},
  props: {
    // 画布尺寸
    width: {
      type: Number,
      default: 990
    },
    height: {
      type: Number,
      default: 265
    },
    // 节点大小 (半径)
    size: {
      type: Number,
      default: 8
    },
    // 画布内边控制
    padding: {
      type: Object,
      default: () => {
        return {
          top: 0,
          bottom: 40,
          left: 40,
          right: 40
        };
      }
    },
    // 节点数据
    nodeInfo: {
      type: Array,
      default: () => {
        return [];
      }
    },
    // 箭头方向数据数据
    arrowInfo: {
      type: Array,
      default: () => {
        return [];
      }
    },
    // 节点填充颜色
    fillColor: {
      type: Object,
      default: () => {
        return {
          requirement_release: {
            label: "需求发布",
            color: "#188cff"
          },
          requirement_change: {
            label: "需求变更",
            color: "#ffac27"
          },
          solution_submit: {
            label: "方案提交",
            color: "#9270ca"
          },
          solution_feedback: {
            label: "方案反馈",
            color: "#e8684a"
          },
          project_approval: {
            label: "项目立项",
            color: "#13c2c2"
          },
          solution_checked: {
            label: "方案验收",
            color: "#5d7092"
          },
          solution_success: {
            label: "方案通过",
            color: "#67c23a"
          }
        };
      }
    }
  },
  data() {
    return {
      rootEleId: `topology-timeline-${UUID.v4()}`, //生成全局唯一id
      arrowEleId: `arrow-${UUID.v4()}`, //生成全局唯一箭头元素id
      y0: 0, // 拖拽初始值
      viewY: 0, // view->output当前y偏移值
      isTooltipVisiable: false, // tooltip
      max: null, // 日期最大值
      min: null, // 日期最小值
      nodeMap: null, //节点信息--map
      nodeDomMap: null, //节点dom--map
      timeArr: [], //存储时间
      xScale: null, // x轴比例尺
      formatDay: null, // 坐标轴文本格式化 -- 日
      formatMonth: null, // 坐标轴文本格式化 -- 年/月
      currentHoverInfo: null, // 当前hover节点的数据
      // currentClickInfo: null, // 当前click节点的数据

      // 相关dom元素
      rootEle: null, //组件根元素
      svg: null, //svg画布
      view: null, // 拓扑图
      grid: null, // 网格
      axis: null, // 坐标轴
      separateLine: null, // 坐标轴与拓扑图分隔线
      xAxisDay: null, // 坐标轴 - 日
      xAxisMonth: null // 坐标轴 - 年/月
    };
  },
  computed: {
    // toolTip 时间转换
    tipTime() {
      if (!this.currentHoverInfo) return;
      const time = this.currentHoverInfo.time;
      let date = new Date(time);
      let Y = date.getFullYear();
      let M = date.getMonth() + 1;
      let D = date.getDate();
      let h = date.getHours();
      let m = date.getMinutes();
      let s = date.getSeconds();
      M = M > 9 ? M : "0" + M;
      D = D > 9 ? D : "0" + D;
      m = m > 9 ? m : "0" + m;
      s = s > 9 ? s : "0" + s;
      return `${Y}年${M}月${D}日 ${h}:${m}:${s}`;
    },
    // toolTip 状态label
    tipText() {
      if (!this.currentHoverInfo) return;
      const type = this.currentHoverInfo.progressType.toLowerCase();
      const text = this.fillColor[type].label;
      return text;
    }
  },
  mounted() {
    this.handleInitGraph();
    this.handleCreatAxis();
    this.handleCreatGrid();
    this.handleCreatTopology();
    this.handleZoom();
    this.handleDrag();
    this.onHoverNode();
    this.onClickNode();
  },
  methods: {
    // 初始化视图
    handleInitGraph() {
      // 节点信息转化为map
      this.nodeMap = new Map();
      this.nodeDomMap = new Map();
      this.nodeInfo.forEach(item => {
        this.nodeMap.set(item.nodeId, item);
        this.timeArr.push(item.time);
      });
      // 根元素
      this.rootEle = d3.select(`#${this.rootEleId}`);
      // 创建画布 svg
      this.svg = this.rootEle
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);
      // 初始化元素
      const backgroundGrey = this.svg.append("rect").attr("class", "bg-grey");
      this.grid = this.svg.append("g").attr("class", "grid");
      this.view = this.svg.append("g").attr("class", "view");
      const backgroundWhite = this.svg.append("rect").attr("class", "bg-white");
      this.axis = this.svg.append("g").attr("class", "axis");
      this.separateLine = this.svg
        .append("line")
        .attr("class", "separate-line");

      // 绘制箭头以供引用
      this.svg
        .append("defs")
        .append("marker")
        .attr("id", this.arrowEleId)
        .attr("viewBox", "0 0 10 10")
        .attr("refX", "17")
        .attr("refY", "5")
        .attr("markerWidth", "6")
        .attr("markerHeight", "6")
        .attr("orient", "auto")
        .append("path")
        .attr("d", "M 0 0 L 10 5 L 0 10 z")
        .style("fill", "#bbbbbb");

      // 添加背景板 rect
      backgroundGrey
        .attr("fill", "#FAFAFA")
        .attr("x", 0)
        .attr("y", 0)
        .attr("width", this.width)
        .attr("height", this.height - this.padding.bottom);
      backgroundWhite
        .attr("fill", "#fff")
        .attr("x", 0)
        .attr("y", this.height - this.padding.bottom)
        .attr("width", this.width)
        .attr("height", this.padding.bottom);
    },
    // 创建坐标轴及确定比例尺
    handleCreatAxis() {
      this.max = new Date(d3.max(this.timeArr));
      this.min = new Date(d3.min(this.timeArr));
      let maxY = this.max.getFullYear();
      let maxM = this.max.getMonth();
      let minY = this.min.getFullYear();
      let minM = this.min.getMonth();
      // console.log(this.max, this.min);
      // 确定比例尺
      this.xScale = d3
        .scaleTime()
        .domain([new Date(minY, minM, 1), new Date(maxY, ++maxM, 1)])
        .range([0, this.width - this.padding.left - this.padding.right]);

      // 坐标轴文本格式化
      this.formatDay = d3.axisBottom(this.xScale).tickFormat(d => {
        const date = new Date(d);
        const day = date.getDate();
        return `${day === 1 ? "" : day}`; // 如果是1号, 不显示刻度,直接由xAxisMonth显示年月
      });
      this.formatMonth = d3
        .axisBottom(this.xScale)
        .ticks(d3.timeMonth.every(1))
        .tickPadding(6)
        .tickSizeInner(20)
        .tickFormat(d => {
          const date = new Date(d);
          const mon = date.getMonth() + 1;
          const year = date.getFullYear();
          return `${year} - ${mon > 9 ? mon : "0" + mon}`;
        });
      this.axis.attr(
        "transform",
        `translate(${this.padding.left},${this.height - this.padding.bottom})`
      );
      this.xAxisDay = this.axis
        .append("g")
        .attr("class", "axis-day")
        .call(this.formatDay);
      this.xAxisMonth = this.axis
        .append("g")
        .attr("class", "axis-month")
        .call(this.formatMonth);
    },
    // 创建X轴网格
    handleCreatGrid() {
      const monthNum = d3.timeMonth.count(this.min, this.max); // 区间月份数量
      const lineGroup = this.grid
        .attr("transform", `translate(${this.padding.left},0)`)
        .selectAll("g")
        .data(this.xScale.ticks(monthNum))
        .enter()
        .append("g");
      lineGroup
        .append("line")
        .attr("x1", d => {
          return this.xScale(new Date(d));
        })
        .attr("x2", d => {
          return this.xScale(new Date(d));
        })
        .attr("y1", this.padding.top)
        .attr("y2", this.height - this.padding.bottom)
        .attr("class", "grid-line")
        .style("stroke", "#DCDCDC")
        .style("stroke-dasharray", 6);

      // 添加坐标轴与拓扑图分隔线
      this.separateLine
        .style("stroke", "#DCDCDC")
        .style("stroke-width", 2)
        .attr("x1", 0)
        .attr("x2", this.width)
        .attr("y1", this.height - this.padding.bottom)
        .attr("y2", this.height - this.padding.bottom);
    },
    // 绘制拓扑图 节点--箭头
    handleCreatTopology() {
      let g = new dagreD3.graphlib.Graph().setDefaultEdgeLabel(function() {
        return {};
      });
      g.setGraph({
        rankdir: "LR", // 拓扑图方向 L->R
        marginx: 0, // 图边距
        marginy: 0
        // nodesep: 0, // 节点距离
        // ranksep: 0 // 节点步长
      });
      this.nodeInfo &&
        this.nodeInfo.map(item => {
          const type = item.progressType.toLowerCase();
          const color = this.fillColor[type].color;
          g.setNode(item.nodeId, {
            label: "",
            // class: item.progressType.toLowerCase(),
            style: `stroke-width: 2px; stroke: #fff; fill: ${color}`,
            shape: "circle",
            id: item.nodeId
          });
        });

      this.arrowInfo &&
        this.arrowInfo.map(item => {
          g.setEdge(item.from, item.to, {
            arrowheadStyle: "stroke:none; fill: none", //  箭头头部样式
            style: "stroke:none; fill: none" //线条样式
          });
        });
      let render = new dagreD3.render();
      render(
        this.view
          .attr("height", this.height - this.padding.bottom)
          .attr("transform", `translate(${this.padding.left},0)`),
        g
      );

      // 修改节点半径
      this.view
        .select(".nodes")
        .selectAll("circle")
        .attr("r", this.size);

      // 重新定位节点x坐标
      const nodesArr = this.view.select(".nodes").selectAll(".node")._groups[0];
      nodesArr.forEach(item => {
        let dom = d3.select(item)._groups[0][0];
        let nodeId = dom.id;
        let date = this.nodeMap.get(nodeId).time;
        const x = this.xScale(new Date(date));
        const y = dom.transform.animVal[0].matrix.f;
        d3.select(item).attr("transform", `translate(${x},${y})`);
        this.nodeDomMap.set(item.id, item);
      });

      // 重新绘制箭头
      this.arrowInfo &&
        this.arrowInfo.map(item => {
          let fromDom = this.nodeDomMap.get(item.from);
          let toDom = this.nodeDomMap.get(item.to);
          const [x1, y1, x2, y2] = [
            fromDom.transform.animVal[0].matrix.e,
            fromDom.transform.animVal[0].matrix.f,
            toDom.transform.animVal[0].matrix.e,
            toDom.transform.animVal[0].matrix.f
          ];
          this.view
            .select(".edgePaths")
            .append("g")
            .append("line")
            .attr("class", `to-${item.to}`) // 设置唯一的class方便修改路径
            .attr("stroke-width", "2")
            .attr("stroke", "#bbbbbb")
            .style("stroke-dasharray", 8)
            .attr("marker-end", `url(#${this.arrowEleId})`)
            .attr("x1", x1)
            .attr("y1", y1)
            .attr("x2", x2)
            .attr("y2", y2);
        });
    },
    // 缩放控制
    handleZoom() {
      // 设置zoom参数
      let zoom = d3
        .zoom()
        .scaleExtent([1, 10])
        .translateExtent([
          [0, 0],
          [this.width * 1.5, this.height]
        ]) //移动的范围
        .extent([
          [0, 0],
          [this.width, this.height]
        ]); //视窗 （左上方，右下方）

      this.svg.call(zoom.on("zoom", reRender.bind(this)));

      // 每次缩放重定位渲染拓扑图
      function reRender() {
        const t = d3.event.transform.rescaleX(this.xScale); //获得缩放后的比例尺
        this.xAxisDay.call(this.formatDay.scale(t)); //重新设置x坐标轴的scale
        this.xAxisMonth.call(this.formatMonth.scale(t)); //重新设置x坐标轴的scale
        // let { x, y, k } = d3.event.transform;
        // 重新绘制节点
        const nodesArr = this.view.select(".nodes").selectAll(".node")
          ._groups[0];
        nodesArr.forEach(item => {
          let dom = d3.select(item)._groups[0][0];
          let nodeId = dom.id;
          let date = this.nodeMap.get(nodeId).time;
          let x1 = t(new Date(date));
          let y1 = dom.transform.animVal[0].matrix.f;
          d3.select(item).attr("transform", `translate(${x1},${y1})`);
          this.nodeDomMap.set(item.id, item);
        });

        // 重新绘制箭头
        this.arrowInfo &&
          this.arrowInfo.map(item => {
            let fromDom = this.nodeDomMap.get(item.from);
            let toDom = this.nodeDomMap.get(item.to);
            const [x1, y1, x2, y2] = [
              fromDom.transform.animVal[0].matrix.e,
              fromDom.transform.animVal[0].matrix.f,
              toDom.transform.animVal[0].matrix.e,
              toDom.transform.animVal[0].matrix.f
            ];
            this.view
              .select(`.to-${item.to}`)
              .attr("x1", x1)
              .attr("y1", y1)
              .attr("x2", x2)
              .attr("y2", y2);
          });

        //重新绘制x网格
        this.grid
          .selectAll(".grid-line")
          .attr("x1", d => {
            return t(new Date(d));
          })
          .attr("x2", d => {
            return t(new Date(d));
          });
      }
    },
    // 拖拽控制
    handleDrag() {
      let self = this;
      this.view.select(".output").call(
        d3
          .drag()
          .on("start", dragstarted)
          .on("drag", dragged)
          .on("end", dragended)
      );
      function dragstarted() {
        self.y0 = d3.event.y;
        const translateY = self.view.select(".output")._groups[0][0].transform
          .animVal[0];
        // 获取 view->output当前偏移值
        self.viewY = translateY ? translateY.matrix.f : 0;
        self.view
          .select(".output")
          .raise()
          .classed("active", true);
      }

      function dragged() {
        self.view
          .select(".output")
          .attr(
            "transform",
            `translate(0, ${d3.event.y - self.y0 + self.viewY})`
          );
      }

      function dragended() {
        self.view.select(".output").classed("active", false);
      }
    },
    // 当鼠标移动到节点时
    onHoverNode() {
      let self = this;
      self.view.selectAll(".node").on("mouseover", function() {
        const nodeId = this.id; // 不可使用箭头函数, 否则this指向vue
        self.isTooltipVisiable = true;
        self.currentHoverInfo = self.nodeMap.get(nodeId);
        const type = self.currentHoverInfo.progressType.toLowerCase();
        const color = self.fillColor[type].color;
        const x = event.clientX + 6;
        const y = event.clientY + 6;
        self.rootEle.select(".tooltip").style("top", y + "px");
        self.rootEle.select(".tooltip").style("left", x + "px");
        self.rootEle
          .select(".tooltip")
          .select(".circle")
          .style("background-color", color);
        // 修改节点半径
        d3.select(this)
          .select("circle")
          .transition()
          .attr("r", self.size * 1.3);
      });

      self.view.selectAll(".node").on("mouseout", function() {
        // 修改节点半径
        d3.select(this)
          .select("circle")
          .transition()
          .attr("r", self.size);
        self.isTooltipVisiable = false;
        self.currentHoverInfo = null;
      });
    },
    // 当点击到节点时
    onClickNode() {
      let self = this;
      self.view.selectAll(".node").on("click", function() {
        // 清空选中状态
        self.view
          .selectAll(".node")
          .select("circle")
          .style("stroke", "#fff")
          .style("stroke-width", 2);
        // 设置选中状态
        d3.select(this)
          .select("circle")
          .style("stroke", "#e0e0e0")
          .style("stroke-width", 5);
        const nodeId = this.id; // 不可使用箭头函数, 否则this指向vue
        const currentClickInfo = self.nodeMap.get(nodeId);
        // 暴露事件, 并且传出当前点击节点信息
        self.$emit("topologyClick", currentClickInfo);
      });
    }
  }
};
</script>

<style scoped lang="scss">
.tooltip {
  width: 180px;
  height: 64px;
  position: fixed;
  z-index: 99;
  padding: 12px 8px;
  font-size: 12px;
  background: rgba(255, 255, 255, 0.85);
  box-shadow: 0px 0px 8px 0px rgba(20, 22, 26, 0.08);
  border-radius: 5px;
  border: 1px solid rgba(226, 232, 239, 1);
  .time {
    color: #333;
    font-weight: 500;
  }
  .content {
    color: #666;
    margin: 4px 0;
    .circle {
      display: inline-block;
      width: 6px;
      height: 6px;
      margin: 0 6px;
      border-radius: 3px;
      background: #188cff;
    }
    .text {
      display: inline-block;
      height: 24px;
      line-height: 24px;
    }
  }
}
</style>
<style lang="scss">
.topology-timeline {
  .node {
    cursor: pointer;
  }
  /* 坐标轴-start */
  .axis {
    path,
    line {
      fill: none;
      stroke: #dcdcdc;
      shape-rendering: crispEdges;
    }
    text {
      font-family: sans-serif;
      font-size: 12px;
      fill: #999999;
    }
    .axis-month {
      text {
        font-size: 14px;
        font-weight: 400;
        fill: #808080;
      }
      .tick {
        stroke-width: 2px;
      }
    }
    /* 坐标轴-end */
  }
}
</style>
```

## HTML版:
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    svg {
      border: 1px solid darkcyan;
    }

    /* 拓扑图--start */
    /* 节点状态颜色 */
    g.type-current>circle {
      fill: #FFAC27;
    }

    g.type-success>circle {
      fill: #9270CA;
    }

    g.type-fail>circle {
      fill: #67C23A;
    }

    g.type-done>circle {
      fill: #E8684A;
    }

    /* 拓扑图--end */

    /* 坐标轴-start */
    .axis path,
    .axis line {
      fill: none;
      stroke: #DCDCDC;
      shape-rendering: crispEdges;
    }

    .axis text {
      font-family: sans-serif;
      font-size: 12px;
      fill: #999999;
    }

    .axis .x2-axis text {
      font-size: 14px;
      font-weight: 400;
      fill: #333;
    }

    .axis .x2-axis .tick {
      stroke-width: 2px;
    }


    /* 坐标轴-end */
  </style>
</head>
<script src=" http://d3js.org/d3.v5.min.js "></script>
<script src="https://cdn.bootcss.com/dagre-d3/0.6.3/dagre-d3.js"></script>

<body>
</body>
<script>

  let nodeInfo = [{
    id: 0,
    label: "",
    status: 'success',
    date: 1575129600000
  }, {
    id: 1,
    label: "",
    status: 'fail',
    date: 1578376890000
  }, {
    id: 2,
    label: '',
    status: 'success',
    date: 1578376890000
  }, {
    id: 3,
    label: '',
    status: 'fail',
    date: 1578895290000
  }, {
    id: 4,
    label: '',
    status: 'current',
    date: 1578895290000
  }, {
    id: 5,
    label: '',
    status: 'done',
    date: 1579327290000
  }, {
    id: 6,
    label: '',
    status: 'done',
    date: 1579932090000
  }, {
    id: 7,
    label: '',
    status: 'done',
    date: 1581487290000
  }, {
    id: 8,
    label: '',
    status: 'success',
    date: 1583461994000
  }]
  let lineInfo = [
    { from: 0, to: 1 },
    { from: 0, to: 2 },
    { from: 0, to: 3 },
    { from: 2, to: 4 },
    { from: 2, to: 5 },
    { from: 3, to: 6 },
    { from: 6, to: 7 },
    { from: 6, to: 8 },
  ]

  let nodeMap = new Map() //节点信息map
  let nodeDomMap = new Map() //节点dom--map
  let timeArr = [] //存储时间

  const width = 1200
  const height = 400
  const padding = { top: 0, bottom: 40, left: 40, right: 40 }

  // 节点信息转化为map
  nodeInfo.forEach(item => {
    nodeMap.set(item.id, item);
    timeArr.push(item.date)
  })
  let max = new Date(d3.max(timeArr))
  let min = new Date(d3.min(timeArr))
  maxY = max.getFullYear()
  maxM = max.getMonth()
  minY = min.getFullYear()
  minM = min.getMonth()

  // 创建画布 svg
  let svg = d3.select("body").append("svg")
    .attr("id", "svg-canvas")
    .attr("preserveAspectRatio", "xMidYMid meet")
    .attr("viewBox", `0 0 ${width} ${height}`)

  // 初始化元素
  let background = svg.append("rect").attr("class", "bg")
  let view = svg.append("g").attr("class", "view")
  let grid = svg.append("g").attr("class", "grid")
  let axis = svg.append("g").attr("class", "axis")
  let separateLine = svg.append("line").attr("class", "separate-line")

  // 绘制箭头以供引用
  d3.select("#svg-canvas").append("defs").append("marker")
    .attr("id", "triangle").attr("viewBox", "0 0 10 10")
    .attr("refX", "17").attr("refY", "5")
    .attr("markerWidth", "6").attr("markerHeight", "6")
    .attr("orient", "auto").append("path")
    .attr("d", "M 0 0 L 10 5 L 0 10 z").style("fill", "#bbbbbb")

  // 添加背景板 rect
  background.attr("fill", "#FAFAFA")
    .attr("x", 0).attr("y", 0)
    .attr("width", width).attr("height", height - padding.bottom)
  const monthNum = d3.timeMonth.count(min, max) // 区间月份数量

  // 确定比例尺
  let xScale = d3.scaleTime()
    .domain([new Date(minY, minM, 1), new Date(maxY, ++maxM, 1)])
    .range([0, width - padding.left - padding.right])

  // 坐标轴文本格式化
  let formatDay = d3.axisBottom(xScale).tickFormat((d, i) => {
    const date = new Date(d)
    const day = date.getDate()
    return `${day === 1 ? "" : day}` // 如果是1号, 不显示刻度,直接由xAxis2显示年月
  })
  let formatMonth = d3.axisBottom(xScale).ticks(d3.timeMonth.every(1)).tickPadding(6).tickSizeInner(20).tickFormat((d, i) => {
    const date = new Date(d)
    const mon = date.getMonth() + 1
    const year = date.getFullYear()
    return `${year} - ${mon > 9 ? mon : "0" + mon}`
  })
  axis.attr('transform', `translate(${padding.left},${height - padding.bottom})`)
  let xAxisDay = axis.append("g")
    .attr("class", "x-axis").call(formatDay)
  let xAxisMonth = axis.append("g")
    .attr("class", "x2-axis").call(formatMonth)


  // 绘制x网格
  const lineGroup = grid.attr("transform", `translate(${padding.left},0)`)
    .selectAll("g")
    .data(xScale.ticks(monthNum))
    .enter().append("g")
  lineGroup.append("line")
    .attr("x1", d => { return xScale(new Date(d)) })
    .attr("x2", d => { return xScale(new Date(d)) })
    .attr("y1", padding.top)
    .attr("y2", height - padding.bottom)
    .attr("class", "grid-line")
    .style("stroke", "#DCDCDC")
    .style("stroke-dasharray", 6)

  // 添加坐标轴与拓扑图分隔线
  separateLine.style("stroke", "#DCDCDC")
    .style("stroke-width", 2)
    .attr("x1", 0)
    .attr("x2", width)
    .attr("y1", height - padding.bottom)
    .attr("y2", height - padding.bottom)

  // 绘制流程图 节点--箭头
  let g = new dagreD3.graphlib.Graph()
    .setGraph({})
    .setDefaultEdgeLabel(function () { return {}; });
  g.graph().rankdir = "LR"; // 控制水平显示
  g.graph().marginx = 0;
  g.graph().marginy = 50;

  nodeInfo && nodeInfo.map((item, i) => {
    g.setNode(item.id, {
      label: item.label,
      class: "type-" + item.status,
      style: "stroke-width: 2px; stroke: #fff",
      shape: "circle",
      id: item.id
    });

  })

  lineInfo && lineInfo.map((item, i) => {
    g.setEdge(item.from, item.to,
      {
        arrowheadStyle: "stroke:none; fill: none", //  箭头头部样式
        style: "stroke:none; fill: none" //线条样式
      })

  })

  let render = new dagreD3.render();
  render(view.attr("transform", `translate(${padding.left},0)`), g);

  // 重新定位节点x坐标
  const nodesArr = d3.select(".nodes").selectAll(".node")._groups[0]
  nodesArr.forEach((item) => {
    let dom = d3.select(item)._groups[0][0]
    let id = Number(dom.id)
    let date = nodeMap.get(id).date
    const x = xScale(new Date(date));
    const y = dom.transform.animVal[0].matrix.f
    d3.select(item).attr("transform", `translate(${x},${y})`)
    nodeDomMap.set(Number(item.id), item)
  })

  // 重新绘制箭头
  lineInfo && lineInfo.map((item, i) => {
    let fromDom = nodeDomMap.get(Number(item.from))
    let toDom = nodeDomMap.get(Number(item.to))
    const [x1, y1, x2, y2] = [
      fromDom.transform.animVal[0].matrix.e,
      fromDom.transform.animVal[0].matrix.f,
      toDom.transform.animVal[0].matrix.e,
      toDom.transform.animVal[0].matrix.f,
    ]
    d3.select(".edgePaths").append("g")
      .append("line")
      .attr("class", `to-${item.to}`) // 设置唯一的class方便修改路径
      .attr("stroke-width", "2")
      .attr("stroke", "#bbbbbb")
      .style("stroke-dasharray", 8)
      .attr("marker-end", "url(#triangle)")
      .attr("x1", x1).attr("y1", y1)
      .attr("x2", x2).attr("y2", y2)

  })

  // 设置zoom参数
  let zoom = d3.zoom()
    .scaleExtent([1, 10])
    .translateExtent([[0, 0], [width, height]]) //移动的范围
    .extent([[0, 0], [width, height]])//视窗 （左上方，右下方）

  svg.call(zoom.on("zoom", reRender.bind(this)));


  // 每次缩放重定位渲染拓扑图
  function reRender() {
    const t = d3.event.transform.rescaleX(xScale)  //获得缩放后的比例尺
    xAxisDay.call(formatDay.scale(t))   //重新设置x坐标轴的scale
    xAxisMonth.call(formatMonth.scale(t))   //重新设置x坐标轴的scale

    const view = d3.select(".output")
    const axis = d3.select(".axis-month")
    const grid = d3.selectAll(".grid-line")

    // 重新绘制节点
    nodesArr.forEach((item) => {
      let dom = d3.select(item)._groups[0][0]
      let id = Number(dom.id)
      let date = nodeMap.get(id).date
      const x = t(new Date(date));
      const y = dom.transform.animVal[0].matrix.f
      d3.select(item).attr("transform", `translate(${x},${y})`)
      nodeDomMap.set(Number(item.id), item)
    })

    // 重新绘制箭头
    lineInfo && lineInfo.map((item, i) => {
      let fromDom = nodeDomMap.get(Number(item.from))
      let toDom = nodeDomMap.get(Number(item.to))
      const [x1, y1, x2, y2] = [
        fromDom.transform.animVal[0].matrix.e,
        fromDom.transform.animVal[0].matrix.f,
        toDom.transform.animVal[0].matrix.e,
        toDom.transform.animVal[0].matrix.f,
      ]
      d3.select(`.to-${item.to}`)
        .attr("x1", x1).attr("y1", y1)
        .attr("x2", x2).attr("y2", y2)

    })

    //重新绘制x网格
    svg.selectAll(".grid-line")
      .attr("x1", d => { return t(new Date(d)) })
      .attr("x2", d => { return t(new Date(d)) })
  }



</script>

</html>
```

