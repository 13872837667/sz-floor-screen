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
    <div class="screen-bg"></div>
    
    <DashboardHeader />
    
    <div class="main-content">
      <LeftPanel />
      
      <div class="center-content">
        <ThreeScene />
        <BottomChart />
      </div>
      
      <RightPanel />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.main-content {
  flex: 1;
  display: flex;
  padding: 20px;
  position: relative;
  z-index: 1;
  overflow: hidden;
}

.center-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  position: relative;
  margin: 0 20px;
}
</style>
