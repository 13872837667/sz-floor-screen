<script setup>
import { ref, computed, onMounted } from 'vue'

const floorList = ref([
  { floor: '14F', name: '交换区', status: 'normal' },
  { floor: '11F', name: '无菌注射器车间（眼罩）', status: 'normal' },
  { floor: '10F', name: '工厂车间（棉柔巾）实验室', status: 'normal' },
  { floor: '9F', name: '工厂车间（胶原/医用胶水）', status: 'normal' },
  { floor: '8F', name: '工厂车间（手术包柔性生产）', status: 'normal' },
  { floor: '7F', name: '工厂车间（手术包自动化）', status: 'active' },
  { floor: '6F', name: '工厂仓库', status: 'normal' },
  { floor: '5F', name: '工厂仓库', status: 'normal' },
  { floor: '4F', name: '工厂车间（手术包组装）', status: 'normal' },
  { floor: '3F', name: '全棉时代 (2B&2C)', status: 'normal' },
  { floor: '2F', name: '集团医疗 (内贸&外贸)', status: 'normal' },
  { floor: '1F', name: '集团发货区', status: 'normal' },
])

// 任务列表
const tasks = ref([
  { id: 1, side: 'left', fromFloor: '7F', toFloor: '9F', type: 'solid', color: '#00ff00' },
  { id: 2, side: 'left', fromFloor: '7F', toFloor: '11F', type: 'dashed', color: '#00f7ff' },
  { id: 3, side: 'right', fromFloor: '7F', toFloor: '9F', type: 'solid', color: '#00ff00' },
  { id: 4, side: 'right', fromFloor: '7F', toFloor: '1F', type: 'dashed', color: '#0066ff' },
])

// SVG 视图框高度 (参考设计高度)
const viewBoxHeight = 816
const rowHeight = 60

// 获取楼层在 SVG 坐标系中的 Y 坐标
const getFloorY = (floorLabel) => {
  const index = floorList.value.findIndex(f => f.floor === floorLabel)
  if (index === -1) return 0
  const spacing = (viewBoxHeight - floorList.value.length * rowHeight) / (floorList.value.length - 1)
  return index * (rowHeight + spacing) + rowHeight / 2
}

// 电梯位置 (两部轿厢始终同步)
const carPos = ref('7F')

// 当前激活楼层
const activeFloor = ref('7F')

// 模拟动态更新
const startSimulation = () => {
  setInterval(() => {
    // 随机选取一个目标楼层
    const floors = floorList.value.map(f => f.floor)
    const randomFloor = floors[Math.floor(Math.random() * floors.length)]
    
    // 左右两侧实线任务同步更新
    const solidTasks = tasks.value.filter(t => t.type === 'solid')
    solidTasks.forEach(task => {
      task.fromFloor = carPos.value // 起点设为当前位置
      task.toFloor = randomFloor    // 终点设为目标
    })
    
    // 轿厢开始移动
    carPos.value = randomFloor
    
    // 1秒后（CSS transition完成时）才激活楼层
    setTimeout(() => { 
      activeFloor.value = randomFloor
    }, 1000)
    
  }, 5000)
}

onMounted(() => {
  startSimulation() // 开启模拟演示
})

const carTop = computed(() => (getFloorY(carPos.value) / viewBoxHeight) * 100 + '%')

const startX_Left = 108 // node at 87 + half of 28 is around 101, but node is at 87. let's use node center
const startX_Right = 258 // node at 238 + half of 28 is around 252, but node is at 238. 

const nodeX_Left = 93  // node at 87 + half of 12 is 93
const nodeX_Right = 244 // node at 238 + half of 12 is 244

// 生成路径数据
const getPathData = (task) => {
  const startY = getFloorY(task.fromFloor)
  const endY = getFloorY(task.toFloor)
  
  if (task.side === 'left') {
    // 右侧面板的左电梯：从轨道右边缘 (108) 出发 -> 偏移量 -> 目标层 -> 指向节点 (93)
    const startX = 108
    const horizontalOffset = task.type === 'solid' ? 140 : 160
    const targetX = nodeX_Left
    return `M ${startX} ${startY} L ${horizontalOffset} ${startY} L ${horizontalOffset} ${endY} L ${targetX} ${endY}`
  } else {
    // 右侧面板的右电梯：从轨道左边缘 (230) 出发 -> 偏移量 -> 目标层 -> 指向节点 (244)
    const startX = 230
    const horizontalOffset = task.type === 'solid' ? 190 : 170
    const targetX = nodeX_Right
    return `M ${startX} ${startY} L ${horizontalOffset} ${startY} L ${horizontalOffset} ${endY} L ${targetX} ${endY}`
  }
}

// 生成箭头坐标 (Polygon points)
const getArrowPoints = (task) => {
  const endY = getFloorY(task.toFloor)
  if (task.side === 'left') {
    // 向左的箭头，顶点在 (nodeX_Left, endY)
    return `${nodeX_Left},${endY} ${nodeX_Left + 7},${endY - 3} ${nodeX_Left + 7},${endY + 3}`
  } else {
    // 向右的箭头，顶点在 (nodeX_Right, endY)
    return `${nodeX_Right},${endY} ${nodeX_Right - 7},${endY - 3} ${nodeX_Right - 7},${endY + 3}`
  }
}
</script>

<template>
  <div class="side-panel right-panel">
    <div class="panel-title">
      <span class="title-text">洁净电梯任务展示</span>
      <span class="title-arrow">»</span>
    </div>
    
    <div class="status-legend">
      <div class="legend-group">
        <span class="legend-item"><i class="dot run"></i>运行</span>
        <span class="legend-item"><i class="dot fail"></i>故障</span>
      </div>
      <div class="legend-group">
        <span class="legend-item"><i class="dot once"></i>运行一次</span>
        <span class="legend-item"><i class="dot twice"></i>运行两次</span>
        <span class="legend-item"><i class="dot thrice"></i>运行三次</span>
      </div>
    </div>
    
    <div class="floor-list-container">
      <div class="elevator-track-container">
        <div class="track-wrapper">
          <!-- 轨道背景 -->
          <div class="track-bar left"></div>
          <div class="track-bar right"></div>
          
          <!-- 楼层节点与标签 -->
          <div class="track-floors">
            <div v-for="(item, index) in floorList" :key="index" class="track-row">
              <div class="floor-tag-box" :class="{ active: activeFloor === item.floor }">
                <span class="tag-text">{{ item.floor }}</span>
              </div>
              <div class="node left-node"></div>
              <div class="node right-node"></div>
            </div>
          </div>
          
          <!-- 任务连线 SVG -->
          <svg class="task-svg" viewBox="0 0 260 816" preserveAspectRatio="none">
            <template v-for="task in tasks" :key="task.id">
              <path 
                :d="getPathData(task)" 
                fill="none" 
                :stroke="task.color" 
                stroke-width="2" 
                :class="['task-path', task.type === 'solid' ? 'solid-flow' : 'dashed-flow']"
              />
              <polygon :points="getArrowPoints(task)" :fill="task.color" />
            </template>
          </svg>

          <!-- 电梯轿厢 -->
          <div class="car left-car" :style="{ top: carTop }">
            <div class="car-dot"></div>
          </div>
          <div class="car right-car" :style="{ top: carTop }">
            <div class="car-dot"></div>
          </div>
        </div>
      </div>
      
      <div class="floor-list">
        <div v-for="(item, index) in floorList" :key="index" :class="['floor-item', { active: activeFloor === item.floor }]">
          <div class="floor-content">
            <div class="floor-label">{{ item.floor }}</div>
            <div class="floor-name">{{ item.name }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.side-panel {
  width: 100%;
  height: 100%;
  background: url('http://s2b2c-prod.oss-cn-shenzhen.aliyuncs.com/0000/file/wmsImg/left_bg.png') no-repeat;
  background-size: 100% 100%;
  display: flex;
  flex-direction: column;
}

.panel-title {
  height: 40px;
  background: url('http://s2b2c-prod.oss-cn-shenzhen.aliyuncs.com/0000/file/wmsImg/group.png') no-repeat;
  background-size: 100% 100%;
  display: flex;
  align-items: center;
  padding-left: 30px;
  margin-bottom: 10px;
  
  .title-text {
    color: #00f7ff;
    font-size: 18px;
    font-weight: bold;
    letter-spacing: 1px;
  }
  
  .title-arrow {
    margin-left: 10px;
    color: #00f7ff;
  }
}

.status-legend {
  display: flex;
  flex-wrap: wrap;
  gap: 80px;
  padding: 10px 0;
  font-size: 14px;
  color: #fff;
  opacity: 0.8;
  
  .legend-group {
    display: flex;
    gap: 20px;
  }
  
  .legend-item {
    display: flex;
    align-items: center;
    gap: 8px;
    
    .dot {
      width: 12px;
      height: 12px;
      border-radius: 2px;
      
      &.run { background: #00ff00; }
      &.fail { background: #ff0000; }
      &.once { background: #00ff00; }
      &.twice { background: #00f7ff; }
      &.thrice { background: #0066ff; }
    }
  }
}

.floor-list-container {
  flex: 1;
  display: flex;
  gap: 20px;
  overflow: hidden;
  padding: 10px 0 60px 0;
}

.floor-list {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  
  .floor-item {
    height: calc(60 / 816 * 100%);
    background: linear-gradient(270deg, rgba(0, 247, 255, 0.15) 0%, rgba(0, 247, 255, 0.05) 100%);
    border: 1px solid;
    border-image: linear-gradient(270deg, rgba(0, 247, 255, 0.5), rgba(0, 247, 255, 0.1)) 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 0 15px;
    border-radius: 4px;
    transition: all 0.3s;
    
    &.active {
      background: linear-gradient(270deg, rgba(0, 255, 0, 0.3) 0%, rgba(0, 255, 0, 0.1) 100%);
      border-image: linear-gradient(270deg, rgba(0, 255, 0, 0.8), rgba(0, 255, 0, 0.2)) 1;
      box-shadow: inset 0 0 15px rgba(0, 255, 0, 0.2);
      
      .floor-label { color: #fff; text-shadow: 0 0 5px #fff; }
    }
    
    .floor-content {
      display: flex;
      flex-direction: column;
      gap: 2px;
      align-items: flex-end;
    }
    
    .floor-label {
      font-size: 16px;
      font-weight: 800;
      font-style: italic;
      color: #00f7ff;
    }
    
    .floor-name {
      font-size: 12px;
      color: #fff;
      opacity: 0.7;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  }
}

.elevator-track-container {
  width: 260px;
  height: 100%;
  position: relative;
  background: rgba(0, 247, 255, 0.02);
  
  .track-wrapper {
    position: relative;
    width: 100%;
    height: 100%;
    
    .track-bar {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 28px;
      background: linear-gradient(90deg, rgba(0, 247, 255, 0.1), rgba(0, 247, 255, 0.05), rgba(0, 247, 255, 0.1));
      border: 1px solid rgba(0, 247, 255, 0.15);
      
      &.left { left: 80px; }
      &.right { left: 230px; }
    }
    
    .track-floors {
      display: flex;
      flex-direction: column;
      height: 100%;
      justify-content: space-between;
      position: relative;
      z-index: 1;
      
      .track-row {
        height: calc(60 / 816 * 100%);
        display: flex;
        align-items: center;
        position: relative;
        
        .node {
          width: 12px;
          height: 12px;
          border-radius: 50%;
          border: 1px solid #00f7ff;
          background: rgba(0, 0, 0, 0.8);
          position: absolute;
          display: flex;
          align-items: center;
          justify-content: center;
          top: 50%;
          transform: translateY(-50%);
          
          &.left-node { left: 87px; }
          &.right-node { left: 238px; }
          
          &::after {
            content: '';
            width: 4px;
            height: 4px;
            background: #00f7ff;
            border-radius: 50%;
            box-shadow: 0 0 5px #00f7ff;
          }
        }
        
        .floor-tag-box {
          position: absolute;
          left: 10px;
          width: 50px;
          height: 100%;
          background: linear-gradient(270deg, rgba(0, 247, 255, 0.15) 0%, rgba(0, 247, 255, 0.05) 100%);
          border: 1px solid;
          border-image: linear-gradient(270deg, rgba(0, 247, 255, 0.5), rgba(0, 247, 255, 0.1)) 1;
          display: flex;
          align-items: center;
          justify-content: center;
          border-radius: 4px;
          
          &.active {
            background: linear-gradient(270deg, rgba(0, 255, 0, 0.3) 0%, rgba(0, 255, 0, 0.1) 100%);
            border-image: linear-gradient(270deg, rgba(0, 255, 0, 0.8), rgba(0, 255, 0, 0.2)) 1;
            .tag-text { color: #fff; text-shadow: 0 0 5px #fff; }
          }
          
          .tag-text {
            font-size: 14px;
            color: #00f7ff;
            font-weight: 800;
            font-style: italic;
          }
        }
      }
    }
    
    .task-svg {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      z-index: 2;
    }
    
    .car {
      position: absolute;
      width: 28px;
      height: 34px;
      background: rgba(0, 255, 0, 0.2);
      border: 1px solid #00ff00;
      box-shadow: 0 0 10px rgba(0, 255, 0, 0.3);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 3;
      transform: translateY(-50%);
      border-radius: 4px;
      transition: top 1s ease-in-out;
      
      &.left-car { left: 80px; }
      &.right-car { left: 230px; }
      
      .car-dot {
        width: 10px;
        height: 10px;
        background: #00ff00;
        border-radius: 50%;
        box-shadow: 0 0 10px #00ff00;
        animation: pulse 1.5s infinite;
      }
    }
  }
}

.task-path {
  transition: d 1s ease-in-out;
}

.solid-flow {
  stroke-dasharray: 6, 4;
  animation: flow 2s linear infinite;
}

.dashed-flow {
  stroke-dasharray: 4, 4;
  animation: flow 4s linear infinite;
}

@keyframes flow {
  from { stroke-dashoffset: 200; }
  to { stroke-dashoffset: 0; }
}

@keyframes pulse {
  0% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.3); opacity: 0.6; }
  100% { transform: scale(1); opacity: 1; }
}
</style>
