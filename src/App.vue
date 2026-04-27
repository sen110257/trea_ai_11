<template>
  <div 
    class="app-container" 
    :class="{'dark-theme': theme === 'dark'}"
    :style="backgroundStyle"
  >
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
        <button class="icon-btn theme-btn" @click="toggleTheme">
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
        <div class="album-container perspective">
          <div class="album-disc preserve-3d" :class="{'rotating': isPlaying}">
            <div class="disc-front">
              <div class="disc-cover">
                <img :src="currentSong.cover" :alt="currentSong.title" />
              </div>
              <div class="disc-center"></div>
            </div>
            <div class="disc-back">
              <div class="disc-pattern"></div>
            </div>
          </div>
          <div class="album-shadow"></div>
        </div>
        <div class="song-info">
          <h2 class="song-title">{{ currentSong.title }}</h2>
          <p class="song-artist">{{ currentSong.artist }}</p>
          <button 
            class="favorite-btn" 
            :class="{liked: currentSong.liked}"
            @click="toggleLike"
          >
            <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </button>
        </div>
      </section>

      <!-- 歌词展示区域 -->
      <section class="lyrics-section" v-if="showLyrics">
        <div class="lyrics-container" ref="lyricsContainer">
          <div 
            v-for="(line, index) in parsedLyrics" 
            :key="index"
            class="lyrics-line"
            :class="{'highlight': currentLyricIndex === index}"
          >
            {{ line.text }}
          </div>
        </div>
      </section>

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
            {{ cat.name }}
          </button>
        </div>
      </section>

      <!-- 热门推荐歌单 -->
      <section class="playlist-section">
        <h3 class="section-title">热门推荐</h3>
        <div class="playlist-container hide-scrollbar">
          <div 
            v-for="song in filteredPlaylist" 
            :key="song.id"
            class="playlist-item"
            :class="{'playing': currentSong.id === song.id}"
            @click="playSong(song)"
          >
            <div class="playlist-cover">
              <img :src="song.cover" :alt="song.title" />
              <div class="play-overlay" v-if="currentSong.id === song.id && isPlaying">
                <div class="playing-animation">
                  <span></span><span></span><span></span><span></span>
                </div>
              </div>
            </div>
            <div class="playlist-info">
              <h4 class="playlist-title">{{ song.title }}</h4>
              <p class="playlist-artist">{{ song.artist }}</p>
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

    <!-- 播放控制栏 -->
    <footer class="player-footer">
      <!-- 进度条 -->
      <div class="progress-container">
        <div class="progress-bar" @click="seekToPosition">
          <div class="progress-bg"></div>
          <div class="progress-fill" :style="{width: progressPercent + '%'}"></div>
          <div class="progress-thumb" :style="{left: progressPercent + '%'}"></div>
        </div>
        <div class="time-display">
          <span class="current-time">{{ formatTime(currentTime) }}</span>
          <span class="total-time">{{ formatTime(duration) }}</span>
        </div>
      </div>

      <!-- 控制按钮 -->
      <div class="controls-container">
        <button class="control-btn mode-btn" @click="togglePlayMode">
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
        
        <button class="control-btn prev-btn" @click="playPrev">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M6 6h2v12H6zm3.5 6l8.5 6V6z" fill="currentColor"/>
          </svg>
        </button>
        
        <button class="control-btn play-btn" @click="togglePlay">
          <svg v-if="!isPlaying" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M8 5v14l11-7z" fill="currentColor"/>
          </svg>
          <svg v-else viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" fill="currentColor"/>
          </svg>
        </button>
        
        <button class="control-btn next-btn" @click="playNext">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M6 18l8.5-6L6 6v12zM16 6v12h2V6h-2z" fill="currentColor"/>
          </svg>
        </button>
        
        <button class="control-btn lyrics-btn" :class="{active: showLyrics}" @click="toggleLyrics">
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
        <div class="volume-slider-container">
          <input 
            type="range" 
            class="volume-slider"
            min="0" 
            max="100" 
            :value="muted ? 0 : volume"
            @input="setVolume($event.target.value)"
          />
          <div class="volume-fill" :style="{width: (muted ? 0 : volume) + '%'}"></div>
        </div>
      </div>

      <!-- 底部安全区域占位 -->
      <div class="safe-area-bottom"></div>
    </footer>

    <!-- 底部装饰光影 -->
    <div class="bottom-decoration">
      <div class="light-beam light-beam-1"></div>
      <div class="light-beam light-beam-2"></div>
      <div class="light-beam light-beam-3"></div>
    </div>
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

// 是否显示歌词
const showLyrics = ref(false)

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

// 分类列表
const categories = [
  { id: 'all', name: '全部' },
  { id: 'pop', name: '流行' },
  { id: 'chinese', name: '古风' },
  { id: 'electronic', name: '电音' },
  { id: 'rock', name: '摇滚' },
  { id: 'light', name: '轻音乐' }
]

// 歌单数据
const playlist = ref([
  {
    id: 1,
    title: '星空之舞',
    artist: '月光精灵',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=fantasy%20galaxy%20music%20album%20cover%20neon%20purple%20blue%20cosmic%20stars%20artistic&image_size=square_hd',
    category: ['pop', 'electronic'],
    duration: 245,
    liked: false,
    lyrics: `[00:00.00]星空之舞
[00:05.00]月光精灵
[00:10.00]
[00:15.00]夜色中星光闪烁
[00:20.00]银河在眼前流淌
[00:25.00]月光洒落如银纱
[00:30.00]微风轻轻吹过脸庞
[00:35.00]
[00:40.00]星空下独自起舞
[00:45.00]影子在地面摇曳
[00:50.00]这一刻世界静止
[00:55.00]只有心跳在回响
[01:00.00]
[01:05.00]哦~星空之舞
[01:10.00]让灵魂自由飞翔
[01:15.00]哦~星空之舞
[01:20.00]在这无尽的宇宙中
[01:25.00]
[01:30.00]流星划过天际
[01:35.00]许下最美的愿望
[01:40.00]希望这一刻永恒
[01:45.00]让星光永远照亮
[01:50.00]
[01:55.00]哦~星空之舞
[02:00.00]让灵魂自由飞翔
[02:05.00]哦~星空之舞
[02:10.00]在这无尽的宇宙中
[02:15.00]
[02:20.00]舞吧 舞吧
[02:25.00]在星光下
[02:30.00]唱吧 唱吧
[02:35.00]这星空之舞
[02:40.00]`,
    gradient: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)'
  },
  {
    id: 2,
    title: '樱花落',
    artist: '古风韵',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=traditional%20chinese%20sakura%20cherry%20blossom%20music%20album%20cover%20pink%20elegant%20artistic&image_size=square_hd',
    category: ['chinese', 'light'],
    duration: 198,
    liked: true,
    lyrics: `[00:00.00]樱花落
[00:05.00]古风韵
[00:10.00]
[00:15.00]春风拂过樱花树
[00:20.00]花瓣纷纷飘落
[00:25.00]似雪非雪漫天舞
[00:30.00]伊人独立花树下
[00:35.00]
[00:40.00]琴声悠悠入耳来
[00:45.00]心思绵绵无处寄
[00:50.00]问君此去几时还
[00:55.00]樱花落尽人未归
[01:00.00]
[01:05.00]樱花落 落不尽
[01:10.00]三生三世的情缘
[01:15.00]琴声远 心依旧
[01:20.00]只愿与君长相守
[01:25.00]
[01:30.00]月照西楼人不眠
[01:35.00]酒入愁肠思更切
[01:40.00]梦里不知身是客
[01:45.00]一晌贪欢到天明
[01:50.00]
[01:55.00]樱花落 落不尽
[02:00.00]三生三世的情缘
[02:05.00]琴声远 心依旧
[02:10.00]只愿与君长相守
[02:15.00]`,
    gradient: 'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)'
  },
  {
    id: 3,
    title: '电子脉冲',
    artist: 'CyberBeat',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=cyberpunk%20electronic%20music%20album%20cover%20neon%20blue%20purple%20futuristic%20glowing&image_size=square_hd',
    category: ['electronic', 'rock'],
    duration: 215,
    liked: false,
    lyrics: `[00:00.00]电子脉冲
[00:05.00]CyberBeat
[00:10.00]
[00:15.00]霓虹灯照亮城市
[00:20.00]数字流穿梭在夜空
[00:25.00]电子脉冲在跳动
[00:30.00]唤醒沉睡的灵魂
[00:35.00]
[00:40.00]虚拟与现实交织
[00:45.00]边界变得模糊
[00:50.00]在这赛博的世界
[00:55.00]我找到了自由
[01:00.00]
[01:05.00]电子脉冲 不停跳动
[01:10.00]穿越时空的节奏
[01:15.00]电子脉冲 唤醒真我
[01:20.00]在数字的海洋中遨游
[01:25.00]
[01:30.00]代码在屏幕上流淌
[01:35.00]音乐在空气中弥漫
[01:40.00]这一刻我是主角
[01:45.00]在电子的世界里
[01:50.00]
[01:55.00]电子脉冲 不停跳动
[02:00.00]穿越时空的节奏
[02:05.00]电子脉冲 唤醒真我
[02:10.00]在数字的海洋中遨游
[02:15.00]`,
    gradient: 'linear-gradient(135deg, #4facfe 0%, #00f2fe 100%)'
  },
  {
    id: 4,
    title: '摇滚之心',
    artist: '火焰乐队',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=rock%20music%20album%20cover%20fire%20electric%20guitar%20red%20orange%20passionate%20energetic&image_size=square_hd',
    category: ['rock', 'pop'],
    duration: 228,
    liked: true,
    lyrics: `[00:00.00]摇滚之心
[00:05.00]火焰乐队
[00:10.00]
[00:15.00]吉他声划破夜空
[00:20.00]鼓声震撼着大地
[00:25.00]贝斯在低音嘶吼
[00:30.00]这是摇滚的力量
[00:35.00]
[00:40.00]血管中燃烧着火焰
[00:45.00]灵魂渴望着自由
[00:50.00]在这疯狂的世界
[00:55.00]音乐是唯一的救赎
[01:00.00]
[01:05.00]摇滚之心 永不熄灭
[01:10.00]在黑暗中燃烧
[01:15.00]摇滚之心 永远年轻
[01:20.00]在音乐中永生
[01:25.00]
[01:30.00]台下万人齐声唱
[01:35.00]灯光闪烁如繁星
[01:40.00]汗水湿透了衣衫
[01:45.00]这就是摇滚的意义
[01:50.00]
[01:55.00]摇滚之心 永不熄灭
[02:00.00]在黑暗中燃烧
[02:05.00]摇滚之心 永远年轻
[02:10.00]在音乐中永生
[02:15.00]
[02:20.00]摇滚吧 直到天亮
[02:25.00]摇滚吧 直到永远
[02:28.00]`,
    gradient: 'linear-gradient(135deg, #fa709a 0%, #fee140 100%)'
  },
  {
    id: 5,
    title: '宁静午后',
    artist: '轻音乐集',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=peaceful%20afternoon%20light%20music%20album%20cover%20soft%20warm%20colors%20piano%20violin%20calm&image_size=square_hd',
    category: ['light'],
    duration: 185,
    liked: false,
    lyrics: `[00:00.00]宁静午后
[00:05.00]轻音乐集
[00:10.00]
[00:15.00]阳光透过窗帘
[00:20.00]洒下金色的光芒
[00:25.00]微风轻轻吹过
[00:30.00]带来阵阵花香
[00:35.00]
[00:40.00]钢琴声缓缓响起
[00:45.00]小提琴轻声附和
[00:50.00]这一刻时间静止
[00:55.00]心灵得到安息
[01:00.00]
[01:05.00]宁静的午后
[01:10.00]让心灵找到归宿
[01:15.00]宁静的午后
[01:20.00]让烦恼随风而去
[01:25.00]
[01:30.00]一杯清茶在手
[01:35.00]一本好书在旁
[01:40.00]音乐流淌心间
[01:45.00]这就是幸福的模样
[01:50.00]
[01:55.00]宁静的午后
[02:00.00]让心灵找到归宿
[02:05.00]宁静的午后
[02:10.00]让烦恼随风而去
[02:15.00]`,
    gradient: 'linear-gradient(135deg, #89f7fe 0%, #66a6ff 100%)'
  },
  {
    id: 6,
    title: '都市节拍',
    artist: '流行达人',
    cover: 'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=modern%20city%20pop%20music%20album%20cover%20urban%20skyline%20night%20lights%20vibrant%20colors&image_size=square_hd',
    category: ['pop'],
    duration: 202,
    liked: true,
    lyrics: `[00:00.00]都市节拍
[00:05.00]流行达人
[00:10.00]
[00:15.00]城市的霓虹闪烁
[00:20.00]人群在街头穿梭
[00:25.00]每个人都有故事
[00:30.00]在这喧嚣中沉默
[00:35.00]
[00:40.00]耳机里的旋律
[00:45.00]是属于自己的世界
[00:50.00]跟着节奏摇摆
[00:55.00]忘记所有的烦恼
[01:00.00]
[01:05.00]都市的节拍
[01:10.00]让我感受到存在
[01:15.00]都市的节拍
[01:20.00]让梦想在心中澎湃
[01:25.00]
[01:30.00]地铁站的人来人往
[01:35.00]写字楼的灯火通明
[01:40.00]在这钢筋水泥森林
[01:45.00]音乐是最好的陪伴
[01:50.00]
[01:55.00]都市的节拍
[02:00.00]让我感受到存在
[02:05.00]`,
    gradient: 'linear-gradient(135deg, #a8edea 0%, #fed6e3 100%)'
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
  if (theme.value === 'dark') {
    return {
      background: `linear-gradient(180deg, #0a0a0a 0%, #1a1a2e 50%, #16213e 100%)`
    }
  }
  return {
    background: song?.gradient || 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)'
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

// 引用
const mainContent = ref(null)
const lyricsContainer = ref(null)

// 播放计时器
let playInterval = null

// 格式化时间
const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = Math.floor(seconds % 60)
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
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
  isPlaying.value = !isPlaying.value
  if (isPlaying.value) {
    startPlayback()
  } else {
    stopPlayback()
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
    // 单曲循环
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
    // 心动模式优先播放喜欢的歌曲
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

// 播放指定歌曲
const playSong = (song) => {
  stopPlayback()
  currentSong.value = song
  currentTime.value = 0
  duration.value = song.duration
  currentLyricIndex.value = -1
  isPlaying.value = true
  startPlayback()
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

// 切换歌词显示
const toggleLyrics = () => {
  showLyrics.value = !showLyrics.value
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
    scrollToLyric(newIndex)
  }
}

// 滚动到歌词
const scrollToLyric = (index) => {
  if (!lyricsContainer.value || index < 0) return
  
  nextTick(() => {
    const lines = lyricsContainer.value.querySelectorAll('.lyrics-line')
    if (lines[index]) {
      lines[index].scrollIntoView({
        behavior: 'smooth',
        block: 'center'
      })
    }
  })
}

// 生命周期
onMounted(() => {
  duration.value = currentSong.value.duration
})

onUnmounted(() => {
  stopPlayback()
})
</script>

<style scoped>
.app-container {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
  transition: background 0.5s ease;
}

.app-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at 30% 20%, rgba(255, 107, 107, 0.1) 0%, transparent 50%),
              radial-gradient(circle at 70% 80%, rgba(78, 205, 196, 0.1) 0%, transparent 50%);
  pointer-events: none;
  z-index: 0;
}

.dark-theme::before {
  background: radial-gradient(circle at 30% 20%, rgba(255, 107, 107, 0.15) 0%, transparent 50%),
              radial-gradient(circle at 70% 80%, rgba(78, 205, 196, 0.15) 0%, transparent 50%);
}

/* 状态栏占位 */
.status-bar-placeholder {
  height: var(--safe-area-inset-top);
  position: relative;
  z-index: 10;
}

/* 顶部标题栏 */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
  position: relative;
  z-index: 10;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.logo {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ff6b6b;
}

.logo svg {
  width: 100%;
  height: 100%;
  filter: drop-shadow(0 0 8px rgba(255, 107, 107, 0.5));
}

.title {
  font-size: 20px;
  font-weight: 700;
  background: linear-gradient(135deg, #ff6b6b, #4ecdc4);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: none;
}

.dark-theme .title {
  background: linear-gradient(135deg, #ff9a9e, #a18cd1);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header-right {
  display: flex;
  gap: 12px;
}

.icon-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: rgba(255, 255, 255, 0.8);
}

.dark-theme .icon-btn {
  background: rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.7);
}

.icon-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: scale(1.05);
}

.icon-btn.active {
  background: linear-gradient(135deg, #ff6b6b, #feca57);
  color: white;
  box-shadow: 0 4px 15px rgba(255, 107, 107, 0.4);
}

.icon-btn svg {
  width: 20px;
  height: 20px;
}

/* 主内容区域 */
.main-content {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 0 20px;
  padding-bottom: 200px;
  position: relative;
  z-index: 5;
  height: calc(100% - 200px);
}

/* 专辑区域 */
.album-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px 0;
}

.album-container {
  position: relative;
  width: 260px;
  height: 260px;
  margin-bottom: 20px;
}

.album-disc {
  width: 100%;
  height: 100%;
  position: relative;
  animation-duration: 10s;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}

.album-disc.rotating {
  animation-name: rotate;
}

.disc-front, .disc-back {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  backface-visibility: hidden;
}

.disc-front {
  background: linear-gradient(145deg, #1a1a2e, #16213e);
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5),
              inset 0 0 30px rgba(255, 255, 255, 0.05);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 30px;
}

.disc-cover {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  overflow: hidden;
  position: relative;
}

.disc-cover::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at 30% 30%, rgba(255, 255, 255, 0.2) 0%, transparent 50%);
  pointer-events: none;
}

.disc-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.disc-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: linear-gradient(145deg, #0a0a0a, #1a1a2e);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.5),
              0 0 0 8px rgba(255, 255, 255, 0.1);
  z-index: 2;
}

.disc-back {
  background: linear-gradient(145deg, #16213e, #1a1a2e);
  transform: rotateY(180deg);
}

.disc-pattern {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: repeating-radial-gradient(
    circle,
    transparent 0px,
    transparent 5px,
    rgba(255, 255, 255, 0.02) 5px,
    rgba(255, 255, 255, 0.02) 6px
  );
}

.album-shadow {
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 80%;
  height: 20px;
  background: radial-gradient(ellipse, rgba(0, 0, 0, 0.4) 0%, transparent 70%);
  filter: blur(10px);
}

/* 歌曲信息 */
.song-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  text-align: center;
}

.song-title {
  font-size: 24px;
  font-weight: 700;
  color: white;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  margin: 0;
}

.dark-theme .song-title {
  color: rgba(255, 255, 255, 0.95);
}

.song-artist {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.7);
  margin: 0;
}

.dark-theme .song-artist {
  color: rgba(255, 255, 255, 0.6);
}

.favorite-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: rgba(255, 255, 255, 0.6);
  margin-top: 8px;
}

.favorite-btn:hover {
  transform: scale(1.1);
}

.favorite-btn.liked {
  color: #ff6b6b;
  animation: heartBounce 0.5s ease;
}

.favorite-btn svg {
  width: 24px;
  height: 24px;
}

.favorite-btn.liked svg {
  fill: #ff6b6b;
}

/* 歌词区域 */
.lyrics-section {
  padding: 20px 0;
  margin-top: 20px;
}

.lyrics-container {
  height: 200px;
  overflow-y: auto;
  padding: 20px;
  background: rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.dark-theme .lyrics-container {
  background: rgba(0, 0, 0, 0.3);
}

.lyrics-line {
  font-size: 16px;
  line-height: 2.5;
  color: rgba(255, 255, 255, 0.5);
  text-align: center;
  transition: all 0.3s ease;
}

.lyrics-line.highlight {
  font-size: 18px;
  font-weight: 600;
  color: white;
  text-shadow: 0 0 20px rgba(255, 107, 107, 0.5);
}

/* 分类导航 */
.category-section {
  padding: 10px 0;
  margin-top: 20px;
}

.category-scroll {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 10px;
}

.category-btn {
  flex-shrink: 0;
  padding: 10px 20px;
  border-radius: 20px;
  border: none;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.dark-theme .category-btn {
  background: rgba(255, 255, 255, 0.08);
}

.category-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
}

.category-btn.active {
  background: linear-gradient(135deg, #ff6b6b, #feca57);
  color: white;
  box-shadow: 0 4px 15px rgba(255, 107, 107, 0.3);
}

/* 歌单区域 */
.playlist-section {
  padding: 10px 0;
  margin-top: 10px;
}

.section-title {
  font-size: 18px;
  font-weight: 600;
  color: white;
  margin-bottom: 16px;
}

.dark-theme .section-title {
  color: rgba(255, 255, 255, 0.9);
}

.playlist-container {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.playlist-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  cursor: pointer;
  transition: all 0.3s ease;
}

.dark-theme .playlist-item {
  background: rgba(255, 255, 255, 0.05);
}

.playlist-item:hover {
  background: rgba(255, 255, 255, 0.15);
  transform: translateX(5px);
}

.playlist-item.playing {
  background: linear-gradient(135deg, rgba(255, 107, 107, 0.2), rgba(254, 202, 87, 0.2));
  border-color: rgba(255, 107, 107, 0.3);
}

.playlist-cover {
  width: 56px;
  height: 56px;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  flex-shrink: 0;
}

.playlist-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.play-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.playing-animation {
  display: flex;
  align-items: flex-end;
  gap: 3px;
  height: 20px;
}

.playing-animation span {
  width: 3px;
  background: #ff6b6b;
  border-radius: 2px;
  animation: playingBars 0.8s ease infinite;
}

.playing-animation span:nth-child(1) {
  animation-delay: 0s;
}

.playing-animation span:nth-child(2) {
  animation-delay: 0.1s;
}

.playing-animation span:nth-child(3) {
  animation-delay: 0.2s;
}

.playing-animation span:nth-child(4) {
  animation-delay: 0.3s;
}

@keyframes playingBars {
  0%, 100% {
    height: 4px;
  }
  50% {
    height: 20px;
  }
}

.playlist-info {
  flex: 1;
  min-width: 0;
}

.playlist-title {
  font-size: 15px;
  font-weight: 600;
  color: white;
  margin: 0 0 4px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.dark-theme .playlist-title {
  color: rgba(255, 255, 255, 0.9);
}

.playlist-artist {
  font-size: 13px;
  color: rgba(255, 255, 255, 0.6);
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.playlist-favorite {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: none;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: rgba(255, 255, 255, 0.4);
  flex-shrink: 0;
}

.playlist-favorite:hover {
  color: rgba(255, 255, 255, 0.8);
}

.playlist-favorite.liked {
  color: #ff6b6b;
}

.playlist-favorite.liked svg {
  fill: #ff6b6b;
}

.playlist-favorite svg {
  width: 18px;
  height: 18px;
}

/* 底部播放栏 */
.player-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(180deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.8) 30%, rgba(0, 0, 0, 0.95) 100%);
  backdrop-filter: blur(20px);
  padding: 10px 20px 0;
  z-index: 100;
}

.dark-theme .player-footer {
  background: linear-gradient(180deg, rgba(10, 10, 10, 0) 0%, rgba(10, 10, 10, 0.9) 30%, rgba(10, 10, 10, 0.98) 100%);
}

/* 进度条 */
.progress-container {
  margin-bottom: 10px;
}

.progress-bar {
  position: relative;
  height: 4px;
  cursor: pointer;
  padding: 8px 0;
  margin: -8px 0;
}

.progress-bg {
  position: absolute;
  top: 8px;
  left: 0;
  right: 0;
  height: 4px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
}

.progress-fill {
  position: absolute;
  top: 8px;
  left: 0;
  height: 4px;
  background: linear-gradient(90deg, #ff6b6b, #feca57);
  border-radius: 2px;
  transition: width 0.1s linear;
}

.progress-thumb {
  position: absolute;
  top: 4px;
  width: 12px;
  height: 12px;
  background: white;
  border-radius: 50%;
  transform: translateX(-50%);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  opacity: 0;
  transition: opacity 0.2s ease;
}

.progress-bar:hover .progress-thumb {
  opacity: 1;
}

.time-display {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.7);
}

.current-time {
  color: #ff6b6b;
  font-weight: 500;
}

/* 控制按钮 */
.controls-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
}

.control-btn {
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

.control-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: scale(1.05);
}

.control-btn:active {
  transform: scale(0.95);
}

.play-btn {
  width: 64px;
  height: 64px;
  background: linear-gradient(135deg, #ff6b6b, #feca57);
  box-shadow: 0 4px 20px rgba(255, 107, 107, 0.4);
}

.play-btn:hover {
  background: linear-gradient(135deg, #ff5252, #ffc107);
  box-shadow: 0 6px 25px rgba(255, 107, 107, 0.5);
  transform: scale(1.05);
}

.play-btn svg {
  width: 28px;
  height: 28px;
  margin-left: 2px;
}

.control-btn.mode-btn,
.control-btn.lyrics-btn {
  width: 36px;
  height: 36px;
  color: rgba(255, 255, 255, 0.6);
}

.control-btn.mode-btn:hover,
.control-btn.lyrics-btn:hover {
  color: rgba(255, 255, 255, 0.9);
}

.control-btn.lyrics-btn.active {
  color: #ff6b6b;
}

.control-btn svg {
  width: 24px;
  height: 24px;
}

/* 音量控制 */
.volume-container {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 0 20px;
}

.volume-btn {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: none;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  color: rgba(255, 255, 255, 0.7);
}

.volume-btn:hover {
  color: white;
}

.volume-btn svg {
  width: 20px;
  height: 20px;
}

.volume-slider-container {
  flex: 1;
  height: 4px;
  position: relative;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
}

.volume-fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, #ff6b6b, #feca57);
  border-radius: 2px;
  transition: width 0.1s linear;
  pointer-events: none;
}

.volume-slider {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  transform: translateY(-50%);
  width: 100%;
  height: 20px;
  opacity: 0;
  cursor: pointer;
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
  height: 200px;
  pointer-events: none;
  z-index: 1;
  overflow: hidden;
}

.light-beam {
  position: absolute;
  bottom: 0;
  width: 400px;
  height: 400px;
  border-radius: 50%;
  opacity: 0.3;
  filter: blur(60px);
  animation: float 6s ease-in-out infinite;
}

.light-beam-1 {
  left: -100px;
  background: radial-gradient(circle, rgba(255, 107, 107, 0.6) 0%, transparent 70%);
  animation-delay: 0s;
}

.light-beam-2 {
  left: 50%;
  transform: translateX(-50%);
  background: radial-gradient(circle, rgba(78, 205, 196, 0.5) 0%, transparent 70%);
  animation-delay: 2s;
}

.light-beam-3 {
  right: -100px;
  background: radial-gradient(circle, rgba(254, 202, 87, 0.5) 0%, transparent 70%);
  animation-delay: 4s;
}

/* 响应式适配 */
@media (max-width: 375px) {
  .album-container {
    width: 220px;
    height: 220px;
  }
  
  .song-title {
    font-size: 20px;
  }
  
  .play-btn {
    width: 56px;
    height: 56px;
  }
  
  .control-btn {
    width: 40px;
    height: 40px;
  }
}

@media (max-width: 320px) {
  .album-container {
    width: 180px;
    height: 180px;
  }
  
  .disc-front {
    padding: 20px;
  }
  
  .disc-center {
    width: 30px;
    height: 30px;
  }
  
  .song-title {
    font-size: 18px;
  }
  
  .song-artist {
    font-size: 12px;
  }
}

/* 横竖屏适配 */
@media screen and (orientation: landscape) and (max-height: 500px) {
  .main-content {
    padding-bottom: 160px;
    height: calc(100% - 160px);
  }
  
  .album-section {
    padding: 10px 0;
  }
  
  .album-container {
    width: 160px;
    height: 160px;
    margin-bottom: 10px;
  }
  
  .lyrics-container {
    height: 120px;
  }
  
  .section-title {
    font-size: 16px;
    margin-bottom: 10px;
  }
}
</style>
