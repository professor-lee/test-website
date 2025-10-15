<template>
  <div class="default-display-container">
    <!-- SVG滤镜定义 - 用于实现边框畸变效果 -->
    <svg width="0" height="0" style="position: absolute;">
      <!-- 桶形畸变滤镜 - 应用于边框，强度动态调整 -->
      <filter id="border-barrel-distortion">
        <feMorphology operator="dilate" radius="0.04"/>
        <feDisplacementMap in="SourceGraphic" scale="0.8" xChannelSelector="R" yChannelSelector="G" x="50%" y="50%">
          <feTurbulence type="fractalNoise" baseFrequency="0.0008" numOctaves="1" result="turbulence"/>
          <!-- 使用feOffset调整中心点 -->
          <feOffset dx="-50%" dy="-50%" in="turbulence" result="offsetTurbulence"/>
          <!-- 使用feComposite将偏移后的湍流与原始图像组合 -->
          <feComposite in2="offsetTurbulence" operator="in"/>
        </feDisplacementMap>
      </filter>
      
      <!-- 枕形畸变滤镜 - 应用于边框，强度动态调整 -->
      <filter id="border-pincushion-distortion">
        <feComponentTransfer>
          <feFuncR type="table" tableValues="0 0.5 1"/>
          <feFuncG type="table" tableValues="0 0.5 1"/>
          <feFuncB type="table" tableValues="0 0.5 1"/>
        </feComponentTransfer>
        <feGaussianBlur in="SourceGraphic" stdDeviation="0.04" result="blurred"/>
        <feDisplacementMap in="SourceGraphic" in2="blurred" scale="0.6" xChannelSelector="R" yChannelSelector="G" x="50%" y="50%">
          <!-- 使用feOffset调整中心点 -->
          <feOffset dx="-50%" dy="-50%" in="blurred" result="offsetBlurred"/>
          <!-- 使用feComposite将偏移后的模糊与原始图像组合 -->
          <feComposite in2="offsetBlurred" operator="in"/>
        </feDisplacementMap>
      </filter>
    </svg>
    
    <!-- 边框容器 -->
    <div class="default-display-border">
      <!-- 内容容器 -->
      <div class="default-display-content">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DefaultDisplay',
  props: {
    // 畸变效果强度，取值范围0-1，默认0.04（4%）
    distortionIntensity: {
      type: Number,
      default: 0.04,
      validator: function(value) {
        return value >= 0 && value <= 1;
      }
    },
    // 畸变效果开关
    distortionEnabled: {
      type: Boolean,
      default: true
    }
  },
  data: function() {
    return {
      // 原始基准值，用于计算各参数
      baseValues: {
        barrelRadius: 0.04,
        barrelScale: 0.8,
        barrelFreq: 0.0008,
        pincushionSigma: 0.04,
        pincushionScale: 0.6
      }
    };
  },
  computed: {
    // 根据强度计算实际参数值
    computedValues: function() {
      // 如果畸变效果关闭，则返回0值
      if (!this.distortionEnabled) {
        return {
          barrelRadius: 0,
          barrelScale: 0,
          barrelFreq: 0,
          pincushionSigma: 0,
          pincushionScale: 0
        };
      }
      
      const factor = this.distortionIntensity / 0.04; // 以4%为基准比例
      return {
        barrelRadius: this.baseValues.barrelRadius * factor,
        barrelScale: this.baseValues.barrelScale * factor,
        barrelFreq: this.baseValues.barrelFreq * factor,
        pincushionSigma: this.baseValues.pincushionSigma * factor,
        pincushionScale: this.baseValues.pincushionScale * factor
      };
    }
  },
  mounted: function() {
    // 确保内容容器可以接收鼠标事件
    if (this.$el && this.$el.querySelector('.default-display-content')) {
      this.$el.querySelector('.default-display-content').style.pointerEvents = 'auto';
    }
    // 使用nextTick确保DOM已完全渲染后再更新参数
    var self = this;
    this.$nextTick(function() {
      self.updateFilterParameters();
    });
  },
  watch: {
    // 监听强度变化，更新滤镜参数
    distortionIntensity: {
      handler: function() {
        // 使用nextTick确保DOM已更新
        var self = this;
        this.$nextTick(function() {
          self.updateFilterParameters();
        });
      },
      immediate: true
    },
    // 监听开关状态变化
    distortionEnabled: {
      handler: function() {
        // 使用nextTick确保DOM已更新
        var self = this;
        this.$nextTick(function() {
          self.updateFilterParameters();
        });
      },
      immediate: true
    }
  },
  methods: {
    // 动态更新SVG滤镜参数
    updateFilterParameters: function() {
      // 确保组件已挂载完成且$el存在
      if (!this.$el) return;
      
      const values = this.computedValues;
      
      // 更新桶形畸变滤镜参数
      const svgElement = this.$el.querySelector('svg');
      if (svgElement) {
        const barrelFilter = svgElement.querySelector('#border-barrel-distortion');
        if (barrelFilter) {
          const morphElement = barrelFilter.querySelector('feMorphology');
          if (morphElement) morphElement.setAttribute('radius', values.barrelRadius);
          
          const displacementElement = barrelFilter.querySelector('feDisplacementMap');
          if (displacementElement) displacementElement.setAttribute('scale', values.barrelScale);
          
          const turbulenceElement = barrelFilter.querySelector('feTurbulence');
          if (turbulenceElement) turbulenceElement.setAttribute('baseFrequency', values.barrelFreq);
        }
        
        // 更新枕形畸变滤镜参数
        const pincushionFilter = svgElement.querySelector('#border-pincushion-distortion');
        if (pincushionFilter) {
          const blurElement = pincushionFilter.querySelector('feGaussianBlur');
          if (blurElement) blurElement.setAttribute('stdDeviation', values.pincushionSigma);
          
          const displacementElement = pincushionFilter.querySelector('feDisplacementMap');
          if (displacementElement) displacementElement.setAttribute('scale', values.pincushionScale);
        }
      }
      
      // 注意：CSS注释更新功能已移除，因为它可能导致样式问题
    }
  }
};
</script>

<style scoped>
/* 容器样式 - 设置为fixed定位，但z-index低于CrtShader */
.default-display-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  pointer-events: none;
  z-index: 999; /* 确保优先级低于CrtShader的1000 */
  display: flex; /* 添加flex布局 */
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
}

/* 边框容器 - 包含所有内容 */
.default-display-border {
  position: relative;
  width: 80%; /* 设置宽度为80% */
  height: 80%; /* 设置高度为80% */
  border: 2px solid #d0b3b3;
  border-radius: 2%; /* 设置圆角为2% */
  box-sizing: border-box;
  pointer-events: none;
  overflow: hidden;
  display: flex; /* 设置为flex布局 */
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
  filter: 
    url(#border-barrel-distortion) 
    url(#border-pincushion-distortion); /* 应用桶形和枕形畸变滤镜到边框，强度动态调整 */
}

/* 内容容器 - 确保所有内容显示在边框内 */
.default-display-content {
  position: relative;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding: 20px;
  overflow-y: auto;
  background: #000;
  /* 子组件内容会显示在这里 */
}

/* 确保内部内容可以响应鼠标事件 */
.default-display-content * {
  pointer-events: auto;
}
</style>