# **Vue3 + Tailwind 暗黑模式完整实现指南**

## 一、环境配置

### 1. Tailwind 暗黑模式配置

```
module.exports = {
  darkMode: 'selector', // 基于类名切换模式
  // 其他配置...
}
```

## 二、核心状态管理（Pinia Store）

### 1. 创建主题管理 Store
```
import { defineStore } from 'pinia'

export const useThemeStore = defineStore('theme', {
  state: () => ({
    isDark: false,
    isSystem: true  // 新增系统偏好跟踪
  }),

  actions: {
    // 初始化主题（兼容本地存储和系统偏好）
    initTheme() {
      const savedTheme = localStorage.getItem('theme')
      
      if (savedTheme) {
        this.isDark = savedTheme === 'dark'
        this.isSystem = false
      } else {
        this.followSystem()
      }
      
      this.applyTheme()
    },

    // 跟随系统偏好
    followSystem() {
      this.isDark = window.matchMedia('(prefers-color-scheme: dark)').matches
      this.isSystem = true
      localStorage.removeItem('theme')  // 清除本地存储
    },

    // 切换主题
    toggleTheme() {
      this.isSystem = false
      this.isDark = !this.isDark
      this.applyTheme()
    },

    // 应用主题到DOM
    applyTheme() {
      const root = document.documentElement
      root.classList.toggle('dark', this.isDark)
      
      // 平滑过渡处理
      root.classList.add('theme-transition')
      setTimeout(() => {
        root.classList.remove('theme-transition')
      }, 300)

      // 持久化存储（仅当非系统模式时）
      if (!this.isSystem) {
        localStorage.setItem('theme', this.isDark ? 'dark' : 'light')
      }
    }
  },

  getters: {
    currentTheme: (state) => 
      state.isSystem ? 'system' : state.isDark ? 'dark' : 'light'
  }
})
```

## 三、初始化主题

### 在根组件初始化
```
<script setup>
import { useThemeStore } from '@/stores/theme'
import { onMounted } from 'vue'

const themeStore = useThemeStore()

onMounted(() => {
  themeStore.initTheme()
  // 监听系统偏好变化
  window.matchMedia('(prefers-color-scheme: dark)')
    .addEventListener('change', e => {
      if (themeStore.isSystem) {
        themeStore.isDark = e.matches
        themeStore.applyTheme()
      }
    })
})
</script>
```

## 四、全局过渡效果（暗紫色）

```
/* ================= 主题切换过渡动画 ================= */
/* 适用场景：主题切换时所有子元素过渡 */
/* 可调节参数：
   - 过渡时长（推荐0.2s-0.5s之间）
   - 贝塞尔曲线值（保持缓入缓出效果）
   - 需要过渡的属性（可添加transform等） */
.theme-transition * {
    transition:
            background-color 0.3s cubic-bezier(0.3, 0, 0.5, 1),  /* 主要背景过渡 */
            border-color 0.2s cubic-bezier(0.3, 0, 0.5, 1),     /* 边框颜色过渡 */
            color 0.2s cubic-bezier(0.3, 0, 0.5, 1) !important; /* 文字颜色过渡 */
}

/* ================= 基础布局样式 ================= */
html, body {
    /* 明暗模式字体颜色配置（可替换品牌色） */
    @apply text-[#24292f] dark:text-[#c9d1d9];
    height: 100%;
    margin: 0;
    padding: 0;
    overflow-x: hidden; /* 防止横向滚动 */
}

body {
    position: relative;
    min-height: 100vh;  /* 确保内容始终撑满视口 */
    width: 100%;
    /* 亮色模式背景色（可替换为其他颜色） */
    @apply bg-white;
}

/* ================= 暗色模式科技风背景 ================= */
/* 主背景层（可调节参数：渐变角度/颜色/位置） */
.dark body {
    background:
        /* 125度角的多色渐变背景（颜色可调整） */
            linear-gradient(125deg,
            #000000 0%,
            #1a1a2f 25%,
            #2a1b3d 50%,
            #1a1a2f 75%,
            #000000 100%
            ),
                /* 中心紫色光晕（尺寸/透明度可调） */
            radial-gradient(
                    circle at 50% 50%,
                    rgba(147, 51, 234, 0.15) 0%,
                    rgba(147, 51, 234, 0) 50%
            );
}

/* 网格覆盖层（伪元素） */
.dark body::before {
    content: '';
    position: fixed;  /* 固定定位避免滚动重绘 */
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    /* 复合网格背景（可调节线条颜色/间距/角度） */
    background:
        /* 水平/垂直渐变线（透明度可调） */
            linear-gradient(90deg, transparent 0%, rgba(147, 51, 234, 0.1) 50%, transparent 100%),
            linear-gradient(0deg, transparent 0%, rgba(147, 51, 234, 0.1) 50%, transparent 100%),
                /* 45度斜线网格（间距10px） */
            repeating-linear-gradient(45deg,
            rgba(147, 51, 234, 0.05) 0px,
            rgba(147, 51, 234, 0.05) 1px,
            transparent 1px,
            transparent 10px),
                /* -45度斜线网格（形成交叉效果） */
            repeating-linear-gradient(-45deg,
            rgba(147, 51, 234, 0.05) 0px,
            rgba(147, 51, 234, 0.05) 1px,
            transparent 1px,
            transparent 10px);
    /* 背景尺寸配置（网格密度可调） */
    background-size:
            100% 100%,   /* 水平/垂直渐变尺寸 */
            100% 100%,
            20px 20px,   /* 斜线网格单元尺寸 */
            20px 20px;
    pointer-events: none; /* 禁用交互提升性能 */
    z-index: -1;     /* 确保内容在上层 */
}

/* 光晕层（伪元素） */
.dark body::after {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    /* 多光源配置（位置/大小可调） */
    background:
        /* 左上角紫色光斑 */
            radial-gradient(
                    circle at 20% 20%,
                    rgba(147, 51, 234, 0.15) 0%,
                    transparent 25%
            ),
                /* 右下角紫色光斑 */
            radial-gradient(
                    circle at 80% 80%,
                    rgba(147, 51, 234, 0.15) 0%,
                    transparent 25%
            ),
                /* 中心光晕效果 */
            radial-gradient(
                    circle at 50% 50%,
                    rgba(147, 51, 234, 0.1) 0%,
                    transparent 50%
            );
    pointer-events: none;
    z-index: -1;
}

/* ================= 系统级暗色模式适配 ================= */
/* 当系统启用暗色模式且未手动设置主题时生效 */
@media (prefers-color-scheme: dark) {
    :root:not([data-theme]) {  /* data-theme属性不存在时应用 */
        @apply text-[#c9d1d9];  /* 使用暗色文字 */
    }
}
```

## 五、主题切换组件示例

```
<script setup>
import { useThemeStore } from '@/stores/theme'
import { MoonIcon, SunIcon } from '@heroicons/vue/24/outline'

const theme = useThemeStore()
</script>

<template>
  <button 
    @click="theme.toggleTheme"
    class="p-3 rounded-full bg-opacity-20 hover:bg-opacity-30 transition-all
           bg-gray-300 dark:bg-gray-600 relative group"
  >
    <SunIcon class="w-6 h-6 text-yellow-500 absolute opacity-0 dark:opacity-100" />
    <MoonIcon class="w-6 h-6 text-blue-400 opacity-100 dark:opacity-0" />
    
    <span class="sr-only">切换主题</span>
    
    <!-- 悬浮提示 -->
    <div class="absolute -bottom-8 left-1/2 -translate-x-1/2 
                bg-gray-800 text-white px-2 py-1 rounded text-sm
                opacity-0 group-hover:opacity-100 transition-opacity">
      {{ theme.currentTheme === 'dark' ? '深色模式' : '浅色模式' }}
    </div>
  </button>
</template>
```

## 六、实现原理

1. **类名控制**：通过切换根元素的 `dark` 类名触发 Tailwind 的暗黑模式样式
2. **三级优先级**：
   - 用户手动选择（localStorage）
   - 系统偏好（matchMedia）
   - 默认 light 模式
3. **平滑过渡**：通过 CSS transition 实现颜色渐变效果