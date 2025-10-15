<template>
  <div class="test-page">
    <h1>测试页面</h1>
    
    <!-- 按钮计数功能 -->
    <div class="counter-section">
      <h2>按钮计数器</h2>
      <button @click="incrementCount">点击计数</button>
      <p>当前计数: {{ count }}</p>
    </div>
    
    <!-- 文字显示功能 -->
    <div class="text-section">
      <h2>文字显示</h2>
      <input v-model="inputText" placeholder="输入文字" />
      <p>您输入的文字: {{ inputText }}</p>
    </div>
    
    <!-- 颜色块显示功能 -->
    <div class="color-section">
      <h2>颜色块显示</h2>
      <div class="color-grid">
        <div class="color-block" style="background-color: #ff0000;">红色</div>
        <div class="color-block" style="background-color: #00ff00;">绿色</div>
        <div class="color-block" style="background-color: #0000ff;">蓝色</div>
        <div class="color-block" style="background-color: #ffff00;">黄色</div>
        <div class="color-block" style="background-color: #ff00ff;">紫色</div>
        <div class="color-block" style="background-color: #00ffff;">青色</div>
      </div>
    </div>
    
    <!-- CRT效果将通过父组件应用 -->
  </div>
</template>

<script>
export default {
  name: 'TestPage',
  data() {
    return {
      count: 0,
      inputText: ''
    }
  },
  methods: {
    incrementCount() {
      this.count++
    }
  }
}
</script>

<style scoped>
.test-page {
  padding: 40px;
  text-align: center;
  font-family: 'Arial', sans-serif;
  color: #ffffff;
}

.counter-section,
.text-section,
.color-section {
  margin-bottom: 40px;
  padding: 20px;
  border: 2px solid #444;
  border-radius: 10px;
  background-color: rgba(0, 0, 0, 0.5);
}

button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #666;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 15px;
}

button:hover {
  background-color: #888;
}

input {
  padding: 10px;
  font-size: 16px;
  margin-bottom: 15px;
  width: 300px;
  border-radius: 5px;
  border: 1px solid #666;
}

.color-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  max-width: 400px;
  margin: 0 auto;
}

.color-block {
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 5px;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}
</style>