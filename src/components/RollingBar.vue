<template>
  <!-- 滚动条样式组件 - 此组件本身不渲染任何内容，仅提供全局滚动条样式 -->
</template>

<script>
export default {
  name: 'RollingBar'
}
</script>

<style>
/* 全局滚动条样式设置 */

/* 滚动条容器 - 设置背景为纯黑 */
::-webkit-scrollbar {
  width: 10px;  /* 垂直滚动条宽度 */
  height: 10px; /* 水平滚动条高度 */
}

/* 滚动条背景 - 纯黑色 */
::-webkit-scrollbar-track {
  background: #000000; /* 纯黑色背景 */
}

/* 滚动条滑块 - 纯白色长方形 */
::-webkit-scrollbar-thumb {
  background: #ffffff; /* 纯白色滑块 */
  border-radius: 0;    /* 无圆角，保持长方形样式 */
  border: none;        /* 无边框 */
}

/* 滚动条滑块悬停效果 */
::-webkit-scrollbar-thumb:hover {
  background: #f0f0f0; /* 稍微浅一点的白色，增强交互反馈 */
}

/* 滚动条角落（垂直和水平滚动条交汇处） */
::-webkit-scrollbar-corner {
  background: #000000; /* 保持与背景一致的纯黑色 */
}

/* Firefox 滚动条样式 */
html {
  scrollbar-width: thin;            /* 滚动条宽度 */
  scrollbar-color: #ffffff #000000; /* 滑块颜色 背景颜色 */
}
</style>