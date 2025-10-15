<template>
  <div class="window-overlay" v-if="isVisible" @click.self="handleClose">
    <div class="window-container" :class="{ 'maximized': isMaximized }">
      <!-- 窗口标题栏 -->
      <div class="window-titlebar">
        <div class="window-title">设置</div>
        <div class="window-controls">
          <button class="window-control-button" @click="toggleMaximize">
            {{ isMaximized ? '▣' : '▢' }}
          </button>
          <button class="window-control-button close-button" @click="handleClose">
            ✕
          </button>
        </div>
      </div>
      
      <!-- 窗口内容 -->
      <div class="window-content">
        <h2>滤镜设置</h2>
        <div class="settings-section">
          <div class="setting-item">
            <label>畸变:</label>
            <div class="toggle-switch">
              <input 
                type="checkbox" 
                id="distortion" 
                :checked="distortionEnabled"
                @change="setDistortionEnabled($event.target.checked)"
              >
              <label for="distortion" class="slider"></label>
            </div>
            <span>{{ distortionEnabled ? '开' : '关' }}</span>
          </div>
          
          <div class="setting-item">
            <label>扫描线:</label>
            <div class="toggle-switch">
              <input 
                type="checkbox" 
                id="scanlines" 
                :checked="scanlinesEnabled"
                @change="setScanlinesEnabled($event.target.checked)"
              >
              <label for="scanlines" class="slider"></label>
            </div>
            <span>{{ scanlinesEnabled ? '开' : '关' }}</span>
          </div>
          
          <div class="setting-item">
            <label>CRT滤镜强度:</label>
            <input 
              type="range" 
              min="0" 
              max="100" 
              :value="crtIntensity"
              @input="setCrtIntensity($event.target.value)"
            >
            <span>{{ crtIntensity }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'setting-Page',
  inject: ['scanlinesEnabled', 'crtIntensity', 'distortionEnabled', 'setScanlinesEnabled', 'setCrtIntensity', 'setDistortionEnabled'],
  data() {
    return {
      isVisible: true,
      isMaximized: false
    }
  },
  methods: {
    handleClose() {
      this.$emit('close')
    },
    toggleMaximize() {
      this.isMaximized = !this.isMaximized
    }
  }
}
</script>

<style scoped>
.window-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.window-container {
  width: 50%;
  height: 50%;
  background-color: #000;
  border: 1px solid white;
  padding: 2px;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease;
  max-width: 90vw;
  max-height: 90vh;
  position: relative;
}

.window-container.maximized {
  width: calc(80vw * 0.95); /* DefaultDisplay边框宽度(80vw)的95% */
  height: calc(80vh * 0.95); /* DefaultDisplay边框高度(80vh)的95% */
  max-width: calc(80vw * 0.95);
  max-height: calc(80vh * 0.95);
}

.window-titlebar {
  background-color: #000;
  color: white;
  padding: 8px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid white;
}

.window-title {
  font-size: 14px;
  font-weight: 500;
}

.window-controls {
  display: flex;
  gap: 8px;
}

.window-control-button {
  width: 32px;
  height: 32px;
  border: none;
  background-color: transparent;
  color: white;
  font-size: 16px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s ease;
}

.window-control-button:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

.close-button:hover {
  background-color: #f45050;
}

.window-content {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  background-color: #000;
}

.window-content h2 {
  margin-top: 0;
  color: white;
}

.window-content h3 {
  color: white;
  margin-top: 20px;
  margin-bottom: 10px;
}

.settings-section {
  margin-bottom: 20px;
}

.setting-item {
  display: flex;
  align-items: center;
  margin-bottom: 15px;
  gap: 10px;
}

.setting-item label {
  width: 100px;
  color: white;
}

.setting-item input[type="range"] {
  flex: 1;
  padding: 0;
  background-color: #000;
  color: white;
  border: 2px solid white;
  height: 20px;
  appearance: none;
  cursor: pointer;
}

/* 自定义拖动条样式 */
.setting-item input[type="range"]::-webkit-slider-track {
  background: #000;
  height: 100%;
}

.setting-item input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 16px;
  height: 24px;
  background: #fff;
  border: 2px solid #000;
  cursor: pointer;
  box-shadow: 0 0 0 2px white;
}

.setting-item input[type="range"]::-moz-range-track {
  background: #000;
  height: 100%;
  border: 2px solid white;
}

.setting-item input[type="range"]::-moz-range-thumb {
  width: 16px;
  height: 24px;
  background: #fff;
  border: 2px solid #000;
  cursor: pointer;
}

.setting-item span {
  width: 40px;
  text-align: right;
  color: white;
}

/* 用户要求的开关样式：长25px，高20px，透明按钮，无文字 */
.toggle-switch {
  position: relative;
  display: inline-block;
  width: 25px;
  height: 20px;
}
.toggle-switch label {
  width: 50px;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #000;
  border: 2px solid white;
  transition: .3s;
  border-radius: 0;
}

.slider:before {
  position: absolute;
  content: "";
  height: 12px;
  width: 5px;
  left: 2px;
  bottom: 2px;
  background-color: transparent;
  /* 设置按钮边框 */
  /* border: 1px solid white; */
  transition: .3s;
  border-radius: 0;
}

input:checked + .slider {
  background-color: white;
}

input:focus + .slider {
  box-shadow: 0 0 2px white;
}

input:checked + .slider:before {
  transform: translateX(14px);
  background-color: transparent;
}

/* 确保按钮与边框对齐的样式 */
.toggle-switch:after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
}
</style>