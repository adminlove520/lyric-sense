# LyricSense - 歌词视野

> 让 AI 真正「听」懂音乐的歌词显示器 🎵

[![GitHub Pages](https://img.shields.io/badge/Demo-GitHub%20Pages-blue)](https://adminlove520.github.io/lyric-sense/)
[![OpenClaw](https://img.shields.io/badge/🦞-OpenClaw%20Skill-orange)](SKILL.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](LICENSE)

## 概念

LyricSense 是一个实时歌词显示器，让 AI 可以像人类一样「看到」正在播放的歌词。

配合 **netease-daily-v3** 使用，实现：
- 🎵 播放音乐的同时获取歌词
- 👀 基于真实 LRC 时间戳的实时滚动
- 🦞 龙虾直接通过 API 读取歌词内容
- 🔗 日推歌单一键跳转歌词界面

## 功能

| 功能 | 说明 |
|------|------|
| 🔍 多源搜索 | 歌手 + 歌名，支持仅填其一 |
| 📝 LRC 解析 | 支持 `[mm:ss.xx]` 和 `[mm:ss:xx]` 两种格式 |
| ⏱️ 真实计时 | 基于 `requestAnimationFrame` + LRC 时间戳，精确同步 |
| 📊 进度条 | 按时间跳转（非按行数） |
| 🖼️ 封面获取 | 自动获取专辑封面 |
| 📋 歌词复制 | 一键复制原始 LRC 文本 |
| 🔗 URL 参数 | `?artist=&title=` 直接加载 |
| 🔁 代理轮换 | corsproxy.io / allorigins.win 自动 fallback |
| 🌐 API 切换 | 公共 / 本地 / 自定义三档 |
| ⌨️ 键盘快捷键 | 空格播放/暂停，方向键切换歌词行 |
| 📱 响应式 | 移动端适配 |

## 快速开始

### 在线使用（零配置）

直接访问：**[https://adminlove520.github.io/lyric-sense/](https://adminlove520.github.io/lyric-sense/)**

### URL 参数直接加载

```
https://adminlove520.github.io/lyric-sense/?artist=颜人中&title=晚安
```

从 **netease-daily-v3** 日推列表点击「歌词」链接，即可通过此方式直接加载。

### Fork 后部署 GitHub Pages

1. Fork 本仓库
2. Settings → Pages → Source 选 `main` 分支，目录 `/ (root)`
3. 访问 `https://你的用户名.github.io/lyric-sense/`

> 💡 Fork 后记得修改 SKILL.md 中的 demo URL

### 本地运行

```bash
git clone https://github.com/adminlove520/lyric-sense.git
cd lyric-sense
# 直接用浏览器打开（无需服务器）
open index.html      # macOS
start index.html     # Windows
xdg-open index.html  # Linux
```

## API 说明

### 公共 API（默认，无需部署）

```
歌词: GET https://api.lrc.cx/lyrics?artist={歌手}&title={歌名}
封面: GET https://api.lrc.cx/cover?artist={歌手}&title={歌名}
```

> 有请求频率限制，高频使用建议自部署

### 本地 API（无限制）

**Windows 可执行文件：**
```powershell
.\scripts\LrcApi\lrcapi-1.6.0-Windows-AMD64.exe --port 8080
```

**Docker：**
```bash
docker run -d -p 8080:8080 hisatri/lrcapi:latest
```

启动后在页面右上角将 API 源切换为「本地 localhost:8080」。

### 给龙虾（OpenClaw Agent）直接调用

```javascript
// 获取歌词（通过 CORS 代理）
async function getLyrics(artist, title) {
    const url   = `https://api.lrc.cx/lyrics?artist=${encodeURIComponent(artist)}&title=${encodeURIComponent(title)}`;
    const proxy = `https://corsproxy.io/?${encodeURIComponent(url)}`;
    const resp  = await fetch(proxy);
    return resp.text();
}

// 解析 LRC
function parseLRC(lrc) {
    return lrc.split('\n')
        .map(l => l.match(/\[(\d{2}):(\d{2})[.:]\d+\](.*)/))
        .filter(Boolean)
        .map(([, m, s, t]) => ({
            time: +m * 60 + +s,
            text: t.trim()
        }))
        .filter(l => l.text);
}

// 生成跳转链接
function lyricSenseUrl(artist, title) {
    return `https://adminlove520.github.io/lyric-sense/?artist=${encodeURIComponent(artist)}&title=${encodeURIComponent(title)}`;
}
```

完整使用示例见 [SKILL.md](SKILL.md)。

## 与 netease-daily-v3 联动

### 联动原理

```
netease-daily-v3           lyric-sense
─────────────────          ───────────────────
每日日推歌单       →  每首歌附带 lyric-sense 链接
GitHub Discussion  →  用户点击链接 → 自动加载歌词
龙虾读取日推 JSON  →  调用 LrcApi 获取歌词内容
```

### 日推输出示例（带歌词链接）

```markdown
## 🎵 网易云 · 日推 · 03月27日

| # | 歌曲 | 歌手 | 专辑 | 时长 | 歌词 |
|---|------|------|------|------|------|
| 1 | [晚安](https://music.163.com/song?id=xxx) | 颜人中 | 晚安 | 3:54 | [🎵 歌词](https://adminlove520.github.io/lyric-sense/?artist=颜人中&title=晚安) |
```

## 工作原理

```
用户输入 歌手 + 歌名
       ↓
调用 LrcApi（公共/本地/自定义）
       ↓
解析 LRC 时间戳
       ↓
requestAnimationFrame 驱动精确同步
       ↓
实时高亮 + 进度条更新
```

## 项目结构

```
lyric-sense/
├── index.html      # 主页面（全部功能）
├── SKILL.md        # OpenClaw 龙虾技能定义
├── README.md       # 本文件
├── CHANGELOG.md    # 更新日志
├── .gitignore
└── scripts/
    └── LrcApi/     # 本地歌词 API 可执行文件
```

## 键盘快捷键

| 按键 | 功能 |
|------|------|
| `Space` | 播放 / 暂停 |
| `←` | 上一句 |
| `→` | 下一句 |
| `Enter`（输入框内）| 搜索 |

## 相关项目

| 项目 | 说明 | 联动 |
|------|------|------|
| [netease-daily-v3](https://github.com/adminlove520/netease-daily-v3) | 网易云日推自动推送 | ✅ 歌词链接联动 |
| [LrcApi](https://github.com/HisAtri/LrcApi) | 歌词 API 服务 | ✅ 数据来源 |
| [movie-subtitle-viewer](https://github.com/adminlove520/movie-subtitle-viewer) | 电影字幕观看器 | 同类项目 |

---

🦞 Made by 小溪 | [OpenClaw Skill](SKILL.md) | [在线演示](https://adminlove520.github.io/lyric-sense/)
