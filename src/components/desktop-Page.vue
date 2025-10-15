<template>
  <div class="desktop-page">
    <!-- 图标容器 -->
    <div class="desktop-icons">
      <!-- Personal图标 -->
      <div 
        class="desktop-icon" 
        @mousedown="startDrag('personal', $event)"
        :style="{ left: icons.personal.x + 'px', top: icons.personal.y + 'px' }"
      >
        <img src="/base-Sources/icon/personal.svg" alt="个人" class="icon-image">
        <div class="icon-label">个人</div>
      </div>
      
      <!-- 应用程序图标 -->
      <div 
        class="desktop-icon" 
        @mousedown="startDrag('app', $event)"
        :style="{ left: icons.app.x + 'px', top: icons.app.y + 'px' }"
      >
        <img src="/base-Sources/icon/app.svg" alt="应用程序" class="icon-image">
        <div class="icon-label">应用程序</div>
      </div>
      
      <!-- Terminal图标 -->
      <div 
        class="desktop-icon" 
        @mousedown="startDrag('terminal', $event)"
        :style="{ left: icons.terminal.x + 'px', top: icons.terminal.y + 'px' }"
      >
        <img src="/base-Sources/icon/terminal.svg" alt="终端" class="icon-image">
        <div class="icon-label">终端</div>
      </div>
      
      <!-- Setting图标 -->
      <div 
        class="desktop-icon" 
        @mousedown="startDrag('setting', $event)"
        :style="{ left: icons.setting.x + 'px', top: icons.setting.y + 'px' }"
      >
        <img src="/base-Sources/icon/setting.svg" alt="设置" class="icon-image">
        <div class="icon-label">设置</div>
      </div>
    </div>
    
    <!-- 子页面组件 -->
    <PersonalPage v-if="showPersonal" @close="hidePersonalPage"/>
    <AppPage v-if="showApp" @close="hideAppPage"/>
    <TerminalPage v-if="showTerminal" @close="hideTerminalPage"/>
    <SettingPage v-if="showSetting" @close="hideSettingPage"/>
  </div>
</template>

<script>
import PersonalPage from './personal-Page.vue'
import TerminalPage from './terminal-Page.vue'
import SettingPage from './setting-Page.vue'
import AppPage from './app-Page.vue'

export default {
  name: 'desktop-Page',
  components: {
    PersonalPage,
    TerminalPage,
    SettingPage,
    AppPage
  },
  data() {
    return {
      showPersonal: false,
      showTerminal: false,
      showSetting: false,
      showApp: false,
      // 存储图标位置
      icons: {
        personal: { x: 20, y: 20 },
        app: { x: 20, y: 20 + 60 + 15 },
        terminal: { x: 20, y: 20 + (60 + 15) * 2 },
        setting: { x: 20, y: 20 + (60 + 15) * 3 }
      },
      // 拖动状态
      isDragging: false,
      currentIcon: null,
      offset: { x: 0, y: 0 },
      // 边界信息
      bounds: { x: 0, y: 0, width: 0, height: 0 },
      // 图标尺寸
      iconSize: { width: 60, height: 60 },
      // 图标间距
      iconSpacing: 15,
      // 用于判断是否为点击操作的阈值（像素）
      clickThreshold: 5,
      // 鼠标按下时的起始位置
      startPos: { x: 0, y: 0 },
      // 是否为纯点击操作
      isClick: true
    }
  },
  mounted() {
    // 初始化边界
    this.updateBounds()
    // 监听窗口大小变化
    window.addEventListener('resize', this.updateBounds)
    // 监听鼠标移动和释放事件
    // 使用passive: false优化响应速度
    document.addEventListener('mousemove', this.onMouseMove, { passive: false })
    document.addEventListener('mouseup', this.handleMouseUp, { passive: false })
  },
  beforeUnmount() {
    // 清理事件监听器
    window.removeEventListener('resize', this.updateBounds)
    document.removeEventListener('mousemove', this.onMouseMove)
    document.removeEventListener('mouseup', this.handleMouseUp)
  },
  methods: {
    showPersonalPage() {
      this.showPersonal = true
    },
    hidePersonalPage() {
      this.showPersonal = false
    },
    showTerminalPage() {
      this.showTerminal = true
    },
    hideTerminalPage() {
      this.showTerminal = false
    },
    showSettingPage() {
      this.showSetting = true
    },
    hideSettingPage() {
      this.showSetting = false
    },
    showAppPage() {
      this.showApp = true
    },
    hideAppPage() {
      this.showApp = false
    },
    
    // 更新边界信息
    updateBounds() {
      // DefaultDisplay容器是视口的80%宽高，居中显示
      // 考虑到20px的内边距
      const viewportWidth = window.innerWidth
      const viewportHeight = window.innerHeight
      const containerWidth = viewportWidth * 0.8
      const containerHeight = viewportHeight * 0.8
      const containerLeft = (viewportWidth - containerWidth) / 2
      const containerTop = (viewportHeight - containerHeight) / 2
      
      this.bounds = {
        x: containerLeft + 20, // 左边界 + 内边距
        y: containerTop + 20,  // 上边界 + 内边距
        width: containerWidth - 40,  // 宽度 - 左右内边距
        height: containerHeight - 40 // 高度 - 上下内边距
      }
    },
    
    // 开始拖动 - 鼠标按住时激活拖动和点击检测
    startDrag(iconId, event) {
      // 阻止默认行为
      event.preventDefault()
      
      // 记录鼠标按下的位置
      this.startPos = {
        x: event.clientX,
        y: event.clientY
      }
      
      // 设置拖动状态
      this.isDragging = true
      this.currentIcon = iconId
      this.isClick = true  // 默认为点击操作，后续移动超过阈值会改为拖动
      
      // 计算鼠标相对于图标左上角的偏移量
      const iconElement = event.currentTarget
      const rect = iconElement.getBoundingClientRect()
      this.offset = {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top
      }
      
      // 提高当前拖动图标的层级，使其显示在其他图标上方
      iconElement.style.zIndex = 100
    },
    
    // 鼠标移动 - 仅在鼠标按住状态下更新图标位置
    onMouseMove(event) {
      if (!this.isDragging || !this.currentIcon) return
      
      // 判断是否为拖动操作（移动距离超过阈值）
      const dx = Math.abs(event.clientX - this.startPos.x)
      const dy = Math.abs(event.clientY - this.startPos.y)
      if (dx > this.clickThreshold || dy > this.clickThreshold) {
        this.isClick = false
      }
      
      // 计算新位置，并考虑边界限制
      let newX = event.clientX - this.offset.x - this.bounds.x
      let newY = event.clientY - this.offset.y - this.bounds.y
      
      // 边界检查
      newX = Math.max(0, Math.min(newX, this.bounds.width - this.iconSize.width))
      newY = Math.max(0, Math.min(newY, this.bounds.height - this.iconSize.height))
      
      // 更新图标位置
      this.icons[this.currentIcon].x = newX
      this.icons[this.currentIcon].y = newY
    },
    
    // 处理鼠标释放事件 - 阻止事件冒泡防止触发图标点击
    handleMouseUp(event) {
      // 阻止事件冒泡，防止结束拖动时激活图标点击
      event.preventDefault()
      event.stopPropagation()
      
      this.onMouseUp()
    },
    
    // 鼠标释放 - 固定图标位置并结束拖动，判断是否为点击操作
    onMouseUp() {
      if (!this.isDragging || !this.currentIcon) return
      
      // 判断是否为纯点击操作
      if (this.isClick) {
        // 根据图标ID执行相应的点击操作
        if (this.currentIcon === 'personal') {
          this.showPersonalPage()
        } else if (this.currentIcon === 'app') {
          this.showAppPage()
        } else if (this.currentIcon === 'terminal') {
          this.showTerminalPage()
        } else if (this.currentIcon === 'setting') {
          this.showSettingPage()
        }
      }
      
      // 重置拖动状态
      this.isDragging = false
      this.currentIcon = null
      this.offset = { x: 0, y: 0 }
      this.isClick = true
      
      // 重置所有图标的z-index
      const iconElements = document.querySelectorAll('.desktop-icon')
      iconElements.forEach(icon => {
        icon.style.zIndex = 1
      })
    }
  }
}
</script>

<style scoped>
.desktop-page {
  width: 100%;
  height: 100%;
  background-color: #000;
  position: relative;
  overflow: hidden;
}

.desktop-icons {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; /* 确保容器不干扰鼠标事件 */
}

.desktop-icon {
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  transition: transform 0.2s ease;
  position: absolute;
  pointer-events: auto; /* 确保图标可以接收鼠标事件 */
  z-index: 1; /* 默认层级 */
  user-select: none; /* 防止拖动时选中文字 */
  width: 60px; /* 固定宽度 */
}

.desktop-icon:hover {
  transform: scale(1.1);
}

.icon-image {
  width: 40px;
  height: 40px;
  margin-bottom: 5px;
}

.icon-label {
  color: white;
  font-size: 12px;
  text-align: center;
}
</style>