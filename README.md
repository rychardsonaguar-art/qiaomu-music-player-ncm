# qiaomu-music-player-ncm

网易云音乐全能 Claude Code Skill —— 播放控制、搜索、队列管理、偏好分析、智能推荐、场景音乐、定时推送，一个 Skill 全搞定。

> 对标 [qiaomu-music-player-spotify](https://github.com/joeseesun/qiaomu-music-player-spotify)，专为华语音乐场景优化。

---

## 功能一览

| 功能 | 说明 |
|------|------|
| 🎵 播放控制 | 播放/暂停/下一首/上一首/音量/进度跳转 |
| 🔍 搜索 | 搜歌曲、专辑、歌单，支持中文关键词 |
| 📋 队列管理 | 加入队列、插入下一首、清空队列 |
| 🌙 场景推荐 | 16 个中文场景映射（深夜/跑步/工作/复古…） |
| 🎯 风格直播 | 16 种风格直接搜歌单播放（民谣/摇滚/粤语…） |
| ☁️ 每日推荐 | 网易云每日30首，自动过滤可播曲目批量播放 |
| 💓 心动模式 | 基于红心歌单口味的深度推荐 |
| 📻 私人漫游FM | 每次3首，持续探索新音乐 |
| 📡 雷达歌单 | 网易云智能推荐歌单 |
| ❤️ 红心管理 | 红心/取消红心当前播放歌曲 |
| 📝 歌词查询 | 获取当前歌曲完整歌词 |
| 📊 听歌排行 | 本周/全部听歌排行 |
| 🧠 偏好分析 | 分析红心歌单，生成偏好画像（内容维度 + 时间维度） |
| 🤖 智能推荐 | 基于偏好画像自动搜索推荐，带评分和两层推荐理由 |
| ⏰ 定时推送 | 配置定时音乐推送（cron 或 Claude CronCreate） |
| 🛠️ 一键安装 | 自动检查并引导安装 ncm-cli、mpv、API Key、登录 |

---

## 前置要求

- **Node.js >= 18**
- **网易云音乐开放平台 API Key**（appId + privateKey）
  - 申请地址：https://developer.music.163.com/st/developer/apply/account?type=INDIVIDUAL
- **mpv**（播放功能需要，Skill 会自动引导安装）

---

## 安装

### 1. 安装 Skill

```bash
# 进入 Claude Code skills 目录
cd ~/.claude/skills

# 克隆此 Skill
git clone https://github.com/joeseesun/qiaomu-music-player-ncm
```

### 2. 安装 ncm-cli

```bash
npm install -g @music163/ncm-cli
```

### 3. 配置 API Key

```bash
ncm-cli config set appId <你的AppId>
ncm-cli config set privateKey <你的PrivateKey>
```

### 4. 配置播放器

```bash
ncm-cli config set player mpv      # 跨平台内置播放器（推荐）
ncm-cli config set player orpheus  # 调用本地云音乐 App（仅 macOS）
```

### 5. 登录

```bash
ncm-cli login --background
# 扫描二维码完成登录
```

---

## 使用方式

在 Claude Code 中直接用自然语言触发：

```
播放许巍的歌
来首深夜适合听的民谣
我想听摇滚
播放每日推荐
开启心动模式
分析我的音乐偏好
帮我推荐几首歌单
暂停
下一首
红心这首歌
显示歌词
```

### 触发关键词

`播放音乐` `来首歌` `想听` `网易云` `云音乐` `暂停` `下一首` `上一首` `音量` `每日推荐` `心动模式` `私人漫游` `红心` `听歌排行` `歌词` `雷达歌单` `分析偏好` `/ncm` `/netease`

---

## 核心设计

### 播放决策流程（5条路径）

```
用户输入
  ├── 路径B（优先）：匹配风格/场景 → 直接搜歌单播放
  ├── 路径A：具体歌手/歌曲 → 搜索 → 可播性检查 → 播放
  ├── 路径D：批量播放 → 并行搜索 → 播第一首 + 其余加队列
  ├── 路径E：网易云特色 → 每日推荐/心动/FM/雷达
  └── 路径C：模糊场景 → 推荐3-5个方向 → 等确认 → 播放
```

### 可播性检查（强制）

播放前必须验证：
- `visible == true`
- `plLevel != "none"`
- `freeTrailFlag == false`

任何一项不满足 → 跳过，找下一首。

### 偏好分析系统

分析红心歌单（最多200首），从两个维度建立偏好画像：
- **内容维度**：曲风占比、情绪方向、语言偏好、代表艺人
- **时间维度**：高频红心时段、周期规律、时段-内容关联

画像缓存到 `~/.config/ncm/ncm-preference.json`（24小时有效）。

### 推荐输出格式

每条推荐包含：
- ⭐ 评分（0-100，与偏好画像的匹配度）
- 📝 两层推荐理由（偏好关联层 + 内容特质层）
- 🔗 可点击链接（`music.163.com` 明文 ID）

---

## 场景映射（16个）

| 场景 | 推荐方向 |
|------|---------|
| 深夜/失眠 | 民谣、轻音乐、空灵 |
| 有活力/热血 | 摇滚、朋克、金属 |
| 伤感/思念 | 情歌、华语流行 |
| 空灵/冥想 | 纯音乐、后摇、氛围 |
| 写代码/专注 | Lo-fi、纯音乐 |
| 开车/公路 | 摇滚、民谣 |
| 运动/跑步 | EDM、电子、嘻哈 |
| 复古/怀旧 | 粤语经典、老歌 |
| 实验/前卫 | 实验、噪音、后摇 |
| 爵士/慵懒 | 爵士、Bossa Nova |
| 古典/优雅 | 古典、钢琴 |
| 国风/传统 | 中国风、国乐、古风 |
| 嘻哈/说唱 | 说唱、Hip-Hop |
| 电子/科技感 | 电子、EDM、House |
| 粤语/港风 | 粤语流行、经典港乐 |
| 台湾/文艺 | 台湾流行、台湾民谣 |

---

## 状态文件

| 文件 | 用途 |
|------|------|
| `~/.config/ncm/ncm-preference.json` | 偏好画像缓存 |
| `~/.config/ncm/ncm-history.json` | 已推荐歌单记录（去重） |
| `~/.config/ncm/ncm-schedule.json` | 定时推送配置 |

---

## 版本历史

### v2.0 (2026-03-23)
- 大合并：整合 ncm-cli-setup、netease-music-cli、netease-music-assistant 全部内容
- 新增偏好分析系统（内容维度 + 时间维度）
- 新增智能推荐流程（意图识别、搜索策略、两层推荐说明）
- 新增定时调度管理

### v1.0 (2026-03-23)
- 初始版本：播放控制、搜索、队列、网易云特色功能、场景映射

---

## 关联 Skill

- [qiaomu-music-player-spotify](https://github.com/joeseesun/qiaomu-music-player-spotify) — Spotify 版本，含5947风格数据库
- [suno-music-creator](https://github.com/joeseesun/suno-music-creator) — AI 音乐生成

---

## 作者

Created by 乔帮主 with Claude Code

- **X (Twitter)**: [@vista8](https://x.com/vista8)
- **微信公众号「向阳乔木推荐看」**:

<p align="center">
  <img src="https://github.com/joeseesun/terminal-boost/raw/main/assets/wechat-qr.jpg?raw=true" alt="向阳乔木推荐看公众号二维码" width="200">
</p>

---

Backend: [ncm-cli](https://www.npmjs.com/package/@music163/ncm-cli) by NetEase Music
