---
name: lyric-sense
version: 2.0.0
description: >
  让 AI 通过歌词「听」音乐的 OpenClaw 技能。
  通过调用 LrcApi 搜索歌词、实时显示当前播放句子、获取封面，
  让龙虾真正「看懂」正在播放的歌曲。支持与 netease-daily-v3 联动，
  自动为日推歌单每首歌生成歌词跳转链接。
author: 小溪
homepage: https://github.com/adminlove520/lyric-sense
demo: https://adminlove520.github.io/lyric-sense
license: MIT
triggers:
  - 听歌
  - 歌词
  - 播放音乐
  - 搜索歌词
  - 显示歌词
  - lyrics
  - 查歌词
  - 这首歌歌词
  - 帮我找歌词
always_apply: false
---

# LyricSense 歌词技能

让 AI 通过歌词「听」音乐的 OpenClaw 技能。

## 功能

1. **搜索歌词** — 通过歌手 + 歌名获取 LRC 格式歌词
2. **展示歌词** — 格式化输出，高亮当前句
3. **获取封面** — 返回专辑封面图片 URL
4. **快速跳转** — 生成 LyricSense 网页界面链接，点击即可同步播放
5. **联动日推** — 与 netease-daily-v3 配合，为日推歌单每首歌添加歌词入口

## API 说明

### 公共 API（无需部署）

```
歌词: GET https://api.lrc.cx/lyrics?artist={歌手}&title={歌名}
封面: GET https://api.lrc.cx/cover?artist={歌手}&title={歌名}
```

响应格式：纯文本 LRC 格式

```
[ti:晚安]
[ar:颜人中]
[00:19.00]我站在 青春的尾巴上
[00:24.00]望着 那盏路灯发呆
```

### 本地 API（自部署，无限制）

```
歌词: GET http://localhost:8080/lyrics?artist={歌手}&title={歌名}
封面: GET http://localhost:8080/cover?artist={歌手}&title={歌名}
```

### 网页界面（带同步高亮）

```
https://adminlove520.github.io/lyric-sense/?artist={歌手}&title={歌名}
```

## 龙虾使用示例

### 示例 1：搜索并展示歌词

**用户：** 帮我查一下《晚安》颜人中的歌词

**龙虾：**
```javascript
// 调用 LrcApi 获取歌词
const resp = await fetch('https://corsproxy.io/?' + encodeURIComponent(
  'https://api.lrc.cx/lyrics?artist=颜人中&title=晚安'
));
const lrc = await resp.text();
```

返回后解析 LRC 并展示：

```
🎵 晚安 — 颜人中
─────────────────────────────
[00:19] 我站在 青春的尾巴上
[00:24] 望着 那盏路灯发呆
[00:29] 时光它 不等人 悄悄走
[00:33] 只剩我 一个人 慢慢熬
─────────────────────────────
🔗 同步界面: https://adminlove520.github.io/lyric-sense/?artist=颜人中&title=晚安
```

### 示例 2：只知道歌名

**用户：** 夜空中最亮的星 歌词给我

```javascript
const resp = await fetch('https://corsproxy.io/?' + encodeURIComponent(
  'https://api.lrc.cx/lyrics?title=夜空中最亮的星'
));
```

### 示例 3：联动 netease-daily-v3 日推

当龙虾获取到日推歌单时，为每首歌自动生成歌词链接：

```javascript
// songs 来自 netease-daily-v3 输出的歌单
const withLyrics = songs.map(song => ({
  ...song,
  lyricsUrl: `https://adminlove520.github.io/lyric-sense/?artist=${encodeURIComponent(song.artist)}&title=${encodeURIComponent(song.name)}`,
  lrcApi: `https://api.lrc.cx/lyrics?artist=${encodeURIComponent(song.artist)}&title=${encodeURIComponent(song.name)}`
}));
```

### 示例 4：格式化输出（供龙虾直接展示）

```javascript
function parseLRC(lrcText) {
  return lrcText.split('\n')
    .map(line => line.match(/\[(\d{2}):(\d{2})[.:]\d+\](.*)/))
    .filter(Boolean)
    .map(([, m, s, text]) => ({
      time: parseInt(m) * 60 + parseInt(s),
      text: text.trim()
    }))
    .filter(l => l.text);
}

function formatLyricsForAI(parsed, maxLines = 20) {
  const lines = parsed.slice(0, maxLines);
  return lines.map(l => {
    const m = Math.floor(l.time / 60).toString().padStart(2, '0');
    const s = (l.time % 60).toString().padStart(2, '0');
    return `[${m}:${s}] ${l.text}`;
  }).join('\n');
}
```

## 错误处理

| 情况 | 处理方式 |
|------|---------|
| 歌词为空 | 提示用户换关键词，或只用歌名/歌手单独搜索 |
| CORS 跨域 | 自动通过 `corsproxy.io` 代理转发 |
| API 超时 | 重试 1 次，仍失败则提示使用本地部署 |
| LRC 格式异常 | 尝试按行分割，过滤无效行 |

## 部署方式

### 在线（默认，零配置）

直接使用公共 API `https://api.lrc.cx`，无需任何部署。

### 本地可执行文件（Windows）

```powershell
.\scripts\LrcApi\lrcapi-1.6.0-Windows-AMD64.exe --port 8080
```

### Docker

```bash
docker run -d -p 8080:8080 hisatri/lrcapi:latest
```

使用本地 API 时，将请求地址改为 `http://localhost:8080`。

## 与 netease-daily-v3 联动

两个技能配合使用效果最佳：

1. **netease-daily-v3** 每天自动获取日推歌单，推送到 GitHub Discussion
2. 日推内容中每首歌都携带 **lyric-sense** 跳转链接
3. 用户点击链接即可在 lyric-sense 界面实时同步歌词

相关项目：[netease-daily-v3](https://github.com/adminlove520/netease-daily-v3)

## 注意事项

- 公共 API `api.lrc.cx` 有请求频率限制，高频使用请自部署
- LRC 格式包含时间戳，可用于进度同步
- 部分冷门歌曲可能无歌词，可尝试英文歌名或不填歌手
- 网页界面支持 `?artist=` `?title=` URL 参数直接加载

---

🦞 Skill for OpenClaw | Made by 小溪 | v2.0.0
