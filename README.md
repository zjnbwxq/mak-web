# mak-os / mak-web

翁小淇 (Xiaoqi Weng) 的个人网站 —— 一个伪装成 1998 年复古操作系统的作品集。
**Sound · Code · Stories.**

纯静态单文件站点，零依赖、零构建步骤，直接丢进 GitHub Pages 就能跑。

---

## 一、这是什么

打开网站后是一段**开机启动序列**，像一台老电脑在加载一个叫 `mak-os` 的操作系统：

```
CRT 通电 → BIOS 自检 (含语言选择 EN/中文) → 内核加载日志
→ system ready → 打招呼 (问访客名字) → 偏好选择 (桌面/探索/终端)
→ 进入对应模式
```

启动过程中按 **Enter** 可以调出"快速启动菜单"跳过动画，直接进入内容。

目前 **桌面 (desktop)** 模式已完成；探索 (explore)、终端 (terminal) 两种独立模式待开发（终端目前作为桌面里的一个应用存在）。

---

## 二、系统架构

```
mak-web/
├── index.html            ← 整个站点 (单文件,~208KB)
├── .nojekyll             ← 告诉 GitHub Pages 不要用 Jekyll 处理
├── README.md             ← 本文件
└── audio/
    ├── playlist.json     ← 音乐清单 (加歌只改这个文件)
    └── *.mp3             ← 音频文件 (自己上传)
```

### index.html 内部结构

整个站是一个 HTML 文件，内含三层：

1. **CSS** —— 纸质美学（米黄纸面、噪点纹理、咖啡渍、墨色字符）+ 桌面 OS 样式（窗口、图标、任务栏）。
2. **三个打字机音效** —— 以 base64 内嵌（约 150KB，占文件 73%）。内嵌是刻意的：音效随页面即时到位，打字声零延迟。
3. **JavaScript** —— 全部逻辑：

   - **打字机引擎** (`type` / `stampChar` / `clickSound`)：逐字符打印，每个字有随机墨色/位移/微倾；按键声从三个真实样本随机变调变速变力度，听起来像一台有脾气的真打字机。
   - **启动序列** (`boot`)：BIOS → 内核 → 打招呼 → 偏好选择。
   - **双语系统** (`T` 字典 + `t()` 函数)：`lang` 变量控制 'en' / 'zh'。BIOS 技术日志保持英文，面向访客的文字双语。
   - **快速启动菜单** (`openBootMenu` 等)：启动过程中按 Enter 调出，可暂停/恢复动画。
   - **桌面环境** (`launchDesktop` / `openWindow` / `winContent`)：
     - 桌面图标（about / projects / papers→研究 / music / fiction / blog）
     - 可拖动、可缩放（右下角把手）、可最大化（圆点或双击标题栏）的窗口
     - 任务栏（品牌名 / 时钟 / 终端入口）
     - 淡色英文名水印
   - **终端** (`openTerminal` / `runTermCommand`)：桌面里的一个终端窗口，支持命令 `help` `clear` `ls` `open <name>` `about` `neofetch`，外加隐藏彩蛋 `captain`（你的边牧）。

### 设计约定

- **配色**：纸面 `#c4b9a0`，墨色 `#1a1612`，强调用铁锈红 `#963a32`（仿打字机黑红双色带，只用在系统名和 tagline）。
- **字体**：Special Elite（打字机字体，Google Fonts 加载）。
- **无外部依赖**：除了 Google Fonts，没有任何框架、库、CDN。断网也能跑（只是没字体）。

---

## 三、部署到 GitHub Pages

### 1. 创建仓库

在 GitHub 新建一个仓库，命名 `mak-web`（公开）。

### 2. 上传文件

把本目录所有文件传上去（保持文件夹结构，`audio/` 要在）。
可以网页拖拽 (Add file → Upload files)，或用 git：

```bash
git clone https://github.com/你的用户名/mak-web.git
cd mak-web
# 把这些文件复制进来
git add .
git commit -m "init mak-os"
git push
```

### 3. 开启 Pages

仓库 → Settings → Pages → Source 选 `main` 分支、根目录 `/ (root)` → Save。
几分钟后站点上线：`https://你的用户名.github.io/mak-web/`

### 4. (可选) 绑定自定义域名

如果你有 `xiaoqiweng.com` 之类的域名，在 Pages 设置里填 Custom domain。

---

## 四、怎么加音乐 (内容与代码分离)

音乐由 `audio/playlist.json` 驱动 —— **加歌永远不用碰 index.html**。

### 步骤

1. 把 mp3 传到 `audio/` 文件夹。**文件名用英文+数字，不要中文和空格**（如 `chopsticks.mp3`、`game-01.mp3`）。
2. 打开 `audio/playlist.json`，在 `tracks` 里加一段。
3. 提交。播放器会自动读取、自动归类、自动显示。

### playlist.json 格式

```json
{
  "categories": {
    "game":      { "en": "Game Music",  "zh": "游戏音乐" },
    "pop":       { "en": "Pop Music",   "zh": "流行音乐" },
    "recording": { "en": "Recording",   "zh": "录音" },
    "compose":   { "en": "Composition", "zh": "作曲" }
  },
  "tracks": [
    {
      "title": "Chopsticks",
      "category": "pop",
      "url": "audio/chopsticks.mp3",
      "year": 2020,
      "note": ""
    }
  ]
}
```

**字段说明**

- `title` (必填)：曲目名
- `category` (必填)：必须是 `categories` 里定义过的 key
- `url` (必填)：相对路径，如 `audio/xxx.mp3`
- `year` (选填)：年份
- `note` (选填)：一句话备注，如 `"for Be My Guest"`；不要就留 `""`

**加新分类**：在 `categories` 里加一行，例如
`"film": { "en": "Film Score", "zh": "影视配乐" }`，
然后某首歌的 `category` 填 `"film"`，分类标签自动出现。

### JSON 注意事项

- 引号一律用英文双引号 `"`
- **最后一项后面不能有逗号**（JSON 严格语法）
- 改完可以丢进 jsonlint.com 之类的工具验证一下，避免格式错误导致整个清单加载失败

> 注意：自定义纸质播放器（读取这个 json）尚未接入 index.html。
> 当前 music 窗口暂用嵌入式 SoundCloud / 网易云播放器。
> 自定义播放器是下一步开发项 —— 届时它会读取 `audio/playlist.json`。

---

## 五、待办 / 路线图

- [ ] 自定义纸质音乐播放器（读取 playlist.json，铁锈红进度条，分类切换）
- [ ] projects 窗口：内部子界面（点项目在站内展开详情，不跳外站）
- [ ] explore 模式（偏好选择里的"探索"）
- [ ] terminal 模式（偏好选择里独立的全功能终端）
- [ ] 1-bit 小游戏（作为桌面里的一个应用 / 彩蛋）
- [ ] Captain 简历页（作为桌面应用挂入）

---

## 六、内容来源

站内真实内容取自 https://zjnbwxq.github.io/Xiaoqi-Weng.github.io/
（GitHub 作品集主页与 portfolio 子页）。

© 2026 Xiaoqi Weng · mak-os
