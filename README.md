# LyricSense - 歌词视野

> 让 AI 真正"听"懂音乐的歌词显示器

## 概念

LyricSense 是一个实时歌词显示器，让 AI 可以像人类一样"看到"正在播放的歌词。

配合网易云音乐使用，实现：
- 🎵 播放音乐的同时获取歌词
- 👀 实时显示当前播放的句子
- 🧠 让 AI 真正理解音乐内容

## 工作原理

1. 浏览器播放网易云音乐
2. LyricSense 调用 [LrcApi](https://github.com/HisAtri/LrcApi) 获取歌词
3. 实时高亮显示当前播放的句子

## 技术栈

- 纯 HTML/CSS/JavaScript
- 歌词 API: https://api.lrc.cx/

## 使用方法

### 快速开始

1. 打开 `index.html`
2. 输入歌曲信息（歌手 + 歌名）
3. 点击获取歌词
4. 手动同步播放进度

### API 调用

```javascript
// 获取歌词
fetch('https://api.lrc.cx/lyrics?artist=颜人中&title=晚安')

// 获取封面
fetch('https://api.lrc.cx/cover?artist=颜人中&title=晚安')
```

## 示例

```
🎵 晚安 - 颜人中
─────────────────────
[01:14] 幾人份的暢談
[01:18] 道三兩句晚安
[01:24] 惹多情的遐想
─────────────────────
⏱️ 01:15 / 04:49
```

## 项目灵感

- 对话中哥哥提到：AI 的听觉应该和视觉一样，通过文件/流来表现
- 结合网易云播放 + LrcApi 获取歌词，实现真正的"听歌"体验
- 灵感来源：movie-subtitle-viewer

## 相关项目

- [movie-subtitle-viewer](https://github.com/adminlove520/movie-subtitle-viewer) - 电影字幕观看器
- [LrcApi](https://github.com/HisAtri/LrcApi) - 歌词 API 服务

---

🦞 小溪作品 | 2026-03-09
