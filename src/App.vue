<script setup>
import { ref, provide, onMounted } from 'vue'
import CrtShader from './components/CrtShader.vue'
import DesktopPage from './components/desktop-Page.vue'
import RollingBar from './components/RollingBar.vue' // 引入自定义滚动条样式组件
import DefaultDisplay from './components/DefaultDisplay.vue' // 引入DefaultDisplay组件
import StartupAnimation from './components/StartupAnimation.vue' // 引入StartupAnimation组件
import TaskBar from './components/Task-Bar.vue' // 引入Task-Bar组件

// CRT效果设置状态
const scanlinesEnabled = ref(true) // 扫描线开关，默认开启
const crtIntensity = ref(100) // CRT滤镜强度，默认100
const distortionEnabled = ref(true) // 畸变效果开关，默认开启

// 设备检测状态
const isMobile = ref(false)

// 检测是否为移动设备
const checkDevice = () => {
  const userAgent = navigator.userAgent || navigator.vendor || window.opera;
  
  // 检查常见的移动设备标识
  const mobileRegex = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i;
  isMobile.value = mobileRegex.test(userAgent);
}

// 在组件挂载时检测设备
onMounted(() => {
  checkDevice();
  
  // 添加窗口大小变化监听器，以处理设备旋转等情况
  window.addEventListener('resize', checkDevice);
})

// 提供设置状态和更新函数给子组件
provide('scanlinesEnabled', scanlinesEnabled)
provide('crtIntensity', crtIntensity)
provide('distortionEnabled', distortionEnabled)
provide('setScanlinesEnabled', (value) => { scanlinesEnabled.value = value })
provide('setCrtIntensity', (value) => { crtIntensity.value = value })
provide('setDistortionEnabled', (value) => { distortionEnabled.value = value })
</script>

<template>
  <div class="app-container">
    <!-- 移动端提示 -->
    <div v-if="isMobile" class="mobile-warning">
      <div class="mobile-warning-border">
        <div class="mobile-warning-content">
          please use computer to access
        </div>
      </div>
    </div>
    
    <!-- 桌面端内容 -->
    <div v-else>
      <!-- 应用完整的CRT效果 -->
      <RollingBar /> <!-- 应用自定义滚动条样式 -->
      <!-- 所有非CrtShader元素放入DefaultDisplay边框内 -->
      <DefaultDisplay :distortionEnabled="distortionEnabled">
        <TaskBar/>
        <StartupAnimation class = startup />
        <DesktopPage/>
      </DefaultDisplay>
    </div>
    
    <!-- CRT滤镜保持最高优先级，对所有设备生效 -->
    <!-- refreshRate参数用于控制刷新率，默认为2Hz -->
    <!-- 可通过修改refreshRate参数来调整刷新率，例如：refreshRate="4" 将刷新率设置为4Hz -->
    <!-- 添加扫描线开关和CRT滤镜强度控制 -->
    <CrtShader 
      :refreshRate="2" 
      :scanlines="scanlinesEnabled"
      :crtIntensity="crtIntensity"
    />
  </div>
</template>

<style scoped>
.app-container {
  width: 100vw;
  height: 100vh;
  background: #000;
  position: relative;
  overflow: auto;
  .startup-animation {
    position: fixed;
    margin-top: -10%;
    margin-left: -15%;
  }
}

/* 移动端警告样式 */
.mobile-warning {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 999; /* 低于CRT效果但高于其他内容 */
}

.mobile-warning-border {
  width: 80%;
  height: 80%;
  border: 2px solid #d0b3b3;
  border-radius: 2%;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

.mobile-warning-content {
  font-size: 10%;
  color: white;
  text-align: center;
}
</style>
