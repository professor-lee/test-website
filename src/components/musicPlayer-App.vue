<template>
  <div class="music-player-container" @keydown.esc="handleEsc" tabindex="0">
    <!-- 左侧歌曲选择栏 -->
    <div class="song-list-container" :class="{ 'collapsed': isCollapsed }">
      <div class="song-list-header" @click="toggleCollapse">
        <span class="collapse-icon">{{ isCollapsed ? '▶' : '◀' }}</span>
      </div>
      <div class="song-list">
        <div 
          v-for="(song, index) in songList" 
          :key="index"
          class="song-item" 
          :class="{ 'active': currentSongIndex === index }"
          @click="playSong(index)"
        >
          <span class="song-name">{{ song.name }}</span>
        </div>
      </div>
      <div class="esc-hint">按下esc退出播放器</div>
    </div>

    <!-- 右侧播放区域 -->
    <div class="playback-container">
      <!-- 歌词显示 -->
      <div class="lyrics-container">
        <div class="lyrics" ref="lyricsRef">
          {{ currentLyric }}
        </div>
      </div>

      <!-- 播放控制 -->
      <div class="player-controls">
        <button class="control-button" @click="playPrevious">
          ◀◀
        </button>
        <button class="control-button play-pause" @click="togglePlayPause">
          {{ isPlaying ? '| |' : '▶' }}
        </button>
        <button class="control-button" @click="playNext">
          ▶▶
        </button>
      </div>

      <!-- 进度条 -->
      <div class="progress-bar-container">
        <div class="progress-bar" @click="handleProgressClick" ref="progressBarRef">
          <div class="progress-bar-fill" :style="{ width: progressPercentage + '%' }"></div>
          <div class="progress-bar-handle" :style="{ left: progressPercentage + '%' }" 
               @mousedown="startDrag" @touchstart="startDrag"></div>
        </div>
        <div class="time-display">
          <span class="current-time">{{ formatTime(currentTime) }}</span>
          <span class="time-separator">/</span>
          <span class="total-time">{{ formatTime(totalTime) }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, watch } from 'vue';

export default {
  name: 'musicPlayer-App',
  emits: ['return-to-app'],
  setup(props, { emit }) {
    const songList = ref([]);
    const currentSongIndex = ref(-1);
    const isPlaying = ref(false);
    const isCollapsed = ref(false);
    const currentLyric = ref('暂无歌词');
    const lyricsRef = ref(null);
    const audioElement = ref(null);
    const scrollTimer = ref(null);
    const progressPercentage = ref(0);
    const progressUpdateTimer = ref(null);
    const currentTime = ref(0);
    const totalTime = ref(0);
    const progressBarRef = ref(null);
    const isDragging = ref(false);
    const lyricsData = ref([]); // 存储解析后的歌词数据
    const currentLyricIndex = ref(-1); // 当前显示的歌词索引

    // 获取歌曲列表
    const loadSongs = async () => {
      try {
        // 在实际应用中，这里会通过API或文件系统获取歌曲列表
        // 模拟从./base-Sources/music文件夹获取歌曲
        songList.value = [
          {
            name: '東京フラッシュ',
            artist: 'Vaundy',
            path: '/base-Sources/music/東京フラッシュ-Vaundy.mp3'
          },
          {
            name: '本当は夜の端まで、',
            artist: 'MAISONdes & おおお & くじら',
            path: '/base-Sources/music/本当は夜の端まで、-MAISONdes&おおお&くじら.mp3'
          }
          // 更多歌曲会根据文件夹内容动态添加
        ];
      } catch (error) {
        console.error('加载歌曲列表失败:', error);
      }
    };

    // 播放指定歌曲
    const playSong = (index) => {
      if (index < 0 || index >= songList.value.length) return;

      currentSongIndex.value = index;
      const song = songList.value[index];

      // 加载歌词
      loadLyrics(song);

      // 在实际应用中，这里会创建或更新音频元素并播放
      if (!audioElement.value) {
        audioElement.value = new Audio(song.path);
        audioElement.value.addEventListener('ended', playNext);
        // 添加时间更新监听器
        audioElement.value.addEventListener('timeupdate', updateProgress);
      } else {
        audioElement.value.pause();
        audioElement.value.src = song.path;
        // 重新添加时间更新监听器
        audioElement.value.addEventListener('timeupdate', updateProgress);
      }

      audioElement.value.play().then(() => {
        isPlaying.value = true;
        // 开始进度条更新
        startProgressUpdate();
      }).catch(error => {
        console.error('播放失败:', error);
      });
    };

    // 切换播放/暂停
    const togglePlayPause = () => {
      if (!audioElement.value) return;

      if (isPlaying.value) {
        audioElement.value.pause();
      } else {
        audioElement.value.play();
      }
      isPlaying.value = !isPlaying.value;
    };

    // 播放上一首
    const playPrevious = () => {
      if (songList.value.length === 0) return;
      const newIndex = (currentSongIndex.value - 1 + songList.value.length) % songList.value.length;
      playSong(newIndex);
    };

    // 播放下一首
    const playNext = () => {
      if (songList.value.length === 0) return;
      const newIndex = (currentSongIndex.value + 1) % songList.value.length;
      playSong(newIndex);
    };

    // 切换歌曲列表折叠状态
    const toggleCollapse = () => {
      isCollapsed.value = !isCollapsed.value;
    };

    // 处理Esc键返回app-Page
    const handleEsc = () => {
      emit('return-to-app');
    };

    // 更新进度条
    const updateProgress = () => {
      // 如果正在拖动，则不自动更新进度条
      if (isDragging.value) return;
      
      if (audioElement.value && audioElement.value.duration) {
        const percentage = (audioElement.value.currentTime / audioElement.value.duration) * 100;
        progressPercentage.value = Math.min(100, Math.max(0, percentage));
        currentTime.value = audioElement.value.currentTime;
        totalTime.value = audioElement.value.duration;
        
        // 更新歌词显示
        updateLyrics(currentTime.value);
      }
    };

    // 开始进度条更新
    const startProgressUpdate = () => {
      // 清除之前的定时器
      if (progressUpdateTimer.value) {
        clearInterval(progressUpdateTimer.value);
      }
      
      // 设置定时器更新进度条
      progressUpdateTimer.value = setInterval(() => {
        updateProgress();
      }, 100);
    };

    // 格式化时间（秒转换为分:秒）
    const formatTime = (seconds) => {
      if (!seconds || isNaN(seconds)) return '00:00';
      
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    };

    // 解析SRT歌词文件格式
    const parseLyrics = (lyricsText) => {
      const lines = lyricsText.split('\n');
      const lyrics = [];
      let i = 0;
      
      while (i < lines.length) {
        // 跳过空行
        if (!lines[i].trim()) {
          i++;
          continue;
        }
        
        // 解析序号
        const index = parseInt(lines[i].trim());
        if (isNaN(index)) {
          i++;
          continue;
        }
        i++;
        
        // 解析时间轴
        const timeLine = lines[i]?.trim();
        if (!timeLine || !timeLine.includes('-->')) {
          i++;
          continue;
        }
        
        const [startTimeStr, endTimeStr] = timeLine.split('-->').map(t => t.trim());
        
        // 转换时间格式 HH:MM:SS,mmm -> 秒
        const startTime = parseTimeToSeconds(startTimeStr);
        const endTime = parseTimeToSeconds(endTimeStr);
        
        i++;
        
        // 解析歌词内容
        const lyricLines = [];
        while (i < lines.length && lines[i]?.trim()) {
          lyricLines.push(lines[i].trim());
          i++;
        }
        
        if (lyricLines.length > 0) {
          lyrics.push({
            index,
            startTime,
            endTime,
            text: lyricLines.join(' ')
          });
        }
      }
      
      return lyrics;
    };

    // 将时间字符串转换为秒数
    const parseTimeToSeconds = (timeStr) => {
      // 格式: HH:MM:SS,mmm 或 HH:MM:SS.mmm
      const timeParts = timeStr.replace(',', '.').split(':');
      if (timeParts.length !== 3) return 0;
      
      const hours = parseInt(timeParts[0]) || 0;
      const minutes = parseInt(timeParts[1]) || 0;
      const seconds = parseFloat(timeParts[2]) || 0;
      
      return hours * 3600 + minutes * 60 + seconds;
    };

    // 加载歌词文件
    const loadLyrics = async (song) => {
      try {
        // 根据歌曲路径生成歌词文件路径
        const lyricPath = song.path.replace('.mp3', '.srt');
        
        // 使用fetch加载歌词文件
        const response = await fetch(lyricPath);
        if (!response.ok) {
          throw new Error('歌词文件不存在');
        }
        
        const lyricsText = await response.text();
        const parsedLyrics = parseLyrics(lyricsText);
        
        lyricsData.value = parsedLyrics;
        currentLyricIndex.value = -1;
        currentLyric.value = `${song.name} - ${song.artist}`;
        
      } catch (error) {
        console.warn('加载歌词失败:', error.message);
        // 如果歌词加载失败，显示默认信息
        lyricsData.value = [];
        currentLyricIndex.value = -1;
        currentLyric.value = `${song.name} - ${song.artist}`;
      }
    };

    // 根据当前时间更新显示的歌词
    const updateLyrics = (currentTime) => {
      if (lyricsData.value.length === 0) return;
      
      // 查找当前时间对应的歌词
      let newIndex = -1;
      for (let i = 0; i < lyricsData.value.length; i++) {
        const lyric = lyricsData.value[i];
        if (currentTime >= lyric.startTime && currentTime < lyric.endTime) {
          newIndex = i;
          break;
        }
      }
      
      // 如果找到了新的歌词，更新显示
      if (newIndex !== currentLyricIndex.value) {
        currentLyricIndex.value = newIndex;
        if (newIndex >= 0) {
          currentLyric.value = lyricsData.value[newIndex].text;
        } else {
          currentLyric.value = `${songList.value[currentSongIndex.value]?.name || ''} - ${songList.value[currentSongIndex.value]?.artist || ''}`;
        }
      }
    };

    // 处理进度条点击
    const handleProgressClick = (event) => {
      if (!audioElement.value || !progressBarRef.value) return;
      
      const rect = progressBarRef.value.getBoundingClientRect();
      const clickX = event.clientX - rect.left;
      const width = rect.width;
      const percentage = Math.max(0, Math.min(100, (clickX / width) * 100));
      
      // 设置进度
      setProgress(percentage);
    };

    // 开始拖动
    const startDrag = (event) => {
      if (!audioElement.value || !progressBarRef.value) return;
      
      isDragging.value = true;
      
      // 阻止默认行为
      event.preventDefault();
      
      // 添加全局事件监听器
      document.addEventListener('mousemove', handleDrag);
      document.addEventListener('mouseup', stopDrag);
      document.addEventListener('touchmove', handleDrag);
      document.addEventListener('touchend', stopDrag);
      
      // 暂停进度条自动更新
      if (progressUpdateTimer.value) {
        clearInterval(progressUpdateTimer.value);
      }
    };

    // 处理拖动
    const handleDrag = (event) => {
      if (!isDragging.value || !progressBarRef.value) return;
      
      const rect = progressBarRef.value.getBoundingClientRect();
      const clientX = event.clientX || (event.touches && event.touches[0].clientX);
      const dragX = clientX - rect.left;
      const width = rect.width;
      const percentage = Math.max(0, Math.min(100, (dragX / width) * 100));
      
      // 更新进度显示
      progressPercentage.value = percentage;
      
      // 更新当前时间显示
      if (audioElement.value && audioElement.value.duration) {
        currentTime.value = (percentage / 100) * audioElement.value.duration;
      }
    };

    // 停止拖动
    const stopDrag = () => {
      if (!isDragging.value) return;
      
      isDragging.value = false;
      
      // 移除全局事件监听器
      document.removeEventListener('mousemove', handleDrag);
      document.removeEventListener('mouseup', stopDrag);
      document.removeEventListener('touchmove', handleDrag);
      document.removeEventListener('touchend', stopDrag);
      
      // 设置实际进度
      if (audioElement.value) {
        setProgress(progressPercentage.value);
      }
      
      // 恢复进度条自动更新
      if (audioElement.value && isPlaying.value) {
        startProgressUpdate();
      }
    };

    // 设置进度
    const setProgress = (percentage) => {
      if (!audioElement.value) return;
      
      progressPercentage.value = percentage;
      
      if (audioElement.value.duration) {
        const newTime = (percentage / 100) * audioElement.value.duration;
        audioElement.value.currentTime = newTime;
        currentTime.value = newTime;
        
        // 更新歌词显示
        updateLyrics(newTime);
      }
    };

    // 歌词滚动效果
    const setupLyricsScroll = () => {
      if (!lyricsRef.value) return;

      const resetScroll = () => {
        if (lyricsRef.value) {
          lyricsRef.value.scrollLeft = 0;
        }
      };

      const startScroll = () => {
        if (!lyricsRef.value) return;
        
        // 清除之前的定时器
        if (scrollTimer.value) {
          clearInterval(scrollTimer.value);
        }

        // 检查是否需要滚动
        if (lyricsRef.value.scrollWidth <= lyricsRef.value.clientWidth) {
          return;
        }

        // 开始滚动
        scrollTimer.value = setInterval(() => {
          if (lyricsRef.value) {
            lyricsRef.value.scrollLeft += 1;
            // 如果滚动到末尾，重置
            if (lyricsRef.value.scrollLeft >= lyricsRef.value.scrollWidth - lyricsRef.value.clientWidth) {
              clearInterval(scrollTimer.value);
              setTimeout(resetScroll, 1000);
              setTimeout(startScroll, 2000);
            }
          }
        }, 50);
      };

      // 监听歌词变化，重新开始滚动
      watch(currentLyric, () => {
        if (scrollTimer.value) {
          clearInterval(scrollTimer.value);
        }
        setTimeout(startScroll, 1000);
      });

      // 初始设置
      setTimeout(startScroll, 1000);
    };

    // 组件挂载时
    onMounted(() => {
      loadSongs();
      setupLyricsScroll();
      // 聚焦以捕获键盘事件
      document.querySelector('.music-player-container')?.focus();
    });

    // 组件卸载时
    onUnmounted(() => {
      if (audioElement.value) {
        audioElement.value.pause();
        audioElement.value = null;
      }
      if (scrollTimer.value) {
        clearInterval(scrollTimer.value);
      }
      if (progressUpdateTimer.value) {
        clearInterval(progressUpdateTimer.value);
      }
    });

    return {
      songList,
      currentSongIndex,
      isPlaying,
      isCollapsed,
      currentLyric,
      lyricsRef,
      progressBarRef,
      progressPercentage,
      currentTime,
      totalTime,
      loadSongs,
      playSong,
      togglePlayPause,
      playPrevious,
      playNext,
      toggleCollapse,
      handleEsc,
      handleProgressClick,
      startDrag,
      formatTime
    };
  }
};
</script>

<style scoped>
.music-player-container {
  width: 100%;
  height: 100%;
  background-color: #000;
  color: #fff;
  display: flex;
  border: 1px solid #fff;
  outline: none; /* 移除聚焦轮廓 */
  box-sizing: border-box;
}

/* 左侧歌曲选择栏 */
.song-list-container {
  width: 20%;
  height: 100%;
  background-color: #000;
  border-right: 1px solid #fff;
  display: flex;
  flex-direction: column;
  transition: width 0.3s ease;
}

.song-list-container.collapsed {
  width: 30px;
}

.song-list-header {
      padding: 5px;
      border-bottom: 1px solid #fff;
      cursor: pointer;
      display: flex;
      align-items: center;
    }

    .collapse-icon {
      font-size: 10px;
      width: 10px;
      height: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
    }

.song-list {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}

.song-list-container.collapsed .song-list {
  display: none;
}

.song-item {
  padding: 10px;
  cursor: pointer;
  border-radius: 4px;
  margin-bottom: 5px;
  white-space: nowrap;
  overflow: hidden;
  position: relative;
}

.song-item:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.song-item.active {
  background-color: rgba(255, 255, 255, 0.2);
}

/* 歌曲名称滚动动画 */
.song-item:hover .song-name {
  animation: scrollText 8s linear infinite;
}

.song-item .song-name {
  display: inline-block;
  min-width: 100%;
}

@keyframes scrollText {
  0% {
    transform: translateX(0%);
  }
  100% {
    transform: translateX(-100%);
  }
}

/* ESC提示文字样式 */
.esc-hint {
  text-align: center;
  color: rgba(255, 255, 255, 0.6);
  font-size: clamp(8px, 5vw, 12px); /* 动态字体大小，最小8px，最大12px，基于视窗宽度的3% */
  padding: 10px;
  margin-top: auto;
  white-space: nowrap;
  user-select: none;
  width: 100%;
  box-sizing: border-box;
  overflow: hidden;
  text-overflow: ellipsis;
}

.song-item:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.song-item.active {
  background-color: rgba(255, 255, 255, 0.2);
}

/* 右侧播放区域 */
.playback-container {
  flex: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 10px;
  box-sizing: border-box;
  overflow: hidden;
}

/* 歌词显示 */
.lyrics-container {
  width: 100%;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  overflow: hidden;
  text-align: center;
}

.lyrics {
  font-size: 16px;
  line-height: 1.4;
  white-space: normal;
  word-wrap: break-word;
  max-width: 90%;
  transition: all 0.3s ease;
  text-shadow: 0 0 5px rgba(255, 255, 255, 0.5);
}

/* 播放控制 */
.player-controls {
  display: flex;
  gap: 20px;
  align-items: center;
  flex-wrap: wrap;
  justify-content: center;
}

.control-button {
  background-color: transparent;
  color: #fff;
  border: 1px solid #fff;
  width: clamp(60px, 5vw, 120px); /* 宽度为页面宽度的10%，限制在60-120px之间 */
  height: clamp(60px, 5vw, 120px); /* 高度与宽度相同 */
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: clamp(6px, 10vw, 12px); /* 字体大小为按钮宽度的10% */
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.3s ease;
  box-sizing: border-box;
}

.control-button:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.control-button.play-pause {
  width: clamp(70px, 5vw, 140px); /* 播放/暂停按钮与其他按钮相同宽度比例 */
  height: clamp(70px, 5vw, 140px);
  font-size: clamp(7px, 10vw, 14px); /* 字体大小为按钮宽度的10% */
}

/* 进度条样式 */
.progress-bar-container {
  width: 90%;
  margin-top: 20px;
  display: flex;
  align-items: center;
  gap: 15px;
}

.progress-bar {
  flex: 1;
  height: 2px;
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 1px;
  overflow: visible;
  position: relative;
  cursor: pointer;
}

.progress-bar-fill {
  height: 100%;
  background-color: #fff;
  border-radius: 1px;
  transition: width 0.1s ease;
}

/* 进度条拖动手柄 */
.progress-bar-handle {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 12px;
  height: 12px;
  background-color: #fff;
  border-radius: 50%;
  cursor: pointer;
  opacity: 0;
  transition: opacity 0.2s ease, transform 0.1s ease;
  box-shadow: 0 0 4px rgba(0, 0, 0, 0.5);
}

.progress-bar:hover .progress-bar-handle {
  opacity: 1;
}

.progress-bar-handle:hover {
  transform: translate(-50%, -50%) scale(1.2);
}

.progress-bar-handle:active {
  transform: translate(-50%, -50%) scale(1.1);
}

/* 时间显示样式 */
.time-display {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 14px;
  color: #fff;
  font-family: monospace;
  white-space: nowrap;
}

.current-time {
  color: #fff;
}

.time-separator {
  color: rgba(255, 255, 255, 0.7);
}

.total-time {
  color: rgba(255, 255, 255, 0.7);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .song-list-container {
    width: 30%;
  }
  
  .progress-bar-container {
    width: 95%;
    gap: 10px;
  }
  
  .time-display {
    font-size: 12px;
    gap: 3px;
  }
}
</style>