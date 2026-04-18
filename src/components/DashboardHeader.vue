<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const currentTime = ref('')
const currentDate = ref('')
const currentWeek = ref('')

const weeks = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六']

const updateTime = () => {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('zh-CN', { hour12: false })
  currentDate.value = now.toLocaleDateString('zh-CN').replace(/\//g, '-')
  currentWeek.value = weeks[now.getDay()]
}

let timer
onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  clearInterval(timer)
})
</script>

<template>
  <div class="dashboard-header">
    <div class="header-logo">
      <img src="https://winner--winplus.oss-cn-shenzhen.aliyuncs.com/static/kanban/%E8%B5%84%E6%BA%90%202%402x.png" alt="logo" class="logo-img">
    </div>
    
    <div class="header-title">深圳稳健产业大厦</div>
    
    <div class="header-info">
      <div class="weather">
        <span class="weather-temp">32℃</span>
        <span class="weather-desc">PM 2.0</span>
      </div>
      <div class="divider"></div>
      <div class="time-info">
        <div class="time">{{ currentTime }}</div>
        <div class="date-week">
          <span>{{ currentDate }}</span>
          <span>{{ currentWeek }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.dashboard-header {
  height: 100px;
  width: 100%;
  background: url('http://s2b2c-prod.oss-cn-shenzhen.aliyuncs.com/0000/file/wmsImg/top_img.png') no-repeat center bottom;
  background-size: 100% 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 40px;
  position: relative;
  z-index: 10;
}

.header-logo {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 50px;
  .logo-img {
    height: 30px;
  }
  
  .logo-text {
    font-size: 20px;
    font-weight: bold;
    color: #fff;
    letter-spacing: 2px;
  }
}

.header-title {
  position: absolute;
  left: 50%;
  top: 45%;
  transform: translate(-50%, -50%);
  font-size: 36px;
  font-weight: 800;
  font-style: italic;
  letter-spacing: 6px;
  
  /* 渐变色文字 */
  background: linear-gradient(180deg, #FFFFFF 20%, #A3F9FF 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  
  /* 调淡发光效果 */
  filter: drop-shadow(0 0 4px rgba(0, 247, 255, 0.6)) drop-shadow(0 0 8px rgba(0, 247, 255, 0.3));
  
  /* 增强边缘清晰度 */
  -webkit-text-stroke: 0.2px rgba(255, 255, 255, 0.3);
}

.header-info {
  display: flex;
  align-items: center;
  gap: 30px;
  margin-top: 50px;
  
  .weather {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    color: #fff;
    
    .weather-temp {
      font-size: 18px;
    }
    
    .weather-desc {
      font-size: 12px;
      opacity: 0.8;
    }
  }
  
  .divider {
    width: 1px;
    height: 28px;
    background: rgba(255, 255, 255, 0.4);
    margin: 0 5px;
  }
  
  .time-info {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    color: #fff;
    
    .time {
      font-size: 24px;
      font-weight: bold;
      font-family: 'Courier New', Courier, monospace;
    }
    
    .date-week {
      font-size: 12px;
      display: flex;
      gap: 10px;
      opacity: 0.8;
    }
  }
}
</style>
