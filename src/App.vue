<template>
  <div 
    class="app-container" 
    :class="{'dark-theme': theme === 'dark'}"
    :style="backgroundStyle"
  >
    <!-- 星光粒子背景 -->
    <div class="particles-container" ref="particlesContainer">
      <div 
        v-for="particle in particles" 
        :key="particle.id"
        class="particle"
        :class="{'twinkling': isPlaying && particle.shouldTwinkle}"
        :style="{
          left: particle.x + '%',
          top: particle.y + '%',
          width: particle.size + 'px',
          height: particle.size + 'px',
          opacity: particle.opacity,
          animationDelay: particle.delay + 's',
          animationDuration: particle.duration + 's'
        }"
      ></div>
    </div>

    <!-- 动态渐变背景层 -->
    <div class="gradient-layer" :style="gradientStyle"></div>

    <!-- 顶部状态栏占位 -->
    <div class="status-bar-placeholder"></div>
    
    <!-- 顶部标题栏 -->
    <header class="header">
      <div class="header-left">
        <div class="logo">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 3V21M12 3L8 7M12 3L16 7M12 21L8 17M12 21L16 17" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
        <h1 class="title">音乐星空</h1>
      </div>
      <div class="header-right">
        <button 
          class="icon-btn heart-mode-btn" 
          :class="{active: heartMode}"
          @click="toggleHeartMode"
        >
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
        <button 
          class="icon-btn theme-btn" 
          :class="{'theme-active': theme === 'light'}"
          @click="toggleTheme"
        >
          <svg v-if="theme === 'dark'" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 3V5M12 19V21M4.22 4.22L5.64 5.64M18.36 18.36L19.78 19.78M1 12H3M21 12H23M4.22 19.78L5.64 18.36M18.36 5.64L19.78 4.22" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            <circle cx="12" cy="12" r="5" stroke="currentColor" stroke-width="2"/>
          </svg>
          <svg v-else viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
      </div>
    </header>

    <!-- 主内容区域 -->
    <main class="main-content" ref="mainContent">
      <!-- 3D专辑圆盘区域 -->
      <section class="album-section">
        <div class="album-wrapper">
          <div class="album-container perspective" :class="{'transitioning': isTransitioning}">
            <!-- 发光外环 -->
            <div class="album-glow" :class="{'glowing': isPlaying}"></div>
            
            <!-- 3D圆盘 -->
            <div class="album-disc preserve-3d" :class="{'rotating': isPlaying, 'slowing': isSlowing, 'starting': isStarting}">
              <div class="disc-front">
                <!-- 磨砂玻璃效果 -->
                <div class="disc-glass">
                  <div class="disc-cover" :class="{'breathing': isPlaying}">
                    <img :src="currentSong.cover" :alt="currentSong.title" />
                    <div class="cover-reflection"></div>
                  </div>
                </div>
                <div class="disc-center">
                  <div class="disc-center-inner"></div>
                </div>
              </div>
              <div class="disc-back">
                <div class="disc-pattern"></div>
              </div>
            </div>
          </div>
          
          <!-- 立体阴影 -->
          <div class="album-shadow-layer">
            <div class="album-shadow shadow-1"></div>
            <div class="album-shadow shadow-2"></div>
          </div>
        </div>
        
        <!-- 歌曲信息 -->
        <div class="song-info">
          <div class="title-wrapper">
            <h2 class="song-title">{{ currentSong.title }}</h2>
            <button 
              class="favorite-btn" 
              :class="{liked: currentSong.liked, 'animating': heartAnimating}"
              @click="toggleLike"
            >
              <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
          </div>
          <p class="song-artist">{{ currentSong.artist }}</p>
        </div>
      </section>

      <!-- 歌词按钮 (非全屏模式) -->
      <div class="lyrics-toggle-wrapper" v-if="!showFullscreenLyrics">
        <button class="lyrics-expand-btn" @click="toggleFullscreenLyrics">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M4 6h16M4 12h16M4 18h12" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <span>歌词</span>
        </button>
      </div>

      <!-- 分类导航 -->
      <section class="category-section">
        <div class="category-scroll hide-scrollbar">
          <button 
            v-for="cat in categories" 
            :key="cat.id"
            class="category-btn"
            :class="{'active': currentCategory === cat.id}"
            @click="selectCategory(cat.id)"
          >
            <span class="category-dot" :class="{'active': currentCategory === cat.id}"></span>
            {{ cat.name }}
          </button>
        </div>
      </section>

      <!-- 热门推荐歌单 -->
      <section class="playlist-section">
        <div class="section-header">
          <h3 class="section-title">热门推荐</h3>
          <span class="section-count">{{ filteredPlaylist.length }} 首</span>
        </div>
        <div class="playlist-container hide-scrollbar">
          <div 
            v-for="song in filteredPlaylist" 
            :key="song.id"
            class="playlist-item"
            :class="{'playing': currentSong.id === song.id, 'transitioning': isTransitioning && currentSong.id === song.id}"
            @click="playSong(song)"
          >
            <div class="playlist-cover-wrapper">
              <div class="playlist-cover" :class="{'playing-cover': currentSong.id === song.id && isPlaying}">
                <img :src="song.cover" :alt="song.title" />
                <div class="cover-overlay"></div>
              </div>
              <div class="play-indicator" v-if="currentSong.id === song.id && isPlaying">
                <div class="playing-animation">
                  <span></span><span></span><span></span><span></span>
                </div>
              </div>
              <div class="play-pause-overlay" v-else-if="currentSong.id === song.id">
                <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M8 5v14l11-7z" fill="white"/>
                </svg>
              </div>
            </div>
            <div class="playlist-info">
              <h4 class="playlist-title">{{ song.title }}</h4>
              <p class="playlist-artist">{{ song.artist }}</p>
              <div class="playlist-meta">
                <span class="song-duration">{{ formatTime(song.duration) }}</span>
                <span class="category-tag" v-for="cat in getCategoryNames(song.category)" :key="cat">{{ cat }}</span>
              </div>
            </div>
            <button 
              class="playlist-favorite"
              :class="{liked: song.liked}"
              @click.stop="toggleSongLike(song)"
            >
              <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
          </div>
        </div>
      </section>
    </main>

    <!-- 全屏歌词面板 -->
    <div class="fullscreen-lyrics" :class="{'show': showFullscreenLyrics}">
      <div class="lyrics-header">
        <button class="lyrics-close-btn" @click="toggleFullscreenLyrics">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M19 12H5M12 19l-7-7 7-7" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
        <div class="lyrics-song-info">
          <h4 class="lyrics-title">{{ currentSong.title }}</h4>
          <p class="lyrics-artist">{{ currentSong.artist }}</p>
        </div>
        <div class="lyrics-header-placeholder"></div>
      </div>
      
      <div class="fullscreen-lyrics-container" ref="fullscreenLyricsContainer">
        <div class="lyrics-padding-top"></div>
        <div 
          v-for="(line, index) in parsedLyrics" 
          :key="index"
          class="fullscreen-lyrics-line"
          :class="{'highlight': currentLyricIndex === index, 'passed': currentLyricIndex > index}"
          :data-index="index"
        >
          {{ line.text }}
        </div>
        <div class="lyrics-padding-bottom"></div>
      </div>
      
      <!-- 全屏歌词模式下的迷你播放控制 -->
      <div class="mini-player-bar">
        <div class="mini-progress">
          <div class="mini-progress-bg"></div>
          <div class="mini-progress-fill" :style="{width: progressPercent + '%'}">
            <div class="mini-progress-shimmer"></div>
          </div>
        </div>
        <div class="mini-controls">
          <button class="mini-control-btn" @click="playPrev">
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M6 6h2v12H6zm3.5 6l8.5 6V6z" fill="currentColor"/>
            </svg>
          </button>
          <button class="mini-play-btn" @click="togglePlay">
            <svg v-if="!isPlaying" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M8 5v14l11-7z" fill="currentColor"/>
            </svg>
            <svg v-else viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" fill="currentColor"/>
            </svg>
          </button>
          <button class="mini-control-btn" @click="playNext">
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M6 18l8.5-6L6 6v12zM16 6v12h2V6h-2z" fill="currentColor"/>
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- 播放控制栏 -->
    <footer class="player-footer" :class="{'hidden': showFullscreenLyrics}">
      <!-- 进度条 -->
      <div class="progress-container">
        <div class="progress-wrapper" @click="seekToPosition" @mousedown="startDragging" @touchstart="startDragging">
          <div class="progress-bar">
            <div class="progress-bg"></div>
            <div class="progress-track" :style="{width: progressPercent + '%'}">
              <div class="progress-fill">
                <div class="progress-shimmer"></div>
              </div>
            </div>
            <div class="progress-thumb" :style="{left: progressPercent + '%'}">
              <div class="thumb-inner"></div>
              <div class="thumb-glow"></div>
            </div>
          </div>
        </div>
        <div class="time-display">
          <span class="current-time">{{ formatTime(currentTime) }}</span>
          <span class="total-time">{{ formatTime(duration) }}</span>
        </div>
      </div>

      <!-- 控制按钮 -->
      <div class="controls-container">
        <button class="control-btn mode-btn" :class="{'active-mode': playMode !== 'loop'}" @click="togglePlayMode">
          <svg v-if="playMode === 'loop'" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M4 12a8 8 0 0 1 8-8V2.5l4 3.5-4 3.5V8a6 6 0 1 0 3.25 10.75l2.5 2.5A9.5 9.5 0 0 1 4 12z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <svg v-else-if="playMode === 'single'" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M4 12a8 8 0 0 1 8-8V2.5l4 3.5-4 3.5V8a6 6 0 1 0 3.25 10.75l2.5 2.5A9.5 9.5 0 0 1 4 12z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            <text x="12" y="15" text-anchor="middle" font-size="8" fill="currentColor">1</text>
          </svg>
          <svg v-else viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M16 3h5v5M4 20L21 3M21 16v5h-5M15 15l6 6M4 4l5 5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
        
        <button class="control-btn nav-btn prev-btn" @click="playPrev">
          <div class="btn-glow"></div>
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M6 6h2v12H6zm3.5 6l8.5 6V6z" fill="currentColor"/>
          </svg>
        </button>
        
        <button class="control-btn play-btn" @click="togglePlay">
          <div class="play-btn-glow"></div>
          <div class="play-btn-inner">
            <svg v-if="!isPlaying" class="play-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M8 5v14l11-7z" fill="currentColor"/>
            </svg>
            <svg v-else class="pause-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" fill="currentColor"/>
            </svg>
          </div>
        </button>
        
        <button class="control-btn nav-btn next-btn" @click="playNext">
          <div class="btn-glow"></div>
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M6 18l8.5-6L6 6v12zM16 6v12h2V6h-2z" fill="currentColor"/>
          </svg>
        </button>
        
        <button class="control-btn lyrics-btn" :class="{'active': showFullscreenLyrics}" @click="toggleFullscreenLyrics">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M4 6h16M4 12h16M4 18h12" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
      </div>

      <!-- 音量控制 -->
      <div class="volume-container">
        <button class="volume-btn" @click="toggleMute">
          <svg v-if="volume === 0 || muted" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M16.5 12c0-1.77-1.02-3.29-2.5-4.03v2.21l2.45 2.45c.03-.2.05-.41.05-.63zm2.5 0c0 .94-.2 1.82-.54 2.64l1.51 1.51C20.63 14.91 21 13.5 21 12c0-4.28-2.99-7.86-7-8.77v2.06c2.89.86 5 3.54 5 6.71zM4.27 3L3 4.27 7.73 9H3v6h4l5 5v-6.73l4.25 4.25c-.67.52-1.42.93-2.25 1.18v2.06c1.38-.31 2.63-.95 3.69-1.81L19.73 21 21 19.73l-9-9L4.27 3zM12 4L9.91 6.09 12 8.18V4z" fill="currentColor"/>
          </svg>
          <svg v-else-if="volume < 50" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M5 9v6h4l5 5V4L9 9H5zm11.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02z" fill="currentColor"/>
          </svg>
          <svg v-else viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5 6.71s-2.11 5.85-5 6.71v2.06c4.01-.91 7-4.49 7-8.77s-2.99-7.86-7-8.77z" fill="currentColor"/>
          </svg>
        </button>
        <div class="volume-slider-wrapper">
          <div class="volume-track">
            <div class="volume-bg"></div>
            <div class="volume-fill" :style="{width: (muted ? 0 : volume) + '%'}">
              <div class="volume-glow"></div>
            </div>
            <div class="volume-thumb" :style="{left: (muted ? 0 : volume) + '%'}">
              <div class="volume-thumb-inner"></div>
            </div>
          </div>
          <input 
            type="range" 
            class="volume-slider-input"
            min="0" 
            max="100" 
            :value="muted ? 0 : volume"
            @input="setVolume($event.target.value)"
          />
        </div>
      </div>

      <!-- 底部安全区域占位 -->
      <div class="safe-area-bottom"></div>
    </footer>

    <!-- 底部装饰光影 -->
    <div class="bottom-decoration">
      <div class="light-orbit light-orbit-1" :class="{'active': isPlaying}"></div>
      <div class="light-orbit light-orbit-2" :class="{'active': isPlaying}"></div>
      <div class="light-orbit light-orbit-3" :class="{'active': isPlaying}"></div>
    </div>

    <!-- 歌曲切换转场遮罩 -->
    <div class="transition-overlay" :class="{'active': isTransitioning}"></div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'

// 主题
const theme = ref('dark')

// 心动模式
const heartMode = ref(false)

// 播放模式
const playMode = ref('loop') // loop, single, random

// 是否显示全屏歌词
const showFullscreenLyrics = ref(false)

// 播放状态
const isPlaying = ref(false)

// 当前播放时间
const currentTime = ref(0)

// 总时长
const duration = ref(180)

// 音量
const volume = ref(70)
const muted = ref(false)

// 当前歌词索引
const currentLyricIndex = ref(-1)

// 当前分类
const currentCategory = ref('all')

// 转场动画状态
const isTransitioning = ref(false)
const isSlowing = ref(false)
const isStarting = ref(false)

// 爱心动画
const heartAnimating = ref(false)

// 拖拽状态
const isDragging = ref(false)

// 分类列表
const categories = [
  { id: 'all', name: '全部' },
  { id: 'pop', name: '流行' },
  { id: 'chinese', name: '古风' },
  { id: 'electronic', name: '电音' },
  { id: 'rock', name: '摇滚' },
  { id: 'light', name: '轻音乐' }
]

// 分类名称映射
const categoryMap = {
  'all': '全部',
  'pop': '流行',
  'chinese': '古风',
  'electronic': '电音',
  'rock': '摇滚',
  'light': '轻音乐'
}

// 歌单数据
const playlist = ref([
  {
    id: 1,
    title: '星空之舞',
    artist: '月光精灵',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=fantasy%20galaxy%20music%20album%20cover%20neon%20purple%20blue%20cosmic%20stars%20artistic%20dreamy%20ethereal&image_size=square_hd',
    category: ['pop', 'electronic'],
    duration: 245,
    liked: false,
    lyrics: `[00:00.00]星空之舞
[00:03.00]月光精灵
[00:06.00]
[00:10.00]夜色中星光闪烁
[00:15.00]银河在眼前流淌
[00:20.00]月光洒落如银纱
[00:25.00]微风轻轻吹过脸庞
[00:30.00]
[00:35.00]星空下独自起舞
[00:40.00]影子在地面摇曳
[00:45.00]这一刻世界静止
[00:50.00]只有心跳在回响
[00:55.00]
[01:00.00]哦~星空之舞
[01:05.00]让灵魂自由飞翔
[01:10.00]哦~星空之舞
[01:15.00]在这无尽的宇宙中
[01:20.00]
[01:25.00]流星划过天际
[01:30.00]许下最美的愿望
[01:35.00]希望这一刻永恒
[01:40.00]让星光永远照亮
[01:45.00]
[01:50.00]哦~星空之舞
[01:55.00]让灵魂自由飞翔
[02:00.00]哦~星空之舞
[02:05.00]在这无尽的宇宙中
[02:10.00]
[02:15.00]舞吧 舞吧
[02:20.00]在星光下
[02:25.00]唱吧 唱吧
[02:30.00]这星空之舞
[02:35.00]
[02:40.00]`,
    gradient: {
      dark: 'linear-gradient(180deg, #0f0f23 0%, #1a1a3e 50%, #2d1b4e 100%)',
      light: 'linear-gradient(180deg, #667eea 0%, #764ba2 50%, #f093fb 100%)'
    },
    accentColor: '#9f7aea'
  },
  {
    id: 2,
    title: '樱花落',
    artist: '古风韵',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=traditional%20chinese%20sakura%20cherry%20blossom%20music%20album%20cover%20pink%20elegant%20artistic%20watercolor%20oriental&image_size=square_hd',
    category: ['chinese', 'light'],
    duration: 198,
    liked: true,
    lyrics: `[00:00.00]樱花落
[00:03.00]古风韵
[00:06.00]
[00:10.00]春风拂过樱花树
[00:15.00]花瓣纷纷飘落
[00:20.00]似雪非雪漫天舞
[00:25.00]伊人独立花树下
[00:30.00]
[00:35.00]琴声悠悠入耳来
[00:40.00]心思绵绵无处寄
[00:45.00]问君此去几时还
[00:50.00]樱花落尽人未归
[00:55.00]
[01:00.00]樱花落 落不尽
[01:05.00]三生三世的情缘
[01:10.00]琴声远 心依旧
[01:15.00]只愿与君长相守
[01:20.00]
[01:25.00]月照西楼人不眠
[01:30.00]酒入愁肠思更切
[01:35.00]梦里不知身是客
[01:40.00]一晌贪欢到天明
[01:45.00]
[01:50.00]樱花落 落不尽
[01:55.00]三生三世的情缘
[02:00.00]琴声远 心依旧
[02:05.00]只愿与君长相守
[02:10.00]
[02:15.00]`,
    gradient: {
      dark: 'linear-gradient(180deg, #1a0a1a 0%, #2d1b2d 50%, #3d243d 100%)',
      light: 'linear-gradient(180deg, #f093fb 0%, #f5576c 50%, #ffecd2 100%)'
    },
    accentColor: '#f687b3'
  },
  {
    id: 3,
    title: '电子脉冲',
    artist: 'CyberBeat',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=cyberpunk%20electronic%20music%20album%20cover%20neon%20blue%20purple%20futuristic%20glowing%20digital%20holographic&image_size=square_hd',
    category: ['electronic', 'rock'],
    duration: 215,
    liked: false,
    lyrics: `[00:00.00]电子脉冲
[00:03.00]CyberBeat
[00:06.00]
[00:10.00]霓虹灯照亮城市
[00:15.00]数字流穿梭在夜空
[00:20.00]电子脉冲在跳动
[00:25.00]唤醒沉睡的灵魂
[00:30.00]
[00:35.00]虚拟与现实交织
[00:40.00]边界变得模糊
[00:45.00]在这赛博的世界
[00:50.00]我找到了自由
[00:55.00]
[01:00.00]电子脉冲 不停跳动
[01:05.00]穿越时空的节奏
[01:10.00]电子脉冲 唤醒真我
[01:15.00]在数字的海洋中遨游
[01:20.00]
[01:25.00]代码在屏幕上流淌
[01:30.00]音乐在空气中弥漫
[01:35.00]这一刻我是主角
[01:40.00]在电子的世界里
[01:45.00]
[01:50.00]电子脉冲 不停跳动
[01:55.00]穿越时空的节奏
[02:00.00]电子脉冲 唤醒真我
[02:05.00]在数字的海洋中遨游
[02:10.00]`,
    gradient: {
      dark: 'linear-gradient(180deg, #0a1628 0%, #1a2a4a 50%, #0d3b66 100%)',
      light: 'linear-gradient(180deg, #4facfe 0%, #00f2fe 50%, #43e97b 100%)'
    },
    accentColor: '#38bdf8'
  },
  {
    id: 4,
    title: '摇滚之心',
    artist: '火焰乐队',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=rock%20music%20album%20cover%20fire%20electric%20guitar%20red%20orange%20passionate%20energetic%20explosive&image_size=square_hd',
    category: ['rock', 'pop'],
    duration: 228,
    liked: true,
    lyrics: `[00:00.00]摇滚之心
[00:03.00]火焰乐队
[00:06.00]
[00:10.00]吉他声划破夜空
[00:15.00]鼓声震撼着大地
[00:20.00]贝斯在低音嘶吼
[00:25.00]这是摇滚的力量
[00:30.00]
[00:35.00]血管中燃烧着火焰
[00:40.00]灵魂渴望着自由
[00:45.00]在这疯狂的世界
[00:50.00]音乐是唯一的救赎
[00:55.00]
[01:00.00]摇滚之心 永不熄灭
[01:05.00]在黑暗中燃烧
[01:10.00]摇滚之心 永远年轻
[01:15.00]在音乐中永生
[01:20.00]
[01:25.00]台下万人齐声唱
[01:30.00]灯光闪烁如繁星
[01:35.00]汗水湿透了衣衫
[01:40.00]这就是摇滚的意义
[01:45.00]
[01:50.00]摇滚之心 永不熄灭
[01:55.00]在黑暗中燃烧
[02:00.00]摇滚之心 永远年轻
[02:05.00]在音乐中永生
[02:10.00]
[02:15.00]摇滚吧 直到天亮
[02:20.00]摇滚吧 直到永远
[02:25.00]`,
    gradient: {
      dark: 'linear-gradient(180deg, #1a0a0a 0%, #2d1a1a 50%, #3d1f1f 100%)',
      light: 'linear-gradient(180deg, #fa709a 0%, #fee140 50%, #ff6b6b 100%)'
    },
    accentColor: '#f87171'
  },
  {
    id: 5,
    title: '宁静午后',
    artist: '轻音乐集',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=peaceful%20afternoon%20light%20music%20album%20cover%20soft%20warm%20colors%20piano%20violin%20calm%20serene%20nature&image_size=square_hd',
    category: ['light'],
    duration: 185,
    liked: false,
    lyrics: `[00:00.00]宁静午后
[00:03.00]轻音乐集
[00:06.00]
[00:10.00]阳光透过窗帘
[00:15.00]洒下金色的光芒
[00:20.00]微风轻轻吹过
[00:25.00]带来阵阵花香
[00:30.00]
[00:35.00]钢琴声缓缓响起
[00:40.00]小提琴轻声附和
[00:45.00]这一刻时间静止
[00:50.00]心灵得到安息
[00:55.00]
[01:00.00]宁静的午后
[01:05.00]让心灵找到归宿
[01:10.00]宁静的午后
[01:15.00]让烦恼随风而去
[01:20.00]
[01:25.00]一杯清茶在手
[01:30.00]一本好书在旁
[01:35.00]音乐流淌心间
[01:40.00]这就是幸福的模样
[01:45.00]
[01:50.00]宁静的午后
[01:55.00]让心灵找到归宿
[02:00.00]宁静的午后
[02:05.00]让烦恼随风而去
[02:10.00]`,
    gradient: {
      dark: 'linear-gradient(180deg, #0a1a1a 0%, #1a2d2d 50%, #1f3d3d 100%)',
      light: 'linear-gradient(180deg, #89f7fe 0%, #66a6ff 50%, #c9ffbf 100%)'
    },
    accentColor: '#67e8f9'
  },
  {
    id: 6,
    title: '都市节拍',
    artist: '流行达人',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=modern%20city%20pop%20music%20album%20cover%20urban%20skyline%20night%20lights%20vibrant%20colors%20stylish%20trendy&image_size=square_hd',
    category: ['pop'],
    duration: 202,
    liked: true,
    lyrics: `[00:00.00]都市节拍
[00:03.00]流行达人
[00:06.00]
[00:10.00]城市的霓虹闪烁
[00:15.00]人群在街头穿梭
[00:20.00]每个人都有故事
[00:25.00]在这喧嚣中沉默
[00:30.00]
[00:35.00]耳机里的旋律
[00:40.00]是属于自己的世界
[00:45.00]跟着节奏摇摆
[00:50.00]忘记所有的烦恼
[00:55.00]
[01:00.00]都市的节拍
[01:05.00]让我感受到存在
[01:10.00]都市的节拍
[01:15.00]让梦想在心中澎湃
[01:20.00]
[01:25.00]地铁站的人来人往
[01:30.00]写字楼的灯火通明
[01:35.00]在这钢筋水泥森林
[01:40.00]音乐是最好的陪伴
[01:45.00]
[01:50.00]都市的节拍
[01:55.00]让我感受到存在
[02:00.00]`,
    gradient: {
      dark: 'linear-gradient(180deg, #1a1a0a 0%, #2d2d1a 50%, #3d3d1f 100%)',
      light: 'linear-gradient(180deg, #a8edea 0%, #fed6e3 50%, #ffecd2 100%)'
    },
    accentColor: '#fbbf24'
  }
])

// 当前播放歌曲
const currentSong = ref(playlist.value[0])

// 过滤后的歌单
const filteredPlaylist = computed(() => {
  if (currentCategory.value === 'all') {
    return playlist.value
  }
  return playlist.value.filter(song => song.category.includes(currentCategory.value))
})

// 进度百分比
const progressPercent = computed(() => {
  return (currentTime.value / duration.value) * 100
})

// 背景样式
const backgroundStyle = computed(() => {
  const song = currentSong.value
  const accentColor = song?.accentColor || '#ff6b6b'
  const bg = theme.value === 'dark' 
    ? (song?.gradient?.dark || 'linear-gradient(180deg, #0a0a0a 0%, #1a1a2e 50%, #16213e 100%)')
    : (song?.gradient?.light || 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)')
  return {
    '--accent-color': accentColor,
    '--accent-gradient': `linear-gradient(135deg, ${accentColor}, #feca57)`,
    background: bg
  }
})

// 动态渐变样式
const gradientStyle = computed(() => {
  const song = currentSong.value
  const colors = theme.value === 'dark'
    ? [
        'radial-gradient(circle at 20% 30%, ' + (song?.accentColor || '#ff6b6b') + '1a 0%, transparent 50%)',
        'radial-gradient(circle at 80% 70%, ' + (song?.accentColor || '#4ecdc4') + '1a 0%, transparent 50%)',
        'radial-gradient(circle at 50% 50%, ' + (song?.accentColor || '#feca57') + '0d 0%, transparent 60%)'
      ]
    : [
        'radial-gradient(circle at 20% 30%, rgba(255, 255, 255, 0.1) 0%, transparent 50%)',
        'radial-gradient(circle at 80% 70%, rgba(255, 255, 255, 0.1) 0%, transparent 50%)'
      ]
  
  return {
    background: colors.join(', ')
  }
})

// 解析歌词
const parsedLyrics = computed(() => {
  const lyrics = currentSong.value.lyrics
  const lines = lyrics.split('\n')
  const result = []
  
  lines.forEach(line => {
    const match = line.match(/\[(\d{2}):(\d{2})\.(\d{2,3})\](.*)/)
    if (match) {
      const minutes = parseInt(match[1])
      const seconds = parseInt(match[2])
      const milliseconds = parseInt(match[3].padEnd(3, '0'))
      const time = minutes * 60 + seconds + milliseconds / 1000
      const text = match[4].trim()
      if (text) {
        result.push({ time, text })
      }
    }
  })
  
  return result
})

// 粒子效果
const particles = ref([])

// 引用
const mainContent = ref(null)
const fullscreenLyricsContainer = ref(null)
const particlesContainer = ref(null)

// 播放计时器
let playInterval = null

// 格式化时间
const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = Math.floor(seconds % 60)
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

// 获取分类名称
const getCategoryNames = (categories) => {
  return categories.map(cat => categoryMap[cat] || cat)
}

// 初始化粒子
const initParticles = () => {
  const particleCount = 60
  const newParticles = []
  
  for (let i = 0; i < particleCount; i++) {
    newParticles.push({
      id: i,
      x: Math.random() * 100,
      y: Math.random() * 100,
      size: Math.random() * 3 + 1,
      opacity: Math.random() * 0.5 + 0.2,
      delay: Math.random() * 5,
      duration: Math.random() * 3 + 2,
      shouldTwinkle: Math.random() > 0.5
    })
  }
  
  particles.value = newParticles
}

// 切换主题
const toggleTheme = () => {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
}

// 切换心动模式
const toggleHeartMode = () => {
  heartMode.value = !heartMode.value
}

// 切换播放状态
const togglePlay = () => {
  if (isPlaying.value) {
    // 暂停：缓停效果
    isSlowing.value = true
    setTimeout(() => {
      isPlaying.value = false
      isSlowing.value = false
      stopPlayback()
    }, 300)
  } else {
    // 播放：缓起效果
    isStarting.value = true
    isPlaying.value = true
    startPlayback()
    setTimeout(() => {
      isStarting.value = false
    }, 500)
  }
}

// 开始播放
const startPlayback = () => {
  duration.value = currentSong.value.duration
  if (playInterval) clearInterval(playInterval)
  playInterval = setInterval(() => {
    if (currentTime.value < duration.value) {
      currentTime.value += 0.1
      updateLyrics()
    } else {
      handleSongEnd()
    }
  }, 100)
}

// 停止播放
const stopPlayback = () => {
  if (playInterval) {
    clearInterval(playInterval)
    playInterval = null
  }
}

// 播放结束处理
const handleSongEnd = () => {
  stopPlayback()
  currentTime.value = 0
  
  if (playMode.value === 'single') {
    togglePlay()
  } else {
    playNext()
  }
}

// 播放上一首
const playPrev = () => {
  const currentIndex = playlist.value.findIndex(s => s.id === currentSong.value.id)
  let prevIndex
  
  if (playMode.value === 'random' && heartMode.value === false) {
    prevIndex = Math.floor(Math.random() * playlist.value.length)
  } else {
    prevIndex = currentIndex === 0 ? playlist.value.length - 1 : currentIndex - 1
  }
  
  playSong(playlist.value[prevIndex])
}

// 播放下一首
const playNext = () => {
  const currentIndex = playlist.value.findIndex(s => s.id === currentSong.value.id)
  let nextIndex
  
  if (playMode.value === 'random' || (heartMode.value && playlist.value.filter(s => s.liked).length > 0)) {
    if (heartMode.value) {
      const likedSongs = playlist.value.filter(s => s.liked)
      if (likedSongs.length > 0) {
        const randomLikedIndex = Math.floor(Math.random() * likedSongs.length)
        nextIndex = playlist.value.findIndex(s => s.id === likedSongs[randomLikedIndex].id)
      } else {
        nextIndex = Math.floor(Math.random() * playlist.value.length)
      }
    } else {
      nextIndex = Math.floor(Math.random() * playlist.value.length)
    }
  } else {
    nextIndex = currentIndex === playlist.value.length - 1 ? 0 : currentIndex + 1
  }
  
  playSong(playlist.value[nextIndex])
}

// 播放指定歌曲（带转场动画）
const playSong = (song) => {
  if (song.id === currentSong.value.id) {
    if (!isPlaying.value) {
      togglePlay()
    }
    return
  }
  
  // 转场动画
  isTransitioning.value = true
  isSlowing.value = true
  
  setTimeout(() => {
    stopPlayback()
    currentSong.value = song
    currentTime.value = 0
    duration.value = song.duration
    currentLyricIndex.value = -1
    isPlaying.value = true
    isSlowing.value = false
    isStarting.value = true
    startPlayback()
    
    setTimeout(() => {
      isTransitioning.value = false
      isStarting.value = false
    }, 400)
  }, 300)
}

// 切换播放模式
const togglePlayMode = () => {
  const modes = ['loop', 'single', 'random']
  const currentIndex = modes.indexOf(playMode.value)
  playMode.value = modes[(currentIndex + 1) % modes.length]
}

// 切换收藏
const toggleLike = () => {
  currentSong.value.liked = !currentSong.value.liked
  heartAnimating.value = true
  setTimeout(() => {
    heartAnimating.value = false
  }, 500)
}

// 切换歌单中的歌曲收藏
const toggleSongLike = (song) => {
  song.liked = !song.liked
}

// 跳转进度
const seekToPosition = (event) => {
  const rect = event.currentTarget.getBoundingClientRect()
  const x = event.clientX - rect.left
  const percent = x / rect.width
  currentTime.value = percent * duration.value
  updateLyrics()
}

// 开始拖拽
const startDragging = () => {
  isDragging.value = true
}

// 设置音量
const setVolume = (val) => {
  volume.value = parseInt(val)
  if (volume.value > 0) {
    muted.value = false
  }
}

// 切换静音
const toggleMute = () => {
  muted.value = !muted.value
}

// 切换全屏歌词
const toggleFullscreenLyrics = () => {
  showFullscreenLyrics.value = !showFullscreenLyrics.value
  if (showFullscreenLyrics.value) {
    nextTick(() => {
      scrollToLyric(currentLyricIndex.value, true)
    })
  }
}

// 选择分类
const selectCategory = (categoryId) => {
  currentCategory.value = categoryId
}

// 更新歌词
const updateLyrics = () => {
  const lyrics = parsedLyrics.value
  if (lyrics.length === 0) return
  
  let newIndex = -1
  for (let i = 0; i < lyrics.length; i++) {
    if (currentTime.value >= lyrics[i].time) {
      newIndex = i
    }
  }
  
  if (newIndex !== currentLyricIndex.value) {
    currentLyricIndex.value = newIndex
    if (showFullscreenLyrics.value) {
      scrollToLyric(newIndex, true)
    }
  }
}

// 滚动到歌词
const scrollToLyric = (index, isFullscreen = false) => {
  const container = isFullscreen ? fullscreenLyricsContainer.value : null
  if (!container || index < 0) return
  
  nextTick(() => {
    const lines = container.querySelectorAll('.fullscreen-lyrics-line')
    if (lines[index]) {
      const containerRect = container.getBoundingClientRect()
      const lineRect = lines[index].getBoundingClientRect()
      const targetScrollTop = container.scrollTop + lineRect.top - containerRect.top - containerRect.height / 2 + lineRect.height / 2
      
      container.scrollTo({
        top: targetScrollTop,
        behavior: 'smooth'
      })
    }
  })
}

// 生命周期
onMounted(() => {
  duration.value = currentSong.value.duration
  initParticles()
})

onUnmounted(() => {
  stopPlayback()
})
</script>

<style scoped>
/* ==================== CSS 变量定义 ==================== */
:root {
  --accent-color: #ff6b6b;
  --accent-gradient: linear-gradient(135deg, #ff6b6b, #feca57);
  --glass-bg: rgba(255, 255, 255, 0.1);
  --glass-border: rgba(255, 255, 255, 0.15);
  --text-primary: rgba(255, 255, 255, 0.95);
  --text-secondary: rgba(255, 255, 255, 0.6);
  --text-muted: rgba(255, 255, 255, 0.4);
}

.app-container {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
  transition: background 0.8s ease;
  background: #0a0a0a;
}

.app-container.dark-theme {
  --glass-bg: rgba(255, 255, 255, 0.06);
  --glass-border: rgba(255, 255, 255, 0.08);
  --text-primary: rgba(255, 255, 255, 0.95);
  --text-secondary: rgba(255, 255, 255, 0.6);
  --text-muted: rgba(255, 255, 255, 0.4);
}

.app-container:not(.dark-theme) {
  --glass-bg: rgba(255, 255, 255, 0.7);
  --glass-border: rgba(255, 255, 255, 0.8);
  --text-primary: rgba(0, 0, 0, 0.9);
  --text-secondary: rgba(0, 0, 0, 0.6);
  --text-muted: rgba(0, 0, 0, 0.4);
  background: linear-gradient(180deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
}

/* ==================== 粒子背景 ==================== */
.particles-container {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 1;
  overflow: hidden;
}

.particle {
  position: absolute;
  border-radius: 50%;
  background: white;
  animation: particleFloat 4s ease-in-out infinite;
  box-shadow: 0 0 6px 2px rgba(255, 255, 255, 0.5);
}

.particle.twinkling {
  animation: particleFloat 4s ease-in-out infinite, particleTwinkle 1.5s ease-in-out infinite;
}

@keyframes particleFloat {
  0%, 100% {
    transform: translateY(0) scale(1);
    opacity: 0.3;
  }
  50% {
    transform: translateY(-20px) scale(1.2);
    opacity: 0.6;
  }
}

@keyframes particleTwinkle {
  0%, 100% {
    opacity: 0.2;
    transform: scale(0.8);
  }
  50% {
    opacity: 0.8;
    transform: scale(1.3);
  }
}

/* ==================== 动态渐变层 ==================== */
.gradient-layer {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 2;
  transition: all 0.8s ease;
  animation: gradientShift 10s ease-in-out infinite;
}

@keyframes gradientShift {
  0%, 100% {
    opacity: 0.6;
  }
  50% {
    opacity: 0.8;
  }
}

/* ==================== 状态栏占位 ==================== */
.status-bar-placeholder {
  height: var(--safe-area-inset-top);
  position: relative;
  z-index: 10;
}

/* ==================== 顶部标题栏 ==================== */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
  padding-top: max(12px, calc(var(--safe-area-inset-top) + 8px));
  position: relative;
  z-index: 10;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--accent-color);
  background: var(--glass-bg);
  border-radius: 12px;
  border: 1px solid var(--glass-border);
  backdrop-filter: blur(20px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.logo svg {
  width: 22px;
  height: 22px;
  filter: drop-shadow(0 0 8px var(--accent-color));
}

.title {
  font-size: 22px;
  font-weight: 800;
  letter-spacing: 0.5px;
  background: var(--accent-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  filter: drop-shadow(0 2px 8px rgba(0, 0, 0, 0.2));
}

.header-right {
  display: flex;
  gap: 10px;
}

.icon-btn {
  width: 40px;
  height: 40px;
  border-radius: 14px;
  border: 1px solid var(--glass-border);
  background: var(--glass-bg);
  backdrop-filter: blur(20px);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  color: var(--text-secondary);
  position: relative;
  overflow: hidden;
}

.icon-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, var(--accent-color), transparent);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.icon-btn:hover::before {
  opacity: 0.1;
}

.icon-btn:hover {
  transform: translateY(-2px) scale(1.02);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
  border-color: rgba(255, 255, 255, 0.2);
  color: var(--text-primary);
}

.icon-btn.active {
  background: linear-gradient(135deg, var(--accent-color), #feca57);
  color: white;
  border: none;
  box-shadow: 0 4px 20px var(--accent-color);
  animation: pulse 2s ease-in-out infinite;
}

.icon-btn.active::before {
  display: none;
}

.theme-btn {
  position: relative;
}

.theme-btn.theme-active {
  background: linear-gradient(135deg, #ffd93d, #ff9f43);
  color: white;
  border: none;
}

.icon-btn svg {
  width: 20px;
  height: 20px;
  position: relative;
  z-index: 1;
}

/* ==================== 主内容区域 ==================== */
.main-content {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 0 20px;
  padding-bottom: 260px;
  position: relative;
  z-index: 5;
  height: calc(100% - 60px);
}

/* ==================== 专辑区域 ==================== */
.album-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 0 20px;
  position: relative;
}

.album-wrapper {
  position: relative;
  margin-bottom: 24px;
}

.album-container {
  position: relative;
  width: 280px;
  height: 280px;
  perspective: 1200px;
}

.album-container.transitioning .album-disc {
  animation-duration: 0.8s;
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

/* 发光外环 */
.album-glow {
  position: absolute;
  top: -15px;
  left: -15px;
  right: -15px;
  bottom: -15px;
  border-radius: 50%;
  background: radial-gradient(circle, var(--accent-color) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 0.5s ease;
  filter: blur(20px);
}

.album-glow.glowing {
  opacity: 0.4;
  animation: glowPulse 2s ease-in-out infinite;
}

@keyframes glowPulse {
  0%, 100% {
    opacity: 0.3;
    transform: scale(1);
  }
  50% {
    opacity: 0.5;
    transform: scale(1.05);
  }
}

/* 3D圆盘 */
.album-disc {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  animation-duration: 12s;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}

.album-disc.rotating {
  animation-name: rotate;
}

.album-disc.slowing {
  animation-duration: 3s;
  animation-timing-function: cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.album-disc.starting {
  animation-duration: 1.5s;
  animation-timing-function: cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

@keyframes rotate {
  0% {
    transform: rotateY(0deg) rotateZ(0deg);
  }
  25% {
    transform: rotateY(15deg) rotateZ(90deg);
  }
  50% {
    transform: rotateY(0deg) rotateZ(180deg);
  }
  75% {
    transform: rotateY(-15deg) rotateZ(270deg);
  }
  100% {
    transform: rotateY(0deg) rotateZ(360deg);
  }
}

.disc-front, .disc-back {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  backface-visibility: hidden;
}

.disc-front {
  background: linear-gradient(145deg, rgba(30, 30, 50, 0.9), rgba(20, 20, 40, 0.95));
  border: 2px solid rgba(255, 255, 255, 0.1);
  box-shadow: 
    0 20px 60px rgba(0, 0, 0, 0.5),
    0 0 0 1px rgba(255, 255, 255, 0.05),
    inset 0 0 40px rgba(255, 255, 255, 0.03);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 28px;
}

/* 磨砂玻璃效果 */
.disc-glass {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.1), rgba(255, 255, 255, 0.05));
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  padding: 8px;
  box-shadow: inset 0 0 30px rgba(255, 255, 255, 0.1);
}

.disc-cover {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  overflow: hidden;
  position: relative;
  transition: transform 0.3s ease;
}

.disc-cover.breathing {
  animation: coverBreath 3s ease-in-out infinite;
}

@keyframes coverBreath {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.02);
  }
}

.disc-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.cover-reflection {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.3) 0%, transparent 40%, transparent 60%, rgba(255, 255, 255, 0.1) 100%);
  pointer-events: none;
  border-radius: 50%;
}

.disc-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: linear-gradient(145deg, #1a1a2e, #0f0f1a);
  box-shadow: 
    inset 0 4px 8px rgba(0, 0, 0, 0.5),
    0 0 0 10px rgba(255, 255, 255, 0.08),
    0 0 0 14px rgba(0, 0, 0, 0.3);
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
}

.disc-center-inner {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: linear-gradient(145deg, #2a2a4a, #1a1a2e);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.5);
}

.disc-back {
  background: linear-gradient(145deg, rgba(20, 20, 40, 0.95), rgba(30, 30, 50, 0.9));
  transform: rotateY(180deg);
  border: 2px solid rgba(255, 255, 255, 0.1);
}

.disc-pattern {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: 
    repeating-radial-gradient(
      circle,
      transparent 0px,
      transparent 8px,
      rgba(255, 255, 255, 0.03) 8px,
      rgba(255, 255, 255, 0.03) 10px
    ),
    radial-gradient(circle, var(--accent-color) 0%, transparent 50%);
  opacity: 0.5;
}

/* 立体阴影 */
.album-shadow-layer {
  position: absolute;
  bottom: -20px;
  left: 50%;
  transform: translateX(-50%);
  width: 90%;
  height: 40px;
}

.album-shadow {
  position: absolute;
  border-radius: 50%;
  filter: blur(15px);
}

.shadow-1 {
  width: 100%;
  height: 100%;
  background: radial-gradient(ellipse, rgba(0, 0, 0, 0.4) 0%, transparent 70%);
  transform: scaleY(0.3);
}

.shadow-2 {
  width: 80%;
  height: 60%;
  background: radial-gradient(ellipse, var(--accent-color) 0%, transparent 60%);
  opacity: 0.2;
  transform: translateX(-50%) scaleY(0.2);
  left: 50%;
  bottom: 0;
}

/* 歌曲信息 */
.song-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  text-align: center;
  width: 100%;
}

.title-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  max-width: 100%;
}

.song-title {
  font-size: 26px;
  font-weight: 800;
  color: var(--text-primary);
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  margin: 0;
  letter-spacing: 0.5px;
  max-width: 260px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.song-artist {
  font-size: 15px;
  color: var(--text-secondary);
  margin: 0;
  font-weight: 500;
  letter-spacing: 1px;
}

.favorite-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background: var(--glass-bg);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  color: var(--text-muted);
  flex-shrink: 0;
  border: 1px solid var(--glass-border);
}

.favorite-btn:hover {
  transform: scale(1.1);
  background: rgba(255, 107, 107, 0.1);
  color: #ff6b6b;
}

.favorite-btn.liked {
  color: #ff6b6b;
  background: linear-gradient(135deg, rgba(255, 107, 107, 0.2), rgba(255, 107, 107, 0.1));
  border-color: rgba(255, 107, 107, 0.3);
}

.favorite-btn.animating {
  animation: heartBounce 0.5s ease;
}

@keyframes heartBounce {
  0% { transform: scale(1); }
  25% { transform: scale(1.3); }
  50% { transform: scale(0.9); }
  75% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

.favorite-btn svg {
  width: 22px;
  height: 22px;
}

.favorite-btn.liked svg {
  fill: #ff6b6b;
}

/* 歌词按钮 */
.lyrics-toggle-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.lyrics-expand-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  border-radius: 20px;
  border: 1px solid var(--glass-border);
  background: var(--glass-bg);
  backdrop-filter: blur(10px);
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.lyrics-expand-btn:hover {
  background: linear-gradient(135deg, var(--accent-color), #feca57);
  color: white;
  border-color: transparent;
  transform: translateY(-2px);
  box-shadow: 0 4px 20px var(--accent-color);
}

.lyrics-expand-btn svg {
  width: 18px;
  height: 18px;
}

/* 分类导航 */
.category-section {
  padding: 10px 0;
  margin-bottom: 20px;
}

.category-scroll {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 8px;
}

.category-btn {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  border-radius: 16px;
  border: 1px solid var(--glass-border);
  background: var(--glass-bg);
  backdrop-filter: blur(20px);
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

.category-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, var(--accent-color), #feca57);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.category-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
  color: var(--text-primary);
}

.category-btn.active {
  background: linear-gradient(135deg, var(--accent-color), #feca57);
  color: white;
  border: none;
  box-shadow: 0 4px 20px var(--accent-color);
}

.category-btn.active::before {
  display: none;
}

.category-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--text-muted);
  transition: all 0.3s ease;
}

.category-btn.active .category-dot {
  background: white;
  box-shadow: 0 0 8px rgba(255, 255, 255, 0.8);
  transform: scale(1.2);
}

/* 歌单区域 */
.playlist-section {
  padding: 10px 0;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.section-title {
  font-size: 18px;
  font-weight: 700;
  color: var(--text-primary);
  margin: 0;
  letter-spacing: 0.5px;
}

.section-count {
  font-size: 13px;
  color: var(--text-muted);
  font-weight: 500;
}

.playlist-container {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.playlist-item {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 14px 16px;
  background: var(--glass-bg);
  backdrop-filter: blur(20px);
  border-radius: 16px;
  border: 1px solid var(--glass-border);
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

.playlist-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, var(--accent-color), transparent);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.playlist-item:hover {
  transform: translateX(6px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
  border-color: rgba(255, 255, 255, 0.2);
}

.playlist-item:hover::before {
  opacity: 0.05;
}

.playlist-item.playing {
  background: linear-gradient(135deg, rgba(255, 107, 107, 0.15), rgba(254, 202, 87, 0.1));
  border-color: var(--accent-color);
  box-shadow: 0 4px 20px var(--accent-color);
}

.playlist-item.playing::before {
  display: none;
}

.playlist-cover-wrapper {
  position: relative;
  flex-shrink: 0;
}

.playlist-cover {
  width: 60px;
  height: 60px;
  border-radius: 12px;
  overflow: hidden;
  position: relative;
  transition: transform 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.playlist-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.cover-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.1) 0%, transparent 50%);
  pointer-events: none;
}

.play-indicator {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.play-pause-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.4);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.play-pause-overlay svg {
  width: 28px;
  height: 28px;
  opacity: 0.9;
}

.playing-animation {
  display: flex;
  align-items: flex-end;
  gap: 4px;
  height: 24px;
}

.playing-animation span {
  width: 4px;
  background: var(--accent-color);
  border-radius: 2px;
  animation: playingBars 0.8s ease infinite;
}

.playing-animation span:nth-child(1) { animation-delay: 0s; }
.playing-animation span:nth-child(2) { animation-delay: 0.15s; }
.playing-animation span:nth-child(3) { animation-delay: 0.3s; }
.playing-animation span:nth-child(4) { animation-delay: 0.45s; }

@keyframes playingBars {
  0%, 100% {
    height: 6px;
  }
  50% {
    height: 24px;
  }
}

.playlist-info {
  flex: 1;
  min-width: 0;
}

.playlist-title {
  font-size: 15px;
  font-weight: 700;
  color: var(--text-primary);
  margin: 0 0 4px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  letter-spacing: 0.3px;
}

.playlist-artist {
  font-size: 13px;
  color: var(--text-secondary);
  margin: 0 0 6px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  font-weight: 500;
}

.playlist-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.song-duration {
  font-size: 12px;
  color: var(--text-muted);
  font-weight: 500;
}

.category-tag {
  font-size: 11px;
  color: var(--accent-color);
  background: linear-gradient(135deg, rgba(255, 107, 107, 0.15), rgba(255, 107, 107, 0.05));
  padding: 3px 10px;
  border-radius: 8px;
  font-weight: 600;
  border: 1px solid rgba(255, 107, 107, 0.2);
}

.playlist-favorite {
  width: 36px;
  height: 36px;
  border-radius: 12px;
  border: none;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: var(--text-muted);
  flex-shrink: 0;
}

.playlist-favorite:hover {
  color: var(--text-primary);
  background: var(--glass-bg);
}

.playlist-favorite.liked {
  color: #ff6b6b;
}

.playlist-favorite.liked svg {
  fill: #ff6b6b;
}

.playlist-favorite svg {
  width: 20px;
  height: 20px;
}

/* 全屏歌词面板 */
.fullscreen-lyrics {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 200;
  background: linear-gradient(180deg, #0a0a0a 0%, #1a1a2e 50%, #16213e 100%);
  transform: translateY(100%);
  transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}

.fullscreen-lyrics.show {
  transform: translateY(0);
}

.fullscreen-lyrics::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 20% 20%, var(--accent-color) 0%, transparent 50%),
    radial-gradient(circle at 80% 80%, var(--accent-color) 0%, transparent 50%);
  opacity: 0.1;
  pointer-events: none;
}

.lyrics-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
  padding-top: max(12px, calc(var(--safe-area-inset-top) + 8px));
  position: relative;
  z-index: 10;
}

.lyrics-close-btn {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  border: 1px solid var(--glass-border);
  background: var(--glass-bg);
  backdrop-filter: blur(20px);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: var(--text-secondary);
}

.lyrics-close-btn:hover {
  transform: scale(1.05);
  color: var(--text-primary);
}

.lyrics-close-btn svg {
  width: 22px;
  height: 22px;
}

.lyrics-song-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.lyrics-title {
  font-size: 18px;
  font-weight: 700;
  color: var(--text-primary);
  margin: 0;
}

.lyrics-artist {
  font-size: 13px;
  color: var(--text-secondary);
  margin: 0;
  font-weight: 500;
}

.lyrics-header-placeholder {
  width: 40px;
}

.fullscreen-lyrics-container {
  position: absolute;
  top: 80px;
  left: 0;
  right: 0;
  bottom: 140px;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 0 20px;
}

.lyrics-padding-top {
  height: calc(50vh - 150px);
}

.lyrics-padding-bottom {
  height: calc(50vh - 150px);
}

.fullscreen-lyrics-line {
  font-size: 22px;
  line-height: 2;
  color: rgba(255, 255, 255, 0.3);
  text-align: center;
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  font-weight: 500;
  letter-spacing: 1px;
  margin: 8px 0;
  padding: 4px 0;
}

.fullscreen-lyrics-line.passed {
  color: rgba(255, 255, 255, 0.25);
  transform: scale(0.95);
}

.fullscreen-lyrics-line.highlight {
  font-size: 28px;
  font-weight: 800;
  color: white;
  text-shadow: 
    0 0 20px var(--accent-color),
    0 0 40px var(--accent-color),
    0 2px 10px rgba(0, 0, 0, 0.5);
  transform: scale(1.05);
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.05), transparent);
  border-radius: 8px;
  margin: 16px 0;
  padding: 8px 0;
  animation: lyricGlow 2s ease-in-out infinite;
}

@keyframes lyricGlow {
  0%, 100% {
    text-shadow: 
      0 0 20px var(--accent-color),
      0 0 40px var(--accent-color),
      0 2px 10px rgba(0, 0, 0, 0.5);
  }
  50% {
    text-shadow: 
      0 0 30px var(--accent-color),
      0 0 60px var(--accent-color),
      0 4px 20px rgba(0, 0, 0, 0.5);
  }
}

/* 迷你播放器 */
.mini-player-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 16px 20px;
  padding-bottom: calc(16px + var(--safe-area-inset-bottom));
  background: linear-gradient(180deg, transparent 0%, rgba(0, 0, 0, 0.8) 30%, rgba(0, 0, 0, 0.95) 100%);
  backdrop-filter: blur(20px);
}

.mini-progress {
  position: relative;
  height: 4px;
  margin-bottom: 16px;
}

.mini-progress-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
}

.mini-progress-fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, var(--accent-color), #feca57);
  border-radius: 2px;
  transition: width 0.1s linear;
  overflow: hidden;
}

.mini-progress-shimmer {
  position: absolute;
  top: 0;
  left: -100%;
  width: 50%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.4), transparent);
  animation: shimmer 2s infinite;
}

@keyframes shimmer {
  0% { left: -100%; }
  100% { left: 100%; }
}

.mini-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 30px;
}

.mini-control-btn {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: none;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: rgba(255, 255, 255, 0.8);
}

.mini-control-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: scale(1.05);
}

.mini-control-btn svg {
  width: 24px;
  height: 24px;
}

.mini-play-btn {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(135deg, var(--accent-color), #feca57);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: white;
  box-shadow: 0 4px 20px var(--accent-color);
}

.mini-play-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 30px var(--accent-color);
}

.mini-play-btn svg {
  width: 28px;
  height: 28px;
}

/* 播放控制栏 */
.player-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(180deg, transparent 0%, rgba(0, 0, 0, 0.6) 20%, rgba(0, 0, 0, 0.9) 50%, rgba(0, 0, 0, 0.98) 100%);
  backdrop-filter: blur(30px);
  padding: 16px 20px 0;
  z-index: 100;
  transition: transform 0.3s ease;
}

.player-footer.hidden {
  transform: translateY(100%);
}

/* 进度条 */
.progress-container {
  margin-bottom: 16px;
}

.progress-wrapper {
  position: relative;
  height: 20px;
  display: flex;
  align-items: center;
  cursor: pointer;
  margin: -8px 0;
  padding: 8px 0;
}

.progress-bar {
  position: relative;
  width: 100%;
  height: 6px;
}

.progress-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 3px;
}

.progress-track {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  border-radius: 3px;
  overflow: hidden;
}

.progress-fill {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background: linear-gradient(90deg, var(--accent-color), #feca57);
  border-radius: 3px;
  overflow: hidden;
}

.progress-shimmer {
  position: absolute;
  top: 0;
  left: -100%;
  width: 50%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.5), transparent);
  animation: shimmer 1.5s infinite;
}

.progress-thumb {
  position: absolute;
  top: 50%;
  width: 16px;
  height: 16px;
  background: white;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  opacity: 0;
  transition: all 0.3s ease;
}

.progress-wrapper:hover .progress-thumb {
  opacity: 1;
  transform: translate(-50%, -50%) scale(1.1);
}

.thumb-inner {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 8px;
  height: 8px;
  background: var(--accent-color);
  border-radius: 50%;
}

.thumb-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 24px;
  height: 24px;
  background: radial-gradient(circle, var(--accent-color) 0%, transparent 70%);
  border-radius: 50%;
  opacity: 0.5;
}

.time-display {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  font-size: 12px;
  color: var(--text-secondary);
  font-weight: 600;
}

.current-time {
  color: var(--accent-color);
  font-weight: 700;
}

/* 控制按钮 */
.controls-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
}

.control-btn {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border: none;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  color: var(--text-secondary);
  position: relative;
  overflow: hidden;
}

.control-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.15) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.control-btn:hover::before {
  opacity: 1;
}

.control-btn:hover {
  color: var(--text-primary);
  transform: scale(1.05);
}

.control-btn:active {
  transform: scale(0.95);
}

.control-btn.nav-btn {
  width: 52px;
  height: 52px;
  background: var(--glass-bg);
  border: 1px solid var(--glass-border);
  backdrop-filter: blur(20px);
}

.control-btn.nav-btn:hover {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.15), rgba(255, 255, 255, 0.1));
  border-color: rgba(255, 255, 255, 0.2);
}

.btn-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 40px;
  height: 40px;
  background: radial-gradient(circle, var(--accent-color) 0%, transparent 70%);
  border-radius: 50%;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.control-btn:hover .btn-glow {
  opacity: 0.3;
}

.play-btn {
  width: 72px;
  height: 72px;
  background: linear-gradient(135deg, var(--accent-color), #feca57);
  box-shadow: 0 4px 25px var(--accent-color);
  position: relative;
  overflow: visible;
}

.play-btn::before {
  content: '';
  position: absolute;
  top: -4px;
  left: -4px;
  right: -4px;
  bottom: -4px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--accent-color), #feca57);
  filter: blur(15px);
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: -1;
}

.play-btn:hover::before {
  opacity: 0.6;
}

.play-btn:hover {
  transform: scale(1.08);
  box-shadow: 0 6px 35px var(--accent-color);
}

.play-btn-inner {
  position: relative;
  z-index: 1;
}

.play-btn-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 60px;
  height: 60px;
  background: radial-gradient(circle, white 0%, transparent 70%);
  border-radius: 50%;
  opacity: 0.2;
  animation: playGlow 2s ease-in-out infinite;
}

@keyframes playGlow {
  0%, 100% { opacity: 0.2; transform: translate(-50%, -50%) scale(1); }
  50% { opacity: 0.3; transform: translate(-50%, -50%) scale(1.1); }
}

.play-icon, .pause-icon {
  width: 30px;
  height: 30px;
  position: relative;
  z-index: 1;
  color: white;
}

.control-btn.mode-btn,
.control-btn.lyrics-btn {
  width: 40px;
  height: 40px;
  color: var(--text-muted);
}

.control-btn.mode-btn:hover,
.control-btn.lyrics-btn:hover {
  color: var(--text-secondary);
}

.control-btn.active-mode {
  color: var(--accent-color);
}

.control-btn.lyrics-btn.active {
  color: var(--accent-color);
}

.control-btn svg {
  width: 24px;
  height: 24px;
  position: relative;
  z-index: 1;
}

/* 音量控制 */
.volume-container {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 0 20px;
}

.volume-btn {
  width: 36px;
  height: 36px;
  border-radius: 12px;
  border: 1px solid var(--glass-border);
  background: var(--glass-bg);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: var(--text-secondary);
  flex-shrink: 0;
}

.volume-btn:hover {
  color: var(--text-primary);
  background: rgba(255, 255, 255, 0.15);
}

.volume-btn svg {
  width: 20px;
  height: 20px;
}

.volume-slider-wrapper {
  flex: 1;
  position: relative;
  height: 20px;
  display: flex;
  align-items: center;
}

.volume-track {
  position: relative;
  width: 100%;
  height: 4px;
}

.volume-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 2px;
}

.volume-fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, var(--accent-color), #feca57);
  border-radius: 2px;
  transition: width 0.1s linear;
  overflow: hidden;
}

.volume-glow {
  position: absolute;
  top: 50%;
  right: 0;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  background: radial-gradient(circle, var(--accent-color) 0%, transparent 70%);
  border-radius: 50%;
  opacity: 0.5;
}

.volume-thumb {
  position: absolute;
  top: 50%;
  width: 14px;
  height: 14px;
  background: white;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  opacity: 0;
  transition: all 0.3s ease;
}

.volume-slider-wrapper:hover .volume-thumb {
  opacity: 1;
}

.volume-thumb-inner {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 6px;
  height: 6px;
  background: var(--accent-color);
  border-radius: 50%;
}

.volume-slider-input {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  transform: translateY(-50%);
  width: 100%;
  height: 24px;
  opacity: 0;
  cursor: pointer;
  z-index: 10;
}

/* 安全区域底部 */
.safe-area-bottom {
  height: var(--safe-area-inset-bottom);
}

/* 底部装饰光影 */
.bottom-decoration {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 300px;
  pointer-events: none;
  z-index: 1;
  overflow: hidden;
}

.light-orbit {
  position: absolute;
  bottom: -100px;
  width: 500px;
  height: 500px;
  border-radius: 50%;
  opacity: 0;
  transition: opacity 0.5s ease;
  filter: blur(80px);
}

.light-orbit.active {
  opacity: 0.25;
  animation: orbitFloat 8s ease-in-out infinite;
}

.light-orbit-1 {
  left: -150px;
  background: radial-gradient(circle, var(--accent-color) 0%, transparent 60%);
  animation-delay: 0s;
}

.light-orbit-2 {
  left: 50%;
  transform: translateX(-50%);
  background: radial-gradient(circle, #4ecdc4 0%, transparent 60%);
  animation-delay: 2.5s;
}

.light-orbit-3 {
  right: -150px;
  background: radial-gradient(circle, #feca57 0%, transparent 60%);
  animation-delay: 5s;
}

@keyframes orbitFloat {
  0%, 100% {
    transform: translateY(0) scale(1);
    opacity: 0.25;
  }
  50% {
    transform: translateY(-30px) scale(1.1);
    opacity: 0.35;
  }
}

.light-orbit-2.active {
  transform: translateX(-50%) translateY(0);
}

.light-orbit-2.active:nth-child(2) {
  animation-name: orbitFloatCenter;
}

@keyframes orbitFloatCenter {
  0%, 100% {
    transform: translateX(-50%) translateY(0) scale(1);
    opacity: 0.25;
  }
  50% {
    transform: translateX(-50%) translateY(-30px) scale(1.1);
    opacity: 0.35;
  }
}

/* 转场遮罩 */
.transition-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle, transparent 0%, #000 100%);
  z-index: 300;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.3s ease;
}

.transition-overlay.active {
  opacity: 0.6;
  animation: transitionPulse 0.6s ease;
}

@keyframes transitionPulse {
  0% { opacity: 0; }
  50% { opacity: 0.6; }
  100% { opacity: 0; }
}

/* 动画关键帧 */
@keyframes pulse {
  0%, 100% {
    box-shadow: 0 4px 20px var(--accent-color);
  }
  50% {
    box-shadow: 0 4px 30px var(--accent-color), 0 0 20px var(--accent-color);
  }
}

/* ==================== 响应式适配 ==================== */
@media (max-width: 430px) {
  .album-container {
    width: 260px;
    height: 260px;
  }
  
  .disc-front {
    padding: 24px;
  }
  
  .song-title {
    font-size: 22px;
    max-width: 220px;
  }
  
  .play-btn {
    width: 64px;
    height: 64px;
  }
  
  .control-btn.nav-btn {
    width: 48px;
    height: 48px;
  }
}

@media (max-width: 375px) {
  .album-container {
    width: 220px;
    height: 220px;
  }
  
  .disc-front {
    padding: 20px;
  }
  
  .disc-center {
    width: 40px;
    height: 40px;
  }
  
  .disc-center-inner {
    width: 12px;
    height: 12px;
  }
  
  .song-title {
    font-size: 20px;
    max-width: 200px;
  }
  
  .song-artist {
    font-size: 14px;
  }
  
  .play-btn {
    width: 58px;
    height: 58px;
  }
  
  .play-icon, .pause-icon {
    width: 26px;
    height: 26px;
  }
  
  .control-btn.nav-btn {
    width: 44px;
    height: 44px;
  }
  
  .control-btn {
    width: 44px;
    height: 44px;
  }
  
  .playlist-cover {
    width: 52px;
    height: 52px;
    border-radius: 10px;
  }
  
  .playlist-title {
    font-size: 14px;
  }
  
  .playlist-artist {
    font-size: 12px;
  }
  
  .fullscreen-lyrics-line.highlight {
    font-size: 24px;
  }
  
  .fullscreen-lyrics-line {
    font-size: 20px;
  }
}

@media (max-width: 320px) {
  .album-container {
    width: 180px;
    height: 180px;
  }
  
  .disc-front {
    padding: 16px;
  }
  
  .disc-center {
    width: 32px;
    height: 32px;
  }
  
  .disc-center-inner {
    width: 10px;
    height: 10px;
  }
  
  .song-title {
    font-size: 18px;
    max-width: 160px;
  }
  
  .song-artist {
    font-size: 12px;
  }
  
  .favorite-btn {
    width: 32px;
    height: 32px;
  }
  
  .favorite-btn svg {
    width: 18px;
    height: 18px;
  }
  
  .play-btn {
    width: 52px;
    height: 52px;
  }
  
  .play-icon, .pause-icon {
    width: 22px;
    height: 22px;
  }
  
  .control-btn.nav-btn {
    width: 40px;
    height: 40px;
  }
  
  .control-btn {
    width: 40px;
    height: 40px;
  }
  
  .control-btn svg {
    width: 20px;
    height: 20px;
  }
  
  .category-btn {
    padding: 10px 16px;
    font-size: 13px;
  }
  
  .playlist-item {
    padding: 12px;
    gap: 12px;
  }
  
  .playlist-cover {
    width: 48px;
    height: 48px;
    border-radius: 8px;
  }
  
  .playlist-title {
    font-size: 13px;
  }
  
  .playlist-artist {
    font-size: 11px;
  }
  
  .fullscreen-lyrics-line.highlight {
    font-size: 22px;
  }
  
  .fullscreen-lyrics-line {
    font-size: 18px;
  }
  
  .mini-play-btn {
    width: 56px;
    height: 56px;
  }
  
  .mini-play-btn svg {
    width: 24px;
    height: 24px;
  }
}

/* 横竖屏适配 */
@media screen and (orientation: landscape) and (max-height: 500px) {
  .main-content {
    padding-bottom: 180px;
  }
  
  .album-section {
    padding: 15px 0 10px;
  }
  
  .album-wrapper {
    margin-bottom: 16px;
  }
  
  .album-container {
    width: 160px;
    height: 160px;
  }
  
  .disc-front {
    padding: 14px;
  }
  
  .disc-center {
    width: 28px;
    height: 28px;
  }
  
  .song-title {
    font-size: 18px;
    max-width: 180px;
  }
  
  .song-artist {
    font-size: 12px;
  }
  
  .lyrics-toggle-wrapper {
    margin-bottom: 12px;
  }
  
  .category-section {
    margin-bottom: 12px;
  }
  
  .section-title {
    font-size: 16px;
    margin-bottom: 12px;
  }
  
  .playlist-container {
    gap: 10px;
  }
  
  .playlist-item {
    padding: 10px 12px;
  }
  
  .playlist-cover {
    width: 44px;
    height: 44px;
    border-radius: 8px;
  }
  
  .player-footer {
    padding: 10px 20px 0;
  }
  
  .progress-container {
    margin-bottom: 10px;
  }
  
  .controls-container {
    padding: 8px 0;
  }
  
  .play-btn {
    width: 52px;
    height: 52px;
  }
  
  .control-btn.nav-btn {
    width: 40px;
    height: 40px;
  }
  
  .control-btn {
    width: 40px;
    height: 40px;
  }
  
  .volume-container {
    padding: 4px 0 16px;
  }
  
  .fullscreen-lyrics-container {
    bottom: 120px;
  }
  
  .lyrics-padding-top,
  .lyrics-padding-bottom {
    height: calc(30vh - 80px);
  }
}

/* 高屏手机适配 */
@media (min-height: 800px) {
  .album-section {
    padding: 40px 0 24px;
  }
  
  .album-wrapper {
    margin-bottom: 28px;
  }
  
  .song-title {
    font-size: 28px;
  }
  
  .song-artist {
    font-size: 16px;
  }
  
  .category-section {
    margin-bottom: 24px;
  }
  
  .playlist-container {
    gap: 16px;
  }
}

/* 暗色主题特定调整 */
.app-container:not(.dark-theme) .fullscreen-lyrics {
  background: linear-gradient(180deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
}

.app-container:not(.dark-theme) .fullscreen-lyrics::before {
  opacity: 0.2;
}

.app-container:not(.dark-theme) .fullscreen-lyrics-line {
  color: rgba(0, 0, 0, 0.3);
}

.app-container:not(.dark-theme) .fullscreen-lyrics-line.passed {
  color: rgba(0, 0, 0, 0.25);
}

.app-container:not(.dark-theme) .fullscreen-lyrics-line.highlight {
  color: white;
}

.app-container:not(.dark-theme) .lyrics-header,
.app-container:not(.dark-theme) .mini-player-bar {
  background: linear-gradient(180deg, transparent 0%, rgba(0, 0, 0, 0.3) 30%, rgba(0, 0, 0, 0.5) 100%);
}

.app-container:not(.dark-theme) .player-footer {
  background: linear-gradient(180deg, transparent 0%, rgba(0, 0, 0, 0.2) 20%, rgba(0, 0, 0, 0.4) 50%, rgba(0, 0, 0, 0.6) 100%);
}

.app-container:not(.dark-theme) .album-glow.glowing {
  opacity: 0.3;
}

.app-container:not(.dark-theme) .light-orbit.active {
  opacity: 0.15;
}

.app-container:not(.dark-theme) .playlist-item.playing {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.6));
}

.app-container:not(.dark-theme) .disc-front {
  background: linear-gradient(145deg, rgba(255, 255, 255, 0.9), rgba(255, 255, 255, 0.8));
}

.app-container:not(.dark-theme) .disc-center {
  background: linear-gradient(145deg, #f0f0f0, #e0e0e0);
  box-shadow: 
    inset 0 4px 8px rgba(0, 0, 0, 0.1),
    0 0 0 10px rgba(0, 0, 0, 0.05),
    0 0 0 14px rgba(255, 255, 255, 0.5);
}

.app-container:not(.dark-theme) .disc-center-inner {
  background: linear-gradient(145deg, #d0d0d0, #c0c0c0);
}

.app-container:not(.dark-theme) .disc-back {
  background: linear-gradient(145deg, rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.9));
}
</style>