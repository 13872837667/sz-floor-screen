<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref(null)
let myChart = null

const initChart = () => {
  if (!chartRef.value) return
  myChart = echarts.init(chartRef.value)
  
  const data = [110, 180, 150, 220, 190, 210, 195]
  const xData = ['03.01', '03.02', '03.03', '03.04', '03.05', '03.06', '03.07']
  
  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' },
      backgroundColor: 'rgba(5, 39, 58, 0.9)',
      borderColor: 'rgba(0, 247, 255, 0.5)',
      borderWidth: 1,
      padding: [10, 15],
      textStyle: { color: '#fff', fontSize: 14 },
      formatter: function(params) {
        let res = `<div style="margin-bottom: 5px;">${params[0].name}</div>`;
        params.forEach(item => {
          if (item.seriesName === '任务数') {
            res += `<div style="display: flex; align-items: center;">
                      <span style="display:inline-block; margin-right:8px; width:12px; height:12px; background:#00f7ff; box-shadow: 0 0 5px #00f7ff; transform: rotate(45deg);"></span>
                      <span style="margin-right: 15px;">${item.seriesName}</span>
                      <span style="color:#00f7ff; font-weight:bold;">${item.value} 次</span>
                    </div>`;
          }
        });
        return res;
      }
    },
    grid: {
      left: '3%',
      right: '3%',
      bottom: '0',
      top: '20%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: xData,
      axisLine: { lineStyle: { color: 'rgba(0, 247, 255, 0.3)' } },
      axisLabel: { color: 'rgba(255, 255, 255, 0.8)', fontSize: 12 },
      axisTick: { show: false }
    },
    yAxis: {
      type: 'value',
      axisLine: { show: false },
      axisLabel: { color: 'rgba(255, 255, 255, 0.8)', fontSize: 12 },
      splitLine: { 
        lineStyle: { 
          color: 'rgba(255, 255, 255, 0.1)', 
          type: 'dashed' 
        } 
      }
    },
    series: [
      {
        name: '任务数',
        type: 'bar',
        barWidth: 20,
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(0, 247, 255, 0.6)' },
            { offset: 1, color: 'rgba(0, 247, 255, 0.1)' }
          ])
        },
        data: data
      },
      // 中间明亮的棱线 - 使用 pictorialBar 以确保与主柱体完美对齐
      {
        name: '棱线',
        type: 'pictorialBar',
        symbol: 'rect',
        symbolSize: [1.5, '100%'],
        z: 11,
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(163, 249, 255, 0.5)' },
            { offset: 1, color: 'rgba(163, 249, 255, 0.2)' }
          ])
        },
        data: data,
        tooltip: { show: false }
      },
      // 底部菱形
      {
        name: '底部菱形',
        type: 'pictorialBar',
        symbol: 'diamond',
        symbolSize: [20, 10],
        symbolOffset: [0, 5],
        z: 10,
        itemStyle: {
          color: 'rgba(0, 247, 255, 0.4)',
          shadowBlur: 3,
          shadowColor: '#00f7ff'
        },
        symbolPosition: 'start',
        data: data
      },
      // 顶部菱形
      {
        name: '顶部菱形',
        type: 'pictorialBar',
        symbol: 'diamond',
        symbolSize: [20, 10],
        symbolOffset: [0, -5],
        z: 12,
        itemStyle: {
          color: 'rgba(163, 249, 255, 0.6)',
          shadowBlur: 5,
          shadowColor: 'rgba(163, 249, 255, 0.5)'
        },
        symbolPosition: 'end',
        data: data
      }
    ]
  }
  
  myChart.setOption(option)
}

const handleResize = () => {
  myChart?.resize()
}

onMounted(() => {
  initChart()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<template>
  <div class="bottom-chart-container">
    <div class="chart-header">
      <div class="title-box">
        <span class="title">电梯任务数</span>
        <span class="title-arrow">»</span>
      </div>
      <span class="unit">单位：次</span>
    </div>
    <div class="chart-body" ref="chartRef"></div>
  </div>
</template>

<style lang="scss" scoped>
.bottom-chart-container {
  height: 100%;
  width: 100%;
  background: rgba(0, 247, 255, 0.05);
  border: 1px solid rgba(0, 247, 255, 0.1);
  position: relative;
  z-index: 5;
  margin-top: auto;
}

.chart-header {
  height: 40px;
  background: url('http://s2b2c-prod.oss-cn-shenzhen.aliyuncs.com/0000/file/wmsImg/group.png') no-repeat;
  background-size: 100% 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px 0 30px;
  margin-bottom: 10px;
  
  .title-box {
    display: flex;
    align-items: center;
  }
  
  .title {
    color: #00f7ff;
    font-size: 18px;
    font-weight: bold;
    letter-spacing: 1px;
  }
  
  .title-arrow {
    margin-left: 10px;
    color: #00f7ff;
  }
  
  .unit {
    color: #fff;
    font-size: 12px;
    opacity: 0.6;
  }
}

.chart-body {
  height: 180px;
  width: 100%;
}
</style>
