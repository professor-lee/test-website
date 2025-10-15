<template>
  <div class="startup-animation" :style="{ clipPath: animationClipPath }">
    <div class="startup-logo">
        <img src="../../base-Sources/img/startup-Logo.png" alt="Logo" />
      </div>
    <loding-Bar :duration="loadingDuration" @animationComplete="onLodingBarComplete" />
  </div>
</template>

<script>
import lodingBar from './loding-Bar.vue'

export default {
  name: 'StartupAnimation',
  components: {
    lodingBar
  },
  data() {
    return {
      loadingDuration: 0,
      animationProgress: 0,
      animationClipPath: 'inset(0 0 0 0)'
    }
  },
  mounted() {
    // 删除鼠标检测与鼠标操作直接响应
    this.$el.style.pointerEvents = 'none';
    
    // 非线性随机生成填充时间（1-3秒）
    this.loadingDuration = 1000 + Math.random() * 2000;
  },
  methods: {
    onLodingBarComplete() {
      // 等待1秒后开始从上到下消除动画
      setTimeout(() => {
        this.startSlideUpAnimation();
      }, 1000);
    },
    
    startSlideUpAnimation() {
      const slideDuration = 2000; // 2秒
      const startTime = Date.now();
      
      const updateClipPath = () => {
        const elapsedTime = Date.now() - startTime;
        const progress = Math.min(elapsedTime / slideDuration, 1);
        
        // 线性进度
        const clipHeight = progress * 100;
        this.animationClipPath = `inset(${clipHeight}% 0 0 0)`; // 修改为从下到上消失
        
        if (progress < 1) {
          requestAnimationFrame(updateClipPath);
        } else {
          // 动画结束
          if (this.$emit) {
            this.$emit('animationEnd');
          }
        }
      };
      
      requestAnimationFrame(updateClipPath);
    }
  }
}
</script>

<style scoped>
.startup-animation {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #000;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 999;
  transition: clip-path 0.1s linear;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  /* 确保在任何情况下都保持居中 */
  pointer-events: none;
}

.startup-logo {
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
  padding: 0;
}

.startup-logo img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

/* 确保加载条居中且与logo有适当间距 */
:deep(.loding-bar-container) {
  margin-top: 15px;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>