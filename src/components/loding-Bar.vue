<template>
  <div class="loding-bar-container">
    <div class="loding-bar">
      <div class="loding-bar-fill" :style="{ width: fillWidth + '%' }"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'loding-Bar',
  props: {
    duration: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      fillWidth: 0,
      isAnimating: false
    }
  },
  mounted() {
    this.startAnimation();
  },
  methods: {
    startAnimation() {
      this.isAnimating = true;
      const startTime = Date.now();
      
      const updateFill = () => {
        const elapsedTime = Date.now() - startTime;
        const progress = Math.min(elapsedTime / this.duration, 1);
        
        // 非线性进度
        const easeProgress = progress < 0.5 ? 2 * progress * progress : -1 + (4 - 2 * progress) * progress;
        
        this.fillWidth = easeProgress * 100;
        
        if (progress < 1) {
          requestAnimationFrame(updateFill);
        } else {
          this.isAnimating = false;
          this.$emit('animationComplete');
        }
      };
      
      requestAnimationFrame(updateFill);
    }
  }
}
</script>

<style scoped>
.loding-bar-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 2px;
}

.loding-bar {
  width: 50px;
  height: 3px;
  border: 1px solid #fff;
  border-radius: 3px;
  overflow: hidden;
  position: relative;
}

.loding-bar-fill {
  height: 100%;
  background: #fff;
  transition: width 0s linear;
  border-radius: 3px;
}
</style>