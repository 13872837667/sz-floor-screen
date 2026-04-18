<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import DashboardHeader from './components/DashboardHeader.vue'
import LeftPanel from './components/LeftPanel.vue'
import RightPanel from './components/RightPanel.vue'
import BottomChart from './components/BottomChart.vue'
import ThreeScene from './components/ThreeScene.vue'

const screenRef = ref(null)

const handleResize = () => {
  if (!screenRef.value) return
  const width = 1920
  const height = 1080
  const ww = window.innerWidth
  const wh = window.innerHeight
  const scaleX = ww / width
  const scaleY = wh / height
  screenRef.value.style.transform = `translate(-50%, -50%) scale(${scaleX}, ${scaleY})`
}

onMounted(() => {
  handleResize()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<template>
  <div class="screen-wrapper" ref="screenRef">
    <DashboardHeader />
    <div class="screen-bg"></div>
    
    
    
    
    <div class="main-content">
      <ThreeScene />
      <LeftPanel class="left-p" />
      <RightPanel class="right-p" />
      <BottomChart class="bottom-chart" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.main-content {
  flex: 1;
  display: flex;
  padding: 20px;
  box-sizing: border-box;
  position: relative;
  z-index: 1;
  overflow: hidden;
  .left-p{
    width: 25%;
    height: 100%;
    position: absolute;
    top: 10px;
    left: 30px;
  }
  .right-p{
    width: 25%;
    height: 100%;
    position: absolute;
    top: 10px;
    right: 30px;
  }
  .bottom-chart{
    position: absolute;
    width: calc(50% - 80px);
    height: 250px;
    bottom: 51px;
    left: 520px;
  }
}

.center-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  position: relative;
  margin: 0 20px;
}
</style>
