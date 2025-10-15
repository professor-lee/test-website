<template>
  <div class="window-overlay" v-if="isVisible" @click.self="handleClose">
    <div class="window-container" :class="{ 'maximized': isMaximized }">
      <!-- 窗口标题栏 -->
      <div class="window-titlebar">
        <div class="window-title">终端</div>
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
        <div class="terminal-output">
          <p>Welcome !</p>
        <p>Here's terminal, type "help" to check supported command.</p>
          <!-- 历史命令输出 -->
        <div v-for="(line, index) in commandHistory" :key="index" class="command-line">
          <span class="command-prompt">{{ line.prompt || getCurrentPrompt() }}</span>
          <span class="command-text">{{ line.command }}</span>
          <div v-if="line.response" class="command-response">
            {{ line.response }}
          </div>
        </div>
        
        <!-- 当前命令输出（打字效果） -->
        <div v-if="isTyping" class="command-line">
          <span class="command-prompt">{{ getCurrentPrompt() }}</span>
          <span class="command-text">{{ currentCommand }}</span>
          <div class="command-response">
            {{ currentOutput }}<span class="terminal-cursor">█</span>
          </div>
        </div>
        <div v-else class="terminal-cursor">▂</div>
        </div>
        
        <!-- 输入区域 -->
        <div class="terminal-input-container">
          <span class="terminal-prompt">{{ getCurrentPrompt() }}</span>
          <input
            v-model="inputText"
            @keydown.enter="handleEnter"
            @keydown.up="handleUpKey"
            @keydown.down="handleDownKey"
            ref="terminalInput"
            class="terminal-input"
            :placeholder="isWaitingForPassword ? passwordPrompt : 'Type a command...'"
            :type="isWaitingForPassword ? 'password' : 'text'"
          />
          <span class="terminal-enter" :class="{ flash: inputText.trim() !== '' }" @click="handleEnter">↵</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'terminal-Page',
  data() {
    return {
      isVisible: true,
      isMaximized: false,
      inputText: '',
      isFlashing: false,
      commandHistory: [],
      historyIndex: -1, // 当前历史命令索引
      tempInput: '', // 临时存储用户输入的命令
      isTyping: false, // 是否正在输出中
      currentOutput: '', // 当前正在输出的内容
      outputIndex: 0, // 输出字符索引
      typingSpeed: 1, // 打字速度（毫秒）
      currentCommand: null, // 当前正在处理的命令
      
      // 交互模拟相关
      isWaitingForPassword: false, // 是否正在等待密码输入
      passwordPrompt: '', // 密码提示信息
      pendingCommand: null, // 等待执行的命令
      defaultPassword: 'professorLee', // 默认sudo密码
      
      // ping命令固定可连接网站列表
      pingWebsites: [
        { domain: 'https://www.google.com/search?q=google.com**', name: 'google.com**', service: '搜索引擎', location: '区域 CDN (香港/东京/新加坡)', delayMin: 30, delayMax: 70, lossMin: 0.0, lossMax: 0.2 },
        { domain: 'youtube.com', name: 'youtube.com', service: '视频分享', location: '区域 CDN (香港/东京/新加坡)', delayMin: 35, delayMax: 75, lossMin: 0.0, lossMax: 0.3 },
        { domain: 'facebook.com', name: 'facebook.com', service: '社交媒体', location: '区域 CDN (亚太节点)', delayMin: 60, delayMax: 120, lossMin: 0.1, lossMax: 0.5 },
        { domain: 'instagram.com', name: 'instagram.com', service: '社交媒体', location: '区域 CDN (亚太节点)', delayMin: 65, delayMax: 130, lossMin: 0.1, lossMax: 0.6 },
        { domain: 'wikipedia.org', name: 'wikipedia.org', service: '在线百科', location: '全球 CDN', delayMin: 80, delayMax: 150, lossMin: 0.1, lossMax: 0.3 },
        { domain: 'reddit.com', name: 'reddit.com', service: '论坛/社交', location: '美国/全球 CDN', delayMin: 120, delayMax: 200, lossMin: 0.3, lossMax: 1.0 },
        { domain: 'https://www.google.com/search?q=chatgpt.com**', name: 'chatgpt.com**', service: 'AI 聊天机器人', location: '美国/全球 CDN', delayMin: 120, delayMax: 220, lossMin: 0.5, lossMax: 1.5 },
        { domain: 'x.com', name: 'x.com (Twitter)', service: '社交媒体', location: '全球 CDN', delayMin: 90, delayMax: 160, lossMin: 0.2, lossMax: 0.8 },
        { domain: 'amazon.com', name: 'amazon.com', service: '电子商务', location: '美国/全球 CDN', delayMin: 130, delayMax: 200, lossMin: 0.5, lossMax: 1.0 },
        { domain: 'whatsapp.com', name: 'whatsapp.com', service: '即时通讯', location: '区域 CDN', delayMin: 70, delayMax: 140, lossMin: 0.1, lossMax: 0.5 },
        { domain: 'yahoo.com', name: 'yahoo.com', service: '门户/新闻', location: '区域 CDN (亚太节点)', delayMin: 50, delayMax: 100, lossMin: 0.0, lossMax: 0.2 },
        { domain: 'tiktok.com', name: 'tiktok.com', service: '短视频', location: '全球 CDN', delayMin: 40, delayMax: 80, lossMin: 0.0, lossMax: 0.3 },
        { domain: 'bing.com', name: 'bing.com', service: '搜索引擎', location: '区域 CDN', delayMin: 40, delayMax: 90, lossMin: 0.0, lossMax: 0.2 },
        { domain: 'linkedin.com', name: 'linkedin.com', service: '职业社交', location: '美国/全球 CDN', delayMin: 100, delayMax: 180, lossMin: 0.2, lossMax: 0.7 },
        { domain: 'yandex.ru', name: 'yandex.ru', service: '俄罗斯搜索', location: '欧洲/俄罗斯', delayMin: 200, delayMax: 350, lossMin: 1.0, lossMax: 3.0 },
        { domain: 'netflix.com', name: 'netflix.com', service: '视频流媒体', location: '全球 CDN (Open Connect)', delayMin: 50, delayMax: 100, lossMin: 0.0, lossMax: 0.2 },
        { domain: 'twitch.tv', name: 'twitch.tv', service: '游戏直播', location: '全球 CDN', delayMin: 90, delayMax: 160, lossMin: 0.3, lossMax: 1.0 },
        { domain: 'https://www.google.com/search?q=microsoftonline.com**', name: 'microsoftonline.com**', service: '云服务/办公', location: '区域数据中心', delayMin: 40, delayMax: 90, lossMin: 0.1, lossMax: 0.4 },
        { domain: 'apple.com', name: 'apple.com', service: '科技/零售', location: '全球 CDN', delayMin: 60, delayMax: 120, lossMin: 0.1, lossMax: 0.4 },
        { domain: 'ebay.com', name: 'ebay.com', service: '电子商务', location: '美国/全球 CDN', delayMin: 130, delayMax: 210, lossMin: 0.5, lossMax: 1.2 }
      ],
      
      // 仿真进程管理系统
      processSystem: {
        // 模拟的Linux系统进程列表（基于systemd）
        processes: [
          {
            pid: 1,
            command: '/sbin/init',
            user: 'root',
            description: '初始化系统，所有进程的父进程（PPID=0）。负责启动、监控和管理所有系统服务。',
            cpu: 0.1,
            memory: 2.5,
            state: 'S'
          },
          {
            pid: 2,
            command: '[kthreadd]',
            user: 'root',
            description: '内核线程管理器。负责管理和调度所有其他内核线程。',
            cpu: 0.0,
            memory: 0.0,
            state: 'S'
          },
          {
            pid: 3,
            command: '[rcu_gp]',
            user: 'root',
            description: '内核工作进程（RCU垃圾回收）。',
            cpu: 0.0,
            memory: 0.0,
            state: 'S'
          },
          {
            pid: 4,
            command: '[ksoftirqd/0]',
            user: 'root',
            description: '内核软件中断处理进程。',
            cpu: 0.0,
            memory: 0.0,
            state: 'S'
          },
          {
            pid: 100,
            command: '[kworker/.../mm_percpu_wq]',
            user: 'root',
            description: '通用内核工作者。执行各种内核任务。',
            cpu: 0.0,
            memory: 0.0,
            state: 'S'
          },
          {
            pid: 500,
            command: 'systemd-journald',
            user: 'root',
            description: '系统日志服务。收集和存储所有系统和应用程序的日志。',
            cpu: 0.2,
            memory: 8.5,
            state: 'S'
          },
          {
            pid: 550,
            command: 'systemd-udevd',
            user: 'root',
            description: '设备管理器。管理内核设备事件，创建设备节点。',
            cpu: 0.1,
            memory: 3.2,
            state: 'S'
          },
          {
            pid: 600,
            command: '/usr/lib/systemd/systemd-logind',
            user: 'root',
            description: '用户登录管理器。管理用户会话、电源按钮和ACPI事件。',
            cpu: 0.1,
            memory: 2.8,
            state: 'S'
          },
          {
            pid: 650,
            command: 'dbus-daemon',
            user: 'messagebus',
            description: '消息总线系统。允许进程间通信（IPC）。',
            cpu: 0.0,
            memory: 1.5,
            state: 'S'
          },
          {
            pid: 700,
            command: 'sshd',
            user: 'root',
            description: '安全Shell守护进程。允许远程安全连接。',
            cpu: 0.1,
            memory: 4.2,
            state: 'S'
          },
          {
            pid: 750,
            command: 'cron',
            user: 'root',
            description: '定时任务守护进程。执行计划好的任务。',
            cpu: 0.0,
            memory: 1.8,
            state: 'S'
          },
          {
            pid: 800,
            command: 'rsyslogd',
            user: 'syslog',
            description: '传统日志服务。',
            cpu: 0.1,
            memory: 5.6,
            state: 'S'
          },
          {
            pid: 1000,
            command: '/usr/bin/bash',
            user: 'user',
            description: '终端Shell（用户已登录）。',
            cpu: 0.0,
            memory: 2.1,
            state: 'S'
          },
          {
            pid: 1001,
            command: 'ps',
            user: 'user',
            description: '显示当前进程状态。',
            cpu: 0.0,
            memory: 0.5,
            state: 'R'
          }
        ],
        // 系统状态信息
        systemInfo: {
          uptime: '1 day, 2:30',
          users: 1,
          loadAverage: [0.05, 0.03, 0.01],
          totalTasks: 15,
          runningTasks: 1,
          sleepingTasks: 14,
          stoppedTasks: 0,
          zombieTasks: 0,
          cpuUsage: {
            user: 0.1,
            system: 0.0,
            nice: 0.0,
            idle: 99.9,
            wait: 0.0,
            hi: 0.0,
            si: 0.0,
            st: 0.0
          },
          memory: {
            total: 8192,
            free: 8180,
            used: 12,
            buffCache: 0
          }
        }
      },
      
      // 虚拟文件系统（8GB磁盘容量）
      fileSystem: {
        currentPath: '/home/user',
        diskCapacity: 8 * 1024 * 1024 * 1024, // 8GB in bytes
        usedSpace: 0,
        root: {
          name: '/',
          type: 'directory',
          permissions: 'drwxr-xr-x',
          owner: 'root',
          group: 'root',
          children: {
            home: {
              name: 'home',
              type: 'directory',
              permissions: 'drwxr-xr-x',
              owner: 'root',
              group: 'root',
              children: {
                user: {
                  name: 'user',
                  type: 'directory',
                  permissions: 'drwxr-xr-x',
                  owner: 'user',
                  group: 'user',
                  children: {
                    Documents: {
                      name: 'Documents',
                      type: 'directory',
                      permissions: 'drwxr-xr-x',
                      owner: 'user',
                      group: 'user',
                      children: {}
                    },
                    Downloads: {
                      name: 'Downloads',
                      type: 'directory',
                      permissions: 'drwxr-xr-x',
                      owner: 'user',
                      group: 'user',
                      children: {}
                    },
                    Music: {
                      name: 'Music',
                      type: 'directory',
                      permissions: 'drwxr-xr-x',
                      owner: 'user',
                      group: 'user',
                      children: {}
                    },
                    Pictures: {
                      name: 'Pictures',
                      type: 'directory',
                      permissions: 'drwxr-xr-x',
                      owner: 'user',
                      group: 'user',
                      children: {}
                    }
                  }
                }
              }
            },
            etc: {
              name: 'etc',
              type: 'directory',
              permissions: 'drwxr-xr-x',
              owner: 'root',
              group: 'root',
              children: {
                'hosts': {
                  name: 'hosts',
                  type: 'file',
                  permissions: '-rw-r--r--',
                  owner: 'root',
                  group: 'root',
                  content: '127.0.0.1\tlocalhost\n::1\t\tlocalhost',
                  size: 32
                },
                'passwd': {
                  name: 'passwd',
                  type: 'file',
                  permissions: '-rw-r--r--',
                  owner: 'root',
                  group: 'root',
                  content: 'root:x:0:0:root:/root:/bin/bash\nuser:x:1000:1000:User:/home/user:/bin/bash',
                  size: 75
                },
                'group': {
                  name: 'group',
                  type: 'file',
                  permissions: '-rw-r--r--',
                  owner: 'root',
                  group: 'root',
                  content: 'root:x:0:\nuser:x:1000:',
                  size: 25
                },
                'fstab': {
                  name: 'fstab',
                  type: 'file',
                  permissions: '-rw-r--r--',
                  owner: 'root',
                  group: 'root',
                  content: '# /etc/fstab: static file system information.\n/dev/sda1 / ext4 defaults 0 1',
                  size: 65
                }
              }
            },
            var: {
              name: 'var',
              type: 'directory',
              permissions: 'drwxr-xr-x',
              owner: 'root',
              group: 'root',
              children: {
                'log': {
                  name: 'log',
                  type: 'directory',
                  permissions: 'drwxr-xr-x',
                  owner: 'root',
                  group: 'root',
                  children: {
                    'syslog': {
                      name: 'syslog',
                      type: 'file',
                      permissions: '-rw-r--r--',
                      owner: 'root',
                      group: 'root',
                      content: '2024-01-01 10:00:00 System booted\n2024-01-01 10:01:00 User logged in',
                      size: 60
                    }
                  }
                },
                'lib': {
                  name: 'lib',
                  type: 'directory',
                  permissions: 'drwxr-xr-x',
                  owner: 'root',
                  group: 'root',
                  children: {}
                }
              }
            },
            tmp: {
              name: 'tmp',
              type: 'directory',
              permissions: 'drwxrwxrwt',
              owner: 'root',
              group: 'root',
              children: {
                'temp_file.txt': {
                  name: 'temp_file.txt',
                  type: 'file',
                  permissions: '-rw-rw-rw-',
                  owner: 'root',
                  group: 'root',
                  content: 'This is a temporary file',
                  size: 24
                }
              }
            },
            usr: {
              name: 'usr',
              type: 'directory',
              permissions: 'drwxr-xr-x',
              owner: 'root',
              group: 'root',
              children: {
                'bin': {
                  name: 'bin',
                  type: 'directory',
                  permissions: 'drwxr-xr-x',
                  owner: 'root',
                  group: 'root',
                  children: {}
                },
                'lib': {
                  name: 'lib',
                  type: 'directory',
                  permissions: 'drwxr-xr-x',
                  owner: 'root',
                  group: 'root',
                  children: {}
                }
              }
            },
            boot: {
              name: 'boot',
              type: 'directory',
              permissions: 'drwxr-xr-x',
              owner: 'root',
              group: 'root',
              children: {
                'grub': {
                  name: 'grub',
                  type: 'directory',
                  permissions: 'drwxr-xr-x',
                  owner: 'root',
                  group: 'root',
                  children: {
                    'grub.cfg': {
                      name: 'grub.cfg',
                      type: 'file',
                      permissions: '-rw-r--r--',
                      owner: 'root',
                      group: 'root',
                      content: 'set default=0\ntimeout=5',
                      size: 25
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  
  mounted() {
    // 组件挂载后初始化磁盘使用统计
    this.updateDiskUsage();
    // 组件挂载后自动聚焦输入框
    this.focusInput();
    
    // 添加点击窗口内容区域时重新聚焦输入框
    this.$nextTick(() => {
      const windowContent = this.$el.querySelector('.window-content');
      if (windowContent) {
        windowContent.addEventListener('click', this.focusInput);
      }
    });
  },
  
  beforeDestroy() {
    // 移除事件监听器
    const windowContent = this.$el.querySelector('.window-content');
    if (windowContent) {
      windowContent.removeEventListener('click', this.focusInput);
    }
    // 组件销毁前的其他逻辑
  },
  methods: {
    handleClose() {
      this.$emit('close')
    },
    toggleMaximize() {
      this.isMaximized = !this.isMaximized
    },
    
    // 聚焦输入框方法
    focusInput() {
      this.$nextTick(() => {
        const input = this.$refs.terminalInput;
        if (input) {
          input.focus();
        }
      });
    },
    
    // 获取当前提示符（固定为>）
    getCurrentPrompt() {
      if (this.isWaitingForPassword) {
        return this.passwordPrompt;
      }
      
      return '>';
    },
    

    
    // 文件系统操作方法
    getCurrentDirectory() {
      const pathParts = this.fileSystem.currentPath.split('/').filter(p => p);
      let current = this.fileSystem.root;
      
      for (const part of pathParts) {
        if (current.children && current.children[part]) {
          current = current.children[part];
        } else {
          return null;
        }
      }
      return current;
    },
    
    resolvePath(path) {
      if (path.startsWith('/')) {
        // 绝对路径
        return path;
      } else {
        // 相对路径
        const currentPath = this.fileSystem.currentPath;
        if (path === '..') {
          const parts = currentPath.split('/').filter(p => p);
          parts.pop();
          return '/' + parts.join('/');
        } else if (path === '.') {
          return currentPath;
        } else {
          return currentPath === '/' ? '/' + path : currentPath + '/' + path;
        }
      }
    },
    
    getDirectoryByPath(path) {
      const pathParts = path.split('/').filter(p => p);
      let current = this.fileSystem.root;
      
      for (const part of pathParts) {
        if (current.children && current.children[part]) {
          current = current.children[part];
        } else {
          return null;
        }
      }
      return current;
    },
    
    createFile(path, content = '') {
      const resolvedPath = this.resolvePath(path);
      const pathParts = resolvedPath.split('/').filter(p => p);
      const fileName = pathParts.pop();
      const dirPath = '/' + pathParts.join('/');
      
      const parentDir = this.getDirectoryByPath(dirPath);
      if (!parentDir || parentDir.type !== 'directory') {
        return false;
      }
      
      // 检查磁盘空间是否足够
      const fileSize = content.length;
      if (!this.hasEnoughSpace(fileSize)) {
        return false;
      }
      
      parentDir.children[fileName] = {
        name: fileName,
        type: 'file',
        permissions: '-rw-r--r--',
        owner: 'user',
        group: 'user',
        content: content,
        size: fileSize
      };
      
      // 更新磁盘使用统计
      this.updateDiskUsage();
      
      return true;
    },
    
    createDirectory(path) {
      const resolvedPath = this.resolvePath(path);
      const pathParts = resolvedPath.split('/').filter(p => p);
      const dirName = pathParts.pop();
      const parentPath = '/' + pathParts.join('/');
      
      const parentDir = this.getDirectoryByPath(parentPath);
      if (!parentDir || parentDir.type !== 'directory') {
        return false;
      }
      
      parentDir.children[dirName] = {
        name: dirName,
        type: 'directory',
        permissions: 'drwxr-xr-x',
        owner: 'user',
        group: 'user',
        children: {}
      };
      
      return true;
    },
    
    removeFile(path) {
      const resolvedPath = this.resolvePath(path);
      const pathParts = resolvedPath.split('/').filter(p => p);
      const fileName = pathParts.pop();
      const dirPath = '/' + pathParts.join('/');
      
      const parentDir = this.getDirectoryByPath(dirPath);
      if (!parentDir || !parentDir.children[fileName]) {
        return false;
      }
      
      delete parentDir.children[fileName];
      
      // 更新磁盘使用统计
      this.updateDiskUsage();
      
      return true;
    },
    
    removeDirectory(path) {
      const resolvedPath = this.resolvePath(path);
      const pathParts = resolvedPath.split('/').filter(p => p);
      const dirName = pathParts.pop();
      const parentPath = '/' + pathParts.join('/');
      
      const parentDir = this.getDirectoryByPath(parentPath);
      if (!parentDir || !parentDir.children[dirName]) {
        return false;
      }
      
      const targetDir = parentDir.children[dirName];
      if (targetDir.type !== 'directory' || Object.keys(targetDir.children).length > 0) {
        return false;
      }
      
      delete parentDir.children[dirName];
      return true;
    },
    
    getFileContent(path) {
      const resolvedPath = this.resolvePath(path);
      const file = this.getDirectoryByPath(resolvedPath);
      if (!file || file.type !== 'file') {
        return null;
      }
      return file.content;
    },
    
    listDirectory(path = null) {
      const dir = path ? this.getDirectoryByPath(this.resolvePath(path)) : this.getCurrentDirectory();
      if (!dir || dir.type !== 'directory') {
        return null;
      }
      
      return Object.values(dir.children).map(item => ({
        name: item.name,
        type: item.type,
        permissions: item.permissions,
        owner: item.owner,
        size: item.size || 0
      }));
    },
    
    // 磁盘占用计算方法
    calculateDirectorySize(directory) {
      let totalSize = 0;
      
      if (directory.type === 'file') {
        return directory.size || 0;
      }
      
      if (directory.children) {
        for (const childName in directory.children) {
          totalSize += this.calculateDirectorySize(directory.children[childName]);
        }
      }
      
      return totalSize;
    },
    
    // 计算整个文件系统的使用空间
    calculateUsedSpace() {
      return this.calculateDirectorySize(this.fileSystem.root);
    },
    
    // 更新磁盘使用统计
    updateDiskUsage() {
      this.fileSystem.usedSpace = this.calculateUsedSpace();
    },
    
    // 检查磁盘空间是否足够
    hasEnoughSpace(requiredSize) {
      this.updateDiskUsage();
      return this.fileSystem.usedSpace + requiredSize <= this.fileSystem.diskCapacity;
    },
    
    // 获取磁盘使用统计信息
    getDiskUsageInfo() {
      this.updateDiskUsage();
      const usedGB = (this.fileSystem.usedSpace / (1024 * 1024 * 1024)).toFixed(2);
      const totalGB = (this.fileSystem.diskCapacity / (1024 * 1024 * 1024)).toFixed(2);
      const freeGB = ((this.fileSystem.diskCapacity - this.fileSystem.usedSpace) / (1024 * 1024 * 1024)).toFixed(2);
      const usagePercent = ((this.fileSystem.usedSpace / this.fileSystem.diskCapacity) * 100).toFixed(1);
      
      return {
        used: this.fileSystem.usedSpace,
        total: this.fileSystem.diskCapacity,
        free: this.fileSystem.diskCapacity - this.fileSystem.usedSpace,
        usedGB: usedGB,
        totalGB: totalGB,
        freeGB: freeGB,
        usagePercent: usagePercent
      };
    },
    
    handleEnter() {
      // 处理回车事件
      this.isFlashing = true
      
      // 保存并显示用户输入的命令
      if (this.inputText.trim()) {
        // 重置历史索引
        this.historyIndex = -1;
        
        // 检查是否正在等待密码输入
        if (this.isWaitingForPassword) {
          this.handlePasswordInput(this.inputText);
          this.inputText = '';
          this.isFlashing = false;
          return;
        }
        
        // 提取命令和参数
        const parts = this.inputText.trim().split(' ');
        const command = parts[0].toLowerCase();
        const args = parts.slice(1);
        
        // 特殊命令处理
        if (command === 'exit') {
          // 不添加到历史记录，直接关闭终端
          this.handleClose();
          return;
        }
        
        if (command === 'clear') {
          this.commandHistory = [];
          this.inputText = '';
          this.isFlashing = false;
          return;
        }
        
        // 设置当前命令和开始输出
        this.currentCommand = this.inputText;
        this.startTypingOutput(command, args);
        
        // 清除输入
        this.inputText = '';
      }
      
      // 200ms后取消闪烁效果
      setTimeout(() => {
        this.isFlashing = false
      }, 200)
    },
    
    // 处理密码输入
    handlePasswordInput(password) {
      // 密码验证逻辑
      if (password.trim() !== '') {
        let response = '';
        let isError = false;
        
        if (this.pendingCommand.command === 'sudo') {
          // 验证sudo密码
          if (password === this.defaultPassword) {
            response = `Command executed with sudo privileges.`;
          } else {
            response = `sudo: authentication error`;
            isError = true;
          }
        } else if (this.pendingCommand.command === 'su') {
          // su命令暂时接受任何非空密码
          response = `Switched to user mode.`;
        }
        
        // 重置密码输入状态
        this.isWaitingForPassword = false;
        this.passwordPrompt = '';
        
        // 保存密码输入到历史记录
        this.commandHistory.push({
          command: '[password hidden]',
          response: '',
          isError: false,
          prompt: this.passwordPrompt
        });
        
        // 设置当前命令为原始命令
        this.currentCommand = `${this.pendingCommand.command} ${this.pendingCommand.args.join(' ')}`;
        this.pendingCommand = null;
        
        // 显示命令结果
        this.isTyping = true;
        this.currentOutput = '';
        this.outputIndex = 0;
        this.typeOutput(response, isError);
      } else {
        // 密码为空，重新提示
        this.isWaitingForPassword = true;
      }
    },
    
    // 设置默认密码
    setDefaultPassword(newPassword) {
      this.defaultPassword = newPassword;
    },
    
    // 获取默认密码提示（用于帮助命令）
    getDefaultPasswordHint() {
      return `Default sudo password is set to: ${this.defaultPassword}`;
    },
    
    // 开始打字输出效果
    startTypingOutput(command, args) {
      let response = '';
      let isError = false;
      
      // Linux基本命令处理逻辑
      switch (command) {
        case 'help':
          response = 'Available commands:\n\nFile and Directory Operations\n- ls: List directory contents\n- cd [dir]: Change directory\n- pwd: Print working directory\n- mkdir [dir]: Create new directory\n- rmdir [dir]: Remove empty directory\n- touch [file]: Create empty file\n- cp [src] [dest]: Copy files/directories\n- mv [src] [dest]: Move/rename files/directories\n- rm [file]: Remove files/directories\n\nFile Content Viewing\n- cat [file]: Display file contents\n- less [file]: View file with pagination\n- head [file]: Display first lines of file\n- tail [file]: Display last lines of file\n- grep [pattern] [file]: Search text in file\n\nPermissions and Users\n- sudo [cmd]: Run command as superuser\n- su [user]: Switch user\n- chmod [perms] [file]: Change file permissions\n- chown [user:group] [file]: Change file owner\n- whoami: Show current user\n\nSystem and Processes\n- ps: Show running processes\n- top: Monitor system processes\n- kill [pid]: Terminate process\n- df: Show disk usage\n- du [dir]: Show directory size\n- uname: Show system information\n- date: Show current date and time\n\nNetwork\n- ping [host]: Test network connectivity\n- ip addr: Show network interfaces\n\nUtilities\n- clear: Clear terminal\n- echo [text]: Print text\n- history: Show command history\n- man [cmd]: Show manual page\n- poweroff: Shut down the system\n- exit: Close terminal';
          break;
        case 'ls':
          const listResult = this.listDirectory(args[0]);
          if (listResult === null) {
            response = `ls: cannot access '${args[0] || '.'}': No such file or directory`;
            isError = true;
          } else {
            response = listResult.map(item => item.name).join('\n');
          }
          break;
        case 'pwd':
          response = this.fileSystem.currentPath;
          break;
        case 'echo':
          response = args.join(' ');
          if (response === '') {
            response = '';
          }
          break;
        case 'whoami':
          response = 'user';
          break;
        case 'date':
          response = new Date().toString();
          break;
        case 'cal':
          // 简单的日历展示
          const now = new Date();
          const month = now.getMonth() + 1;
          const year = now.getFullYear();
          response = `   ${new Intl.DateTimeFormat('en-US', { month: 'long' }).format(now)} ${year}\nSu Mo Tu We Th Fr Sa\n 1  2  3  4  5  6  7\n 8  9 10 11 12 13 14\n15 16 17 18 19 20 21\n22 23 24 25 26 27 28\n29 30 31`;
          break;
        case 'history':
          if (this.commandHistory.length === 0) {
            response = 'No command history available';
          } else {
            response = this.commandHistory.map((entry, idx) => `${idx + 1}  ${entry.command}`).join('\n');
          }
          break;
        case 'cd':
          if (args.length === 0 || args[0] === '~') {
            this.fileSystem.currentPath = '/home/user';
            response = '';
          } else {
            const targetPath = this.resolvePath(args[0]);
            const targetDir = this.getDirectoryByPath(targetPath);
            if (targetDir && targetDir.type === 'directory') {
              this.fileSystem.currentPath = targetPath;
              response = '';
            } else {
              response = `cd: ${args[0]}: No such file or directory`;
              isError = true;
            }
          }
          break;
        case 'mkdir':
          if (args.length === 0) {
            response = 'Usage: mkdir [directory_name]';
            isError = true;
          } else {
            if (this.createDirectory(args[0])) {
              response = `Directory '${args[0]}' created`;
            } else {
              response = `mkdir: cannot create directory '${args[0]}': File exists or invalid path`;
              isError = true;
            }
          }
          break;
        case 'rmdir':
          if (args.length === 0) {
            response = 'Usage: rmdir [directory_name]';
            isError = true;
          } else {
            if (this.removeDirectory(args[0])) {
              response = `Directory '${args[0]}' removed`;
            } else {
              response = `rmdir: failed to remove '${args[0]}': Directory not empty or does not exist`;
              isError = true;
            }
          }
          break;
        case 'touch':
          if (args.length === 0) {
            response = 'Usage: touch [file_name]';
            isError = true;
          } else {
            if (this.createFile(args[0])) {
              response = `File '${args[0]}' created`;
            } else {
              // 检查是否是磁盘空间不足
              const diskInfo = this.getDiskUsageInfo();
              if (diskInfo.free <= 0) {
                response = `touch: cannot create file '${args[0]}': No space left on device`;
              } else {
                response = `touch: cannot create file '${args[0]}': Invalid path`;
              }
              isError = true;
            }
          }
          break;
        case 'cp':
          if (args.length < 2) {
            response = 'Usage: cp [source] [destination]';
            isError = true;
          } else {
            response = `Copied '${args[0]}' to '${args[1]}'`;
          }
          break;
        case 'mv':
          if (args.length < 2) {
            response = 'Usage: mv [source] [destination]';
            isError = true;
          } else {
            response = `Moved '${args[0]}' to '${args[1]}'`;
          }
          break;
        case 'rm':
          if (args.length === 0) {
            response = 'Usage: rm [file_name]';
            isError = true;
          } else {
            const target = args[0];
            const isRecursive = args.includes('-rf') || args.includes('--recursive');
            
            if (this.removeFile(target)) {
              response = `File '${target}' removed`;
            } else {
              response = `rm: cannot remove '${target}': No such file or directory`;
              isError = true;
            }
          }
          break;
        case 'cat':
          if (args.length === 0) {
            response = 'Usage: cat [file_name]';
            isError = true;
          } else {
            const content = this.getFileContent(args[0]);
            if (content !== null) {
              response = content;
            } else {
              response = `cat: ${args[0]}: No such file or directory`;
              isError = true;
            }
          }
          break;
        case 'less':
          if (args.length === 0) {
            response = `Usage: ${command} [file_name]`;
            isError = true;
          } else {
            const content = this.getFileContent(args[0]);
            if (content !== null) {
              response = `${content}\n\n(END) - Press 'q' to quit`;
            } else {
              response = `${command}: ${args[0]}: No such file or directory`;
              isError = true;
            }
          }
          break;
        case 'head':
          if (args.length === 0) {
            response = `Usage: ${command} [file_name]`;
            isError = true;
          } else {
            const content = this.getFileContent(args[0]);
            if (content !== null) {
              const lines = content.split('\n');
              response = lines.slice(0, 10).join('\n');
            } else {
              response = `${command}: ${args[0]}: No such file or directory`;
              isError = true;
            }
          }
          break;
        case 'tail':
          if (args.length === 0) {
            response = `Usage: ${command} [file_name]`;
            isError = true;
          } else {
            const content = this.getFileContent(args[0]);
            if (content !== null) {
              const lines = content.split('\n');
              response = lines.slice(-10).join('\n');
            } else {
              response = `${command}: ${args[0]}: No such file or directory`;
              isError = true;
            }
          }
          break;
        case 'grep':
          if (args.length < 2) {
            response = 'Usage: grep [pattern] [file_name]';
            isError = true;
          } else {
            const content = this.getFileContent(args[1]);
            if (content !== null) {
              const lines = content.split('\n');
              const matches = lines.filter(line => line.includes(args[0]));
              if (matches.length > 0) {
                response = matches.join('\n');
              } else {
                response = `No matches found for '${args[0]}' in ${args[1]}`;
              }
            } else {
              response = `grep: ${args[1]}: No such file or directory`;
              isError = true;
            }
          }
          break;
        case 'sudo':
          // 交互：密码输入
          this.isWaitingForPassword = true;
          this.passwordPrompt = '[sudo] password for user: ';
          this.pendingCommand = { command: 'sudo', args: args };
          return; // 不立即输出，等待密码输入
        case 'su':
          // 交互：密码输入
          this.isWaitingForPassword = true;
          this.passwordPrompt = 'Password: ';
          this.pendingCommand = { command: 'su', args: args };
          return; // 不立即输出，等待密码输入

        case 'ps':
          // 更新系统状态
          this.updateSystemStatus();
          
          // 生成ps命令输出
          let psOutput = '  PID TTY          TIME CMD\n';
          this.processSystem.processes.forEach(process => {
            // 模拟TTY和TIME信息
            const tty = process.pid <= 100 ? '?' : 'pts/0';
            const time = '00:00:00';
            psOutput += `  ${process.pid} ${tty}    ${time} ${process.command}\n`;
          });
          response = psOutput;
          break;
        case 'top':
          // 更新系统状态
          this.updateSystemStatus();
          
          // 生成top命令输出
          const sys = this.processSystem.systemInfo;
          response = `top - 00:00:00 up ${sys.uptime},  ${sys.users} user,  load average: ${sys.loadAverage[0].toFixed(2)}, ${sys.loadAverage[1].toFixed(2)}, ${sys.loadAverage[2].toFixed(2)}\n` +
                    `Tasks: ${sys.totalTasks} total,   ${sys.runningTasks} running,  ${sys.sleepingTasks} sleeping,  ${sys.stoppedTasks} stopped,  ${sys.zombieTasks} zombie\n` +
                    `%Cpu(s): ${sys.cpuUsage.user.toFixed(1)} us,  ${sys.cpuUsage.system.toFixed(1)} sy,  ${sys.cpuUsage.nice.toFixed(1)} ni,${sys.cpuUsage.idle.toFixed(1)} id,  ${sys.cpuUsage.wait.toFixed(1)} wa,  ${sys.cpuUsage.hi.toFixed(1)} hi,  ${sys.cpuUsage.si.toFixed(1)} si,  ${sys.cpuUsage.st.toFixed(1)} st\n` +
                    `KiB Mem :  ${sys.memory.total} total,   ${sys.memory.free} free,    ${sys.memory.used} used,    ${sys.memory.buffCache} buff/cache\n\n` +
                    `  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND\n`;
          
          // 添加进程列表（按CPU使用率排序）
          this.getProcessesByCPU().forEach(process => {
            const virt = Math.round(process.memory * 1024); // 转换为KB
            const res = Math.round(process.memory * 1024);
            const shr = Math.round(process.memory * 512);
            const cpuPercent = process.cpu.toFixed(1);
            const memPercent = ((process.memory / sys.memory.total) * 100).toFixed(1);
            const timePlus = '00:00.00';
            
            response += `${process.pid.toString().padStart(5)} ${process.user.padEnd(8)} 20   0  ${virt.toString().padStart(6)} ${res.toString().padStart(6)} ${shr.toString().padStart(6)} ${process.state}  ${cpuPercent.padStart(4)} ${memPercent.padStart(4)} ${timePlus} ${process.command}\n`;
          });
          
          response += '\nPress q to quit, h for help';
          break;
        case 'kill':
          if (args.length === 0) {
            response = 'Usage: kill [process_id]';
            isError = true;
          } else {
            const pid = parseInt(args[0]);
            const process = this.processSystem.processes.find(p => p.pid === pid);
            
            if (!process) {
              response = `kill: (${pid}) - No such process`;
              isError = true;
            } else if (process.pid === 1) {
              response = `kill: (${pid}) - Operation not permitted (cannot kill init process)`;
              isError = true;
            } else if (process.pid <= 100) {
              response = `kill: (${pid}) - Operation not permitted (cannot kill kernel processes)`;
              isError = true;
            } else {
              // 模拟杀死进程（实际上只是从列表中移除）
              this.processSystem.processes = this.processSystem.processes.filter(p => p.pid !== pid);
              // 更新系统状态
              this.updateSystemStatus();
              response = `Sent signal to process ${pid}`;
            }
          }
          break;
        case 'df':
          // 获取磁盘使用信息
          const diskInfo = this.getDiskUsageInfo();
          response = `Filesystem     1K-blocks    Used Available Use% Mounted on\n/dev/sda1       ${Math.floor(diskInfo.total / 1024).toString().padStart(8)} ${Math.floor(diskInfo.used / 1024).toString().padStart(8)} ${Math.floor(diskInfo.free / 1024).toString().padStart(8)} ${diskInfo.usagePercent}% /`;
          break;
        case 'du':
          let targetPath = args[0] || '.';
          const resolvedPath = this.resolvePath(targetPath);
          const targetDir = this.getDirectoryByPath(resolvedPath);
          
          if (!targetDir) {
            response = `du: cannot access '${targetPath}': No such file or directory`;
            isError = true;
          } else {
            // 递归计算目录大小
            const calculateDirSizes = (dir, path = '') => {
              const sizes = [];
              
              // 计算当前目录大小
              const currentSize = this.calculateDirectorySize(dir);
              sizes.push({ path: path || '.', size: currentSize });
              
              // 递归计算子目录
              if (dir.children) {
                for (const childName in dir.children) {
                  const child = dir.children[childName];
                  if (child.type === 'directory') {
                    const childSizes = calculateDirSizes(child, path ? `${path}/${childName}` : childName);
                    sizes.push(...childSizes);
                  }
                }
              }
              
              return sizes;
            };
            
            const sizes = calculateDirSizes(targetDir, targetPath === '.' ? '' : targetPath);
            
            // 生成输出
            response = sizes.map(item => {
              const sizeKB = Math.ceil(item.size / 1024); // 转换为KB并向上取整
              return `${sizeKB}K\t${item.path}`;
            }).join('\n');
          }
          break;
        case 'uname':
          response = `Linux personal-web-site 5.15.0-100-generic #110-Ubuntu SMP Tue Feb 20 12:03:28 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux`;
          break;
        case 'ping':
          if (args.length === 0) {
            response = 'Usage: ping [hostname/IP]';
            isError = true;
          } else {
            const target = args[0];
            // 检查是否为固定可连接网站
            const website = this.pingWebsites.find(w => 
              w.domain.toLowerCase() === target.toLowerCase() || 
              w.name.toLowerCase().includes(target.toLowerCase())
            );
            
            if (website) {
              // 生成随机延迟和丢包率（在指定范围内）
              const delay1 = (Math.random() * (website.delayMax - website.delayMin) + website.delayMin).toFixed(1);
              const delay2 = (Math.random() * (website.delayMax - website.delayMin) + website.delayMin).toFixed(1);
              const avgDelay = ((parseFloat(delay1) + parseFloat(delay2)) / 2).toFixed(1);
              const lossRate = (Math.random() * (website.lossMax - website.lossMin) + website.lossMin).toFixed(1);
              
              response = `PING ${website.name} (${website.domain}) 56(84) bytes of data.\n64 bytes from ${website.domain}: icmp_seq=1 ttl=64 time=${delay1} ms\n64 bytes from ${website.domain}: icmp_seq=2 ttl=64 time=${delay2} ms\n\n--- ${website.name} ping statistics ---\n2 packets transmitted, 2 received, ${lossRate}% packet loss, time 100ms\nrtt min/avg/max/mdev = ${Math.min(delay1, delay2)}/${avgDelay}/${Math.max(delay1, delay2)}/0.005 ms`;
            } else {
              // 其他网站返回100%丢包
              response = `PING ${target} (127.0.0.1) 56(84) bytes of data.\n\n--- ${target} ping statistics ---\n2 packets transmitted, 0 received, 100% packet loss, time 100ms`;
            }
          }
          break;
        case 'ip':
          if (args[0] === 'addr' || args[0] === 'address') {
            response = `1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000\n    inet 127.0.0.1/8 scope host lo\n       valid_lft forever preferred_lft forever\n2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000\n    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0\n       valid_lft 86399sec preferred_lft 86399sec`;
          } else {
            response = 'Usage: ip addr (to show network interfaces)';
            isError = true;
          }
          break;

        case 'reboot':
          // 刷新浏览器
          setTimeout(() => {
            window.location.reload();
          }, 1000);
          response = 'System reboot initiated...';
          break;
        case 'poweroff':
          // 关闭浏览器窗口
          setTimeout(() => {
            window.close();
          }, 1000);
          response = 'System poweroff initiated...';
          break;
        default:
          response = `Command not found: ${this.currentCommand}`;
          isError = true;
      }
      
      // 开始打字效果
      this.isTyping = true;
      this.currentOutput = '';
      this.outputIndex = 0;
      this.typingSpeed = 1; // 再次翻倍速度（原7.5ms改为1ms）
      
      // 模拟处理时间（根据命令复杂度）
      const processingTime = this.getProcessingTime(command);
      
      setTimeout(() => {
        this.typeOutput(response, isError);
      }, processingTime);
    },
    
    // 获取命令处理时间（模拟真实终端的响应时间，基于命令复杂度和输出长度）
    getProcessingTime(command) {
      const processingTimes = {
        // 简单命令（快速响应）
        'pwd': 30,
        'whoami': 30,
        'echo': 40,
        'cd': 50,
        'kill': 60,
        'touch': 70,
        'mkdir': 80,
        'rmdir': 80,
        'rm': 90,
        'uname': 100,
        
        // 中等复杂度命令
        'ls': 120,
        'date': 120,
        'cal': 150,
        'history': 130,
        'cp': 140,
        'mv': 140,

        'ps': 180,
        'df': 200,
        'du': 220,
        'ip': 180,
        
        // 复杂命令（需要更多处理时间）
        'cat': 250,
        'less': 280,
        'head': 280,
        'tail': 280,
        'grep': 300,
        'sudo': 350,
        'su': 350,
        'top': 400,
        'ping': 450,
        
        // 特殊命令
        'help': 600,  // 输出内容非常长
        'reboot': 100,
        'poweroff': 100
      };
      
      return processingTimes[command] || 150;
    },
    
    // 获取格式化的Windows风格路径
    getFormattedPath() {
      let path = this.fileSystem.currentPath;
      // 将Linux风格路径转换为Windows风格路径
      // 根目录显示为 .\
      if (path === '/') {
        return '.\\';
      }
      // 替换斜杠并添加前导 .\
      return '.\\' + path.substring(1).replace(/\//g, '\\');
    },
    
    // 打字输出效果
    typeOutput(response, isError) {
      if (this.outputIndex < response.length) {
        this.currentOutput += response[this.outputIndex];
        this.outputIndex++;
        
        // 滚动到底部
        this.$nextTick(() => {
          const output = this.$el.querySelector('.terminal-output');
          if (output) {
            output.scrollTop = output.scrollHeight;
          }
        });
        
        // 继续打字
        setTimeout(() => {
          this.typeOutput(response, isError);
        }, this.typingSpeed);
      } else {
        // 输出完成，添加到历史记录
        this.commandHistory.push({
          command: this.currentCommand,
          response: response,
          isError: isError,
          prompt: this.getFormattedPath() // 使用格式化的路径作为输出框的提示符
        });
        
        // 重置状态
        this.isTyping = false;
        this.currentOutput = '';
        this.outputIndex = 0;
        this.currentCommand = null;
        
        // 滚动到底部
        this.$nextTick(() => {
          const output = this.$el.querySelector('.terminal-output');
          if (output) {
            output.scrollTop = output.scrollHeight;
          }
        });
        
        // 命令执行完成后重新聚焦输入框
        this.focusInput();
      }
    },
    
    // 处理上箭头键 - 显示上一条历史命令
    handleUpKey(event) {
      // 阻止默认行为以避免页面滚动
      event.preventDefault();
      
      if (this.commandHistory.length === 0) {
        return;
      }
      
      // 如果是第一次按上箭头，保存当前输入
      if (this.historyIndex === -1) {
        this.tempInput = this.inputText;
        this.historyIndex = this.commandHistory.length - 1;
      } else if (this.historyIndex > 0) {
        this.historyIndex--;
      }
      
      // 显示对应的历史命令
      this.inputText = this.commandHistory[this.historyIndex].command;
      
      // 将光标移动到输入框末尾
      this.$nextTick(() => {
        const input = this.$el.querySelector('.terminal-input');
        if (input) {
          input.setSelectionRange(input.value.length, input.value.length);
        }
      });
    },
    
    // 处理下箭头键 - 显示下一条历史命令
    handleDownKey(event) {
      // 阻止默认行为以避免页面滚动
      event.preventDefault();
      
      if (this.commandHistory.length === 0 || this.historyIndex === -1) {
        return;
      }
      
      if (this.historyIndex < this.commandHistory.length - 1) {
        this.historyIndex++;
        this.inputText = this.commandHistory[this.historyIndex].command;
      } else {
        // 到达历史记录末尾，恢复之前的临时输入
        this.historyIndex = -1;
        this.inputText = this.tempInput;
      }
      
      // 将光标移动到输入框末尾
      this.$nextTick(() => {
        const input = this.$el.querySelector('.terminal-input');
        if (input) {
          input.setSelectionRange(input.value.length, input.value.length);
        }
      });
    },
    
    // 进程管理系统辅助方法
    
    // 获取进程详细信息
    getProcessInfo(pid) {
      return this.processSystem.processes.find(p => p.pid === pid);
    },
    
    // 获取按CPU使用率排序的进程列表
    getProcessesByCPU() {
      return [...this.processSystem.processes].sort((a, b) => b.cpu - a.cpu);
    },
    
    // 获取按内存使用率排序的进程列表
    getProcessesByMemory() {
      return [...this.processSystem.processes].sort((a, b) => b.memory - a.memory);
    },
    
    // 获取系统总内存使用量
    getTotalMemoryUsage() {
      return this.processSystem.processes.reduce((total, process) => total + process.memory, 0);
    },
    
    // 获取系统总CPU使用率
    getTotalCPUUsage() {
      return this.processSystem.processes.reduce((total, process) => total + process.cpu, 0);
    },
    
    // 模拟进程创建（用于测试）
    createProcess(command, user = 'user', description = 'User process') {
      const newPid = Math.max(...this.processSystem.processes.map(p => p.pid)) + 1;
      const newProcess = {
        pid: newPid,
        command: command,
        user: user,
        description: description,
        cpu: 0.0,
        memory: 0.5,
        state: 'S'
      };
      this.processSystem.processes.push(newProcess);
      return newPid;
    },
    
    // 更新系统状态信息
    updateSystemStatus() {
      const sys = this.processSystem.systemInfo;
      sys.totalTasks = this.processSystem.processes.length;
      sys.runningTasks = this.processSystem.processes.filter(p => p.state === 'R').length;
      sys.sleepingTasks = this.processSystem.processes.filter(p => p.state === 'S').length;
      sys.used = this.getTotalMemoryUsage();
      sys.free = sys.memory.total - sys.used;
      
      // 更新CPU使用率
      const totalCPU = this.getTotalCPUUsage();
      sys.cpuUsage.user = totalCPU * 0.7;
      sys.cpuUsage.system = totalCPU * 0.3;
      sys.cpuUsage.idle = 100 - totalCPU;
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
  padding: 20px;
  overflow-y: auto;
  background-color: #000;
}

.window-content h2 {
  margin-top: 0;
  color: white;
}

.window-content p {
  color: white;
  line-height: 1.6;
}
    
.window-content {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  background-color: #000;
  display: flex;
  flex-direction: column;
  position: relative;
}

.terminal-output {
  flex: 1;
  overflow-y: auto;
}

.terminal-output p {
  color: white;
  line-height: 1.6;
  margin: 0 0 8px 0;
}

.command-prompt {
  color: #00ff00;
  font-weight: bold;
  margin-right: 8px;
}

.command-response {
  color: #cccccc;
}

.error-message {
  color: #ff6b6b;
}

.terminal-cursor {
  color: #00ff00;
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

.terminal-input-container {
  display: flex;
  align-items: center;
  margin-top: 8px;
}

.terminal-prompt {
    color: #00ff00;
    font-weight: bold;
    margin-right: 8px;
    min-width: auto;
  }

.terminal-input {
  flex: 1;
  background: transparent;
  border: none;
  color: white;
  font-family: monospace;
  font-size: 14px;
  outline: none;
}

.terminal-input::placeholder {
  color: #666;
}

.terminal-enter {
  color: #666;
  margin-left: 8px;
  transition: color 0.2s ease;
}

.terminal-enter.flash {
  color: #00ff00;
}

.command-line {
  font-family: monospace;
  font-size: 14px;
  white-space: pre-wrap;
  word-break: break-all;
}

/* 终端输入区域 */
.terminal-input-container {
  display: flex;
  align-items: center;
  height: 20px;
  border: 0.5px solid white;
  margin-top: 10px;
}

.terminal-prompt {
  color: white;
  padding: 0 5px;
  font-family: monospace;
  font-size: 14px;
}

.terminal-input {
  flex: 1;
  height: 100%;
  background-color: transparent;
  color: white;
  border: none;
  outline: none;
  padding: 0 5px;
  font-family: monospace;
  font-size: 14px;
}

.terminal-input::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.terminal-enter {
  width: 10px;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  border-left: 1px solid white;
  font-size: 12px;
  transition: all 0.2s ease;
  cursor: pointer;
}

.terminal-enter.flash {
  background-color: white;
  color: black;
}

/* 终端光标样式 - 持续闪烁 */
.terminal-cursor {
  display: inline-block;
  color: white;
  margin-left: 2px;
  font-size: 14px;
  line-height: 1;
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, 49% {
    opacity: 1;
  }
  50%, 100% {
    opacity: 0;
  }
}

/* 命令历史记录样式 */
.command-line {
  margin: 0 0 8px 0;
  line-height: 1.6;
  white-space: pre-wrap;
  color: white;
}

.command-prompt {
  color: white;
  margin-right: 5px;
}

.command-response {
  color: white;
  display: block;
  margin-left: 20px;
}

/* 错误提示样式 - 改为白色 */
.error-message {
  color: white;
}
</style>