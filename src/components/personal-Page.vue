<template>
  <div class="window-overlay" v-if="isVisible" @click.self="handleClose">
    <div class="window-container" :class="{ 'maximized': isMaximized }">
      <!-- 窗口标题栏 -->
      <div class="window-titlebar">
        <div class="window-title">个人页面</div>
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
        <h2>个人信息</h2>
        <p>这是个人信息页面的内容区域。</p>
        <p>您可以在此查看和编辑个人资料。</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'personal-Page',
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

.window-content p {
  color: white;
  line-height: 1.6;
}
</style>