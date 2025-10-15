<template>
  <div class="crt-shader">
    <!-- SVG滤镜定义 - 用于实现畸变效果 -->
    <svg width="0" height="0" style="position: absolute;">
      <filter id="barrel-distortion">
        <!-- 桶形畸变滤镜 - 增强至100%强度，中心点设置为窗口中心 -->
        <feMorphology operator="dilate" radius="1"/>
        <feDisplacementMap in="SourceGraphic" scale="20" xChannelSelector="R" yChannelSelector="G" x="50%" y="50%">
          <feTurbulence type="fractalNoise" baseFrequency="0.02" numOctaves="1" result="turbulence"/>
          <!-- 使用feOffset调整中心点 -->
          <feOffset dx="-50%" dy="-50%" in="turbulence" result="offsetTurbulence"/>
          <!-- 使用feComposite将偏移后的湍流与原始图像组合 -->
          <feComposite in2="offsetTurbulence" operator="in"/>
        </feDisplacementMap>
      </filter>
      
      <!-- 枕形畸变滤镜 - 强度100%，中心点设置为窗口中心 -->
      <filter id="pincushion-distortion">
        <feComponentTransfer>
          <feFuncR type="table" tableValues="0 0.5 1"/>
          <feFuncG type="table" tableValues="0 0.5 1"/>
          <feFuncB type="table" tableValues="0 0.5 1"/>
        </feComponentTransfer>
        <feGaussianBlur in="SourceGraphic" stdDeviation="1" result="blurred"/>
        <feDisplacementMap in="SourceGraphic" in2="blurred" scale="15" xChannelSelector="R" yChannelSelector="G" x="50%" y="50%">
          <!-- 使用feOffset调整中心点 -->
          <feOffset dx="-50%" dy="-50%" in="blurred" result="offsetBlurred"/>
          <!-- 使用feComposite将偏移后的模糊与原始图像组合 -->
          <feComposite in2="offsetBlurred" operator="in"/>
        </feDisplacementMap>
      </filter>
    </svg>
    
    <!-- CRT效果容器 -->
    <div class="crt-screen" :style="{ 
      animationDuration: `${1000 / refreshRate}ms`,
      '--crt-scale': crtIntensity / 100
    }">
      <!-- 内容区域 - 这里会显示实际的页面内容 -->
      <div class="crt-content">
        <!-- 内容将由父组件提供 -->
        <slot></slot>
      </div>
      <!-- 扫描线效果元素 -->
      <div class="scanning-shade" ref="scanningShade" :style="{ display: scanlines ? 'block' : 'none' }"></div>
      <!-- 画面撕裂效果元素 -->
      <div class="screen-tear" ref="screenTear"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CrtShader',
  props: {
    // 刷新率参数，默认2Hz
    refreshRate: {
      type: Number,
      default: 2
    },
    // 扫描线开关，默认开启
    scanlines: {
      type: Boolean,
      default: true
    },
    // CRT滤镜强度，范围0-100，默认100
    crtIntensity: {
      type: Number,
      default: 100
    }
  },
  mounted() {
    // 启动扫描线动画
    this.startScanningAnimation();
    // 启动画面撕裂效果
    this.startScreenTearEffect();
  },
  methods: {
    startScanningAnimation() {
      const scanningShade = this.$refs.scanningShade;
      const screenTear = this.$refs.screenTear;
      if (!scanningShade || !screenTear) return;
      
      // 设置扫描线初始位置（使用像素单位）
      let position = -5; // 初始位置在屏幕外
      
      // 目标速度：20px/s
      const targetSpeedPxPerSec = 20;
      // 假设requestAnimationFrame的平均帧率为60fps
      const frameRate = 60;
      // 计算每帧移动的像素数
      const pixelsPerFrame = targetSpeedPxPerSec / frameRate;
      
      // 扫描线动画函数
      const animateScanline = () => {
        position += pixelsPerFrame; // 按固定速度移动
        
        // 获取屏幕高度
        const screenHeight = window.innerHeight;
        // 如果扫描线超出屏幕底部，重置到顶部
        if (position > screenHeight) {
          position = -5; // 重置到屏幕外
        }
        
        // 设置扫描线位置（使用像素单位）
        scanningShade.style.top = `${position}px`;
        
        // 根据scanlines prop决定是否创建画面撕裂效果
        if (this.scanlines && Math.random() > 0.95) { // 较低的概率创建撕裂效果
          this.createTearAtScanline(position, screenTear);
        }
        
        // 继续动画
        requestAnimationFrame(animateScanline);
      };
      
      // 开始动画
      animateScanline();
    },
    
    // 在扫描线位置创建画面撕裂效果
    createTearAtScanline(scanlinePosition, screenTear) {
      // 设置撕裂线高度和位置
      const height = Math.random() * 5 + 1; // 撕裂线高度1-6px
      
      // 设置撕裂线样式，位置与扫描线一致
      screenTear.style.height = `${height}px`;
      screenTear.style.top = `${scanlinePosition}px`;
      screenTear.style.opacity = '0.7';
      
      // 100ms后隐藏撕裂线
      setTimeout(() => {
        screenTear.style.opacity = '0';
      }, 100);
    },
    
    // 更新后的startScreenTearEffect方法，现在只初始化撕裂效果元素
    startScreenTearEffect() {
      // 保留这个方法以保持向后兼容性
      // 现在撕裂效果在扫描线动画中直接创建
    }
  }
}
</script>

<style scoped>
.crt-shader {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; /* 鼠标指针操作穿透 */
  z-index: 1000;
  overflow: hidden;
  /* 添加5%透明度纯白图层滤镜 */
  /* 使用::before作为最低优先级的滤镜图层 */
  &::before {
    content: '';
    display: block;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(255, 255, 255, 0.2); /* 5%透明度的白色 */
    pointer-events: none;
    z-index: -1; /* 确保优先级最低 */
  }
}

/* CRT屏幕容器 */
.crt-screen {
  position: relative;
  width: 100%;
  height: 100%;
  /* 确保伪元素能覆盖在上面 */
  animation: crt-flicker 500ms steps(1) infinite;
  /* steps(1) 确保是突变的，而不是平滑过渡 */
  transform: 
    perspective(1000px)
    rotateX(calc(2deg * var(--crt-scale)))
    rotateY(calc(-2deg * var(--crt-scale)))
    scale(calc(1 + 0.05 * var(--crt-scale))); /* 轻微放大 */
  filter: 
    brightness(calc(0.9 + 0.1 * var(--crt-scale)))
    contrast(calc(1 + 0.2 * var(--crt-scale)))
    url(#barrel-distortion)
    url(#pincushion-distortion); /* 同时应用桶形畸变和枕形畸变滤镜 */
}

/* 1. 扫描线（Scanlines）效果实现 - 使用伪元素叠加扫描线 */
.crt-screen::before {
  content: '';
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  /* 关键：使用重复的线性渐变创建水平条纹 */
  background: repeating-linear-gradient(
    0deg,
    rgba(0, 0, 0, calc(0.5 * var(--crt-scale))) 0px, /* 扫描线颜色，半透明黑色 */
    rgba(0, 0, 0, calc(0.5 * var(--crt-scale))) 1px, /* 扫描线宽度 */
    rgba(0, 0, 0, calc(0.1 * var(--crt-scale))) 1px, /* 屏幕缝隙颜色，较透明黑色 */
    rgba(0, 0, 0, calc(0.1 * var(--crt-scale))) 2px /* 屏幕缝隙宽度 */
  );
  pointer-events: none; /* 确保不影响鼠标交互 */
  opacity: var(--crt-scale);
}

/* 禁用扫描线效果的类已移除，改为直接控制scanning-shade元素的显示/隐藏 */
/* 这样可以确保即使在关闭状态下，后台仍继续计算扫描线位置 */

/* 2. 闪烁（Flicker）效果实现 - 定义闪烁关键帧动画 */
@keyframes crt-flicker {
  0%, 100% {
    opacity: 1;
  }
  50% {
    /* 快速降低屏幕透明度，模拟变暗的瞬间 */
    opacity: 0.98;
  }
}

/* 3. 从上到下扫描线效果 */
.scanning-shade {
  position: absolute;
  left: 0;
  width: 100%;
  height: 5px;
  background: rgba(255, 255, 255, calc(0.2 * var(--crt-scale)));
  pointer-events: none;
  z-index: 5;
  box-shadow: 0 0 10px rgba(255, 255, 255, calc(0.3 * var(--crt-scale)));
}

/* 4. 画面撕裂效果 */
.screen-tear {
  position: absolute;
  left: 0;
  width: 100%;
  background: rgba(255, 255, 255, calc(0.3 * var(--crt-scale)));
  pointer-events: none;
  z-index: 6;
  transition: opacity 0.1s ease-out;
}

/* 5. 几何失真（Geometric Distortion）效果实现 */
/* 6. 会聚误差（Convergence Error）效果实现 */
.crt-content {
  /* 包装需要显示的内容 */
  color: white;
  position: relative;
  width: 100%;
  height: 100%;
  /* 确保内容本身是白色的，利用阴影创建色散 */
  /* R (红色) 往左上方轻微偏移 */
  /* G (绿色) 不偏移，作为主体 */
  /* B (蓝色) 往右下方轻微偏移 */
  text-shadow: 
    calc(-1px * var(--crt-scale)) calc(-1px * var(--crt-scale)) 0 #FF0000, /* 红色阴影 */
    calc(1px * var(--crt-scale)) calc(1px * var(--crt-scale)) 0 #0000FF;   /* 蓝色阴影 */
}
</style>