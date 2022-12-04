<!-- d3js 气泡图 -->
<template>
  <div style="display: flex; justify-content: center">
  <div id="bubble" style="width: 500px;height:500px;"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts/core';
import {
  DatasetComponent,
  TooltipComponent,
  VisualMapComponent
} from 'echarts/components';
import { CustomChart } from 'echarts/charts';
import { CanvasRenderer } from 'echarts/renderers';
echarts.use([
  DatasetComponent,
  TooltipComponent,
  VisualMapComponent,
  CustomChart,
  CanvasRenderer
]);
import * as d3 from 'd3';
export default {
  name:"bubbleChart",
  data() {
    return {
      option: {},
      colorList: ['#5470c6', '#91cc75', '#fac858', '#ee6666', '#73c0de', '#3ba272', '#fc8452', '#9a60b4', '#ea7ccc']
    };
  },

  mounted() {
    let that = this
    let seriesData = [
        // 父亲节点
      {
        depth: 0,
        id: 'classroom',
        index: 0,
        vacancy: 0,
        name: '',
      },
      {
        depth: 1,
        id: 'classroom.东上院101',
        index: 1,
        vacancy: 100,
        name: '东上院101',
        url:"https://www.runoob.com/jsref/jsref-tutorial.html"
      },
      {
        depth: 1,
        id: 'classroom.东下院101',
        name: '东下院101',
        index: 2,
        vacancy: 92,
        url:"https://www.runoob.com/jsref/jsref-tutorial.html"
      },
      {
        depth: 1,
        name: '教室404',
        id: 'classroom.dataZoom-inside',
        index: 3,
        vacancy: 30
      },
      {
        depth: 1,
        id: 'classroom.dataZoom1',
        index: 4,
        vacancy: 40
      },
      {
        depth: 1,
        id: 'classroom.dataZoom2',
        index: 5,
        vacancy: 50
      },
      {
        depth: 1,
        id: 'classroom.dataZoom3',
        index: 5,
        vacancy: 60
      },
      {
        depth: 1,
        id: 'classroom.dataZoom4',
        index: 5,
        vacancy: 70
      },
      {
        depth: 1,
        id: 'classroom.dataZoom5',
        index: 5,
        vacancy: 80
      }
    ];
    let displayRoot = stratify1();
    function stratify1() {
      return d3
          .stratify()
          .parentId(function (d) {
            return d.id.substring(0, d.id.lastIndexOf('.'));
          })(seriesData)
          .sum(function (d) {
            return d.vacancy || 0;
          })
          .sort(function (a, b) {
            return b.vacancy - a.vacancy;
          });
    }
    function overallLayout(params, api) {
      let context = params.context;
      d3
          .pack()
          .size([api.getWidth() - 2, api.getHeight() - 2])
          .padding(0)(displayRoot);
      context.nodes = {}; // 把节点加入字典

      displayRoot.descendants().forEach(function (node) {

        context.nodes[node.id] = node;
      });
    }
    function renderItem(params, api) {
      // context: // {Object} 一个可供开发者暂存东西的对象。生命周期只为：当前次的渲染。 dataIndex: // {number} 数据项的 index。
      let context = params.context;
      let idx = params.dataIndex;
      // console.log(params);
      
      // Only do that layout once in each time `setOption` called.
      // 每次调用“setOption”时，只能进行一次布局。
      if (!context.layout) {
        context.layout = true;
        overallLayout(params, api);
      }

      let nodePath = api.value('id'); // 例如 classroom.dataZoom
      let nodeName = nodePath // 这里用了正则表达式来从节点路径得到名字，其实可以自定义
          .slice(nodePath.lastIndexOf('.') + 1)
          .split(/(?=[A-Z][^A-Z])/g)
          .join('')
      let node = context.nodes[nodePath];
      if (node.id === 'classroom') {
        node.r = 0 // 大圆不显示
      }
      if (!node) {
        // Render nothing.
        return;
      }
      if(node.data.name){
        nodeName = node.data.name;
        // nodeName = node.name;
      }

      let z2 = api.value('depth') * 2;
      return {
        type: 'circle',
        shape: {
          cx: node.x,
          cy: node.y,
          r: node.r
        },
        // transition: ['shape'],
        z2: z2,
        textContent: {
          type: 'text',
          style: {
            // transition: isLeaf ? 'fontSize' : null,
            text: nodeName, // 圆圈内部的文字
            fill: '#fff',
            fontFamily: 'Arial',
            width: node.r * 1.3,
            overflow: 'truncate',
            fontSize: node.r / 3
          },
          emphasis: {
            style: {
              overflow: null,
              fontSize: Math.max(node.r / 3, 12)
            }
          }
        },
        textConfig: {
          position: 'inside'
        },
        style: {
          fill: that.colorList[idx % that.colorList.length]
        },
        emphasis: {
          style: {
            fontFamily: 'Arial',
            fontSize: 12,
            shadowBlur: 20,
            shadowOffsetX: 3,
            shadowOffsetY: 5,
            shadowColor: 'rgba(0,0,0,0.3)'
          }
        }
      };
    }
    this.option = {
      dataset: {
        source: seriesData
      },
      tooltip: {
        formatter(params) {
            return (
                params.data.name +
                "的剩余位置：" +
                    params.data.vacancy // 这里注意要先进入data，否则会找不到！https://blog.csdn.net/metooyoume/article/details/108726385
            );
        },

      }, // 决定鼠标悬浮是否显示信息
      hoverLayerThreshold: Infinity,
      series: [{
        type: 'custom',
        colorBy: 'data',
        renderItem: renderItem, // 自定义custom需要这个函数
        progressive: 0,
        coordinateSystem: 'none',
        encode: {
          // tooltip: ['name','vacancy'],
          // itemName: 'id'
        }
      }]
    }
    this.initEcharts()
  },

  methods: {
    initEcharts() {
      let myChart = echarts.init(document.getElementById('bubble'))
      myChart.setOption(this.option)
      myChart.on('click',function (params){
        window.open(params.data.url);
      })
    }
  }
}

</script>
