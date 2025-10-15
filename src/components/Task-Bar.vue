<template>
  <div class="task-bar">
    <div class="system-time">{{ currentTime }}</div>
    <button class="shutdown-button" @click="showShutdownDialog" title="关机">
      <img src="../../base-Sources/icon/shutdown.svg" alt="关机" class="shutdown-icon">
    </button>
  </div>
  
  <!-- 关机确认弹窗 -->
  <div v-if="showDialog" class="shutdown-dialog-overlay">
    <div class="shutdown-dialog">
      <div class="dialog-content">
        <h2>是否关机？</h2>
      </div>
      <div class="dialog-buttons">
        <button class="dialog-button shutdown-confirm-button" @click="shutdown">关机</button>
        <button class="dialog-button cancel-button" @click="hideShutdownDialog">取消</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Task-Bar',
  data() {
    return {
      currentTime: '',
      showDialog: false
    }
  },
  mounted() {
    // 初始化时间
    this.updateTime();
    // 设置定时器每秒更新时间
    this.timeInterval = setInterval(() => {
      this.updateTime();
    }, 1000);
  },
  beforeUnmount() {
    // 清理定时器
    if (this.timeInterval) {
      clearInterval(this.timeInterval);
    }
  },
  methods: {
    updateTime() {
      const now = new Date();
      const hours = now.getHours().toString().padStart(2, '0');
      const minutes = now.getMinutes().toString().padStart(2, '0');
      const seconds = now.getSeconds().toString().padStart(2, '0');
      this.currentTime = `${hours}:${minutes}:${seconds}`;
    },
    
    showShutdownDialog() {
      this.showDialog = true;
    },
    
    hideShutdownDialog() {
      this.showDialog = false;
    },
    
    shutdown() {
      // 关闭网页
      window.close();
    }
  }
}
</script>

<style scoped>
.task-bar {
  position: fixed;
  top: 0px; /* margin-up:2px */
  left: 0; /* 定位在(0,0) */
  width: 100%; /* 宽100% */
  height: 3%; /* 高3% */
  background-color: #d0b3b3; /* 颜色为纯白 */
  z-index: 998; /* 显示层级仅次于DefaultDisplay的999 */
  pointer-events: none; /* 不干扰鼠标事件 */
}

.system-time {
  position: absolute;
  left: 2%; /* x坐标2% */
  top: 30%; 
  transform: translateY(-50%); 
  font-family: monospace;
  color: black;
  white-space: nowrap;
  max-height: 100%; /* 确保不超过Task-Bar高度 */
  font-size: 2.5vh; /* 字体大小，可根据需要调整 */
  font-weight: 1000; /* 加粗字体 */
}

.shutdown-button {
  position: absolute;
  right: 2%; /* 水平距右侧2% */
  top: 40%;
  transform: translateY(-50%); /* 垂直居中 */
  background: none;
  border: none;
  padding: 0;
  cursor: pointer;
  height: 2vh; /* 高度3vh */
  pointer-events: auto;
}

.shutdown-button:hover {
  opacity: 0.8;
}

.shutdown-icon {
  height: 100%;
  width: auto;
  filter: invert(0%); /* 确保颜色为纯黑 */
}

/* 关机确认弹窗样式 */
.shutdown-dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.shutdown-dialog {
  background-color: #000000; /* 背景纯黑 */
  border: 1.5px solid #ffffff; /* 描边为白色2px */
  border-radius: 0; /* 圆角为0 */
  padding: 20px;
  width: 300px;
  text-align: center;
}

.dialog-content h2 {
  color: #ffffff;
  margin-bottom: 20px;
}

.dialog-buttons {
  display: flex;
  justify-content: space-around;
  gap: 20px;
}

.dialog-button {
  background-color: #000000; /* 背景纯黑 */
  border: 1px solid #ffffff; /* 描边为白色2px */
  color: #ffffff;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 0; /* 圆角为0 */
  transition: background-color 0.3s;
}

.dialog-button:hover {
  background-color: #333333; /* 鼠标hover提示效果 */
}
</style>