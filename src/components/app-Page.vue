<template>
  <div class="window-overlay" v-if="isVisible" @click.self="handleClose">
    <div class="window-container" :class="{ 'maximized': isMaximized }">
      <!-- 窗口标题栏 -->
      <div class="window-titlebar">
        <div class="window-title">应用程序</div>
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
        <!-- 应用图标列表 -->
        <div v-if="!showMusicPlayer" class="app-icons">
          <div class="app-icon" @click="openMusicPlayer">
            <img src="../../base-Sources/icon/musicPlayer.svg" alt="音乐播放器" class="icon-image">
            <div class="icon-label">音乐</div>
          </div>
        </div>
        
        <!-- 音乐播放器界面 -->
        <div v-else class="music-player-wrapper">
          <musicPlayer-App @return-to-app="showAppList" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import musicPlayerApp from './musicPlayer-App.vue';

export default {
  name: 'app-Page',
  components: {
    musicPlayerApp
  },
  data() {
    return {
      isVisible: true,
      isMaximized: false,
      showMusicPlayer: false
    }
  },
  methods: {
    handleClose() {
      this.$emit('close')
    },
    toggleMaximize() {
      this.isMaximized = !this.isMaximized
    },
    openMusicPlayer() {
      this.showMusicPlayer = true
    },
    showAppList() {
      this.showMusicPlayer = false
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
  padding: 0px;
  overflow-y: auto;
  background-color: #000;
  position: relative;
}

/* 应用图标样式 */
.app-icons {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  padding: 5px;
}

.app-icon {
  width: 60px;
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  padding: 10px;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.app-icon:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.icon-image {
  width: 50px;
  height: 50px;
  margin-bottom: 10px;
  object-fit: contain;
}

.icon-label {
  color: white;
  font-size: 14px;
  text-align: center;
}

/* 音乐播放器包装器 */
.music-player-wrapper {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  overflow: hidden;
}
</style>