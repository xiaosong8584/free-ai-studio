# Free AI Studio — Obsidian 插件

> **语言 / Language**：[English](./README.md) · [中文](./README-cn.md)

在 Obsidian 里一站式生成 **文本 / 图片 / 视频 / 配音 / 小说封面 / 数字人口播**，并合成 **短剧成片**（含红果规范七阶段流水线）。多厂商 API 统一管理，产物直接写进 Vault，支持 Markdown 双链与重启续跑。

- 当前版本：**v0.16.3**（2026-10-12）
- 要求 Obsidian **≥ 1.7.2**
- 桌面端专用（`isDesktopOnly: true`；需 Node/Electron 与 ffmpeg）
- 发布说明：[`CHANGELOG.md`](./CHANGELOG.md)

---

## 核心能力

| 面板 | 能力 |
|------|------|
| **文本** | 4 种模式（自由写作 / 短剧大纲 / 分镜脚本 / 提示词优化）；JSON 输出自动结构化渲染；保存为带 frontmatter 的笔记 |
| **图片** | 文生图 / 图生图；参考图可从 **Vault 选择**、**本地上传**或粘贴 URL；自动压缩归一化；并发闸门防 429 |
| **视频** | 文生视频 / 图生视频；四分支协议；任务持久化，**重启自动续跑** |
| **配音** | **edge-tts 免费配音**（9 音色 / 6 情感 / 3 语速 / 标点停顿）+ **豆包 Seed-Audio 整集配音**（多角色对白 + 环境音 + BGM + 字级字幕） |
| **短剧** | 梗概 → 大纲 → 分镜 JSON → 逐镜图片 → 逐镜视频 → **配音 → 成片合成** → Markdown 笔记，一键编排 |
| **红果 Pro** | 剧本 → 分镜 → 定妆 → 首尾帧 → 视频 → 配音 → 成片，**七阶段编排**；多版选优 + 补写 + 定点校正；**视频 KEY 池轮换**；断点续跑 + 单点重生 |
| **封面** | 书名 + 作者 + 简介（**可导入 TXT/DOCX**）→ 文本模型产出提示词 → 出图 → **裁到目标画幅** → **排版书名/作者名** → 三件套写入 Vault；实时排版预览 |
| **数字人** | 文稿 → **分段** → 配音 → **形象图 I2V**（失败自动降级 Ken Burns）→ 段成片 → 字幕 → 成片，**四阶段编排**；5 种表演风格；逐段重生 |
| **解说片** | 调研 → 口播稿 → 分镜 → 帧图 → 配音 → **Remotion 渲染**；**BGM 底轨**（C 大调 + 音阶琶音）+ 静态 QC 报告；三阶段 system 提示词可配置 |
| **日志** | **运行期追踪面板**：HTTP 请求 / ffmpeg 子进程 / 各流水线阶段推进的完整时间线；模块过滤、自动跟随最新、一键导出为笔记 |

**共同特性**

- 8 厂商统一路由 + Key 回退链（自定义模型 → 单厂商 Key → 全局 text Key → Agnes 主 Key）
- **账号池（本机免费额度池）**：国内 doubao-account-pool / 海外 dola-pool；地址与模型 ID 解耦，带连通性检测
- **视频 KEY 池**：最多 3 把按镜号轮换，配进程级提交窗口，解决"一把 KEY 撑不完 10 个镜"
- **ffmpeg 成片**：拼接（自动处理分辨率/帧率不一致）、字幕烧录、音画合轨；探测不到二进制时逐级降级
- **Canvas 排版**：封面裁剪与文字排版全在 Canvas 2D 完成，不依赖 PIL / ffmpeg
- **幂等阶段**：三条主流水线的每个阶段都是独立幂等函数，每步落盘 → **断点续跑白捡**
- **VIP 门禁**：`deviceFingerprint` + gate 双层，VIP 用户不显示广告
- **顶部广告位**：预留态，默认不显示，作者可后期一键开启
- **通用韧性**：指数退避重试器 + 并发信号量 + edge-tts 时钟漂移自愈 + 微软服务故障识别
- **Ollama 本地模型自动探测**（启动 + 每 5 分钟）
- **任务清理**：30 天自动 + 手动按钮
- **Popout 窗口与主题切换适配**
- **UI 可访问性**：删除二次确认 5 处危险操作、状态 chip、折叠角标、快捷键可见、聚焦态样式

---

## 安装

### 从 GitHub Release 下载（推荐）

1. 打开 GitHub Release 页面，下载最新版本的 **`.zip`** 压缩包
2. 解压到 ` <YourVault>/.obsidian/plugins/agnes-ai-studio/`，确保目录结构如下：

```
<YourVault>/.obsidian/plugins/agnes-ai-studio/
├── main.js
├── manifest.json
├── styles.css
└── versions.json
```

3. 在 Obsidian 设置 → 第三方插件 → 启用 **Free AI Studio**
4. 点击左侧边栏 ✨ 图标打开工作台

### 前置依赖

- **ffmpeg**（配音、成片、红果 Pro 合成阶段必需）：不随插件分发，需自行安装；探测不到时相关功能会自动降级隐藏
  - Windows：`winget install ffmpeg` 或 [gyan.dev](https://www.gyan.dev/ffmpeg/builds/)
  - macOS：`brew install ffmpeg`
  - Linux：`sudo apt install ffmpeg`

---

## 快速开始

1. 打开工作台 → 右下角「打开设置」
2. 填入至少一个厂商的 API Key（推荐 Agnes 主 Key，作为所有回退链的终点）
3. 回到任一 Tab，输入内容 → 生成
4. 产物直接落到 Vault 当前目录，Markdown 笔记带 frontmatter 便于双链与检索

---

## 厂商支持

| 厂商 | 文本 | 图片 | 视频 | Base URL |
|------|:---:|:---:|:---:|----------|
| Agnes AI | ✅ | ✅ | ✅ | `api.agnes-ai.cn/v1` |
| DeepSeek | ✅ | ❌ | ❌ | `api.deepseek.com/v1` |
| OpenAI GPT | ✅ | ❌ | ❌ | `api.openai.com/v1` |
| 通义千问 | ✅ | ✅ | ✅ | `dashscope.aliyun.com/...` |
| 豆包 | ✅ | ✅ | ✅ | `ark.cn-beijing.volces.com/api/v3` |
| MiniMax | ✅ | ✅ | ✅ | `api.minimaxi.com/v1` |
| SenseNova | ✅ | ✅ | ❌ | `token.sensenova.cn/v1` |
| Ollama | ✅ | ❌ | ❌ | `localhost:11434/v1` |

### API Key 回退链

```
自定义模型 → {vendor}_key → text_api_key → agnes 全局 Key → agnes_extra_keys[0]
```

### 隐私与安全

- API Key **桌面端经 Electron safeStorage 加密**落盘（`enc:v1:` 前缀），内存中仍为明文
- 移动端或无系统凭据服务时回退明文；换机器或凭据失效时字段清空并提示
- **本地模型**（Ollama）流量不出本机；其他厂商请求走对应 API

---

## 已知限制

- **仅桌面端**：配音、成片与红果 Pro 的 ⑦ 合成、数字人口的 ④ 合成需 Node + 外部 ffmpeg
- **文本生成无流式输出**：`requestUrl` 不支持 SSE
- **大视频（>100MB）**：全量读入内存，可能 OOM
- **封面**：不打包字体文件，中文渲染依赖系统字体栈
- **多参考图**：支持多张 URL，但无"参考板合并"
- **MTV 生成、无限画布、InkOS 写作台**：暂未实现

---

## 常见问题

**Q：为什么启用后没有反应？**
先确认 `manifest.json` 里 `minAppVersion` 与你的 Obsidian 版本匹配（当前要求 ≥ 1.7.2）。

**Q：ffmpeg 报找不到？**
命令行运行 `ffmpeg -version` 确认已安装且在 PATH 中。Obsidian 桌面端继承系统 PATH，命令行能跑通，Obsidian 里也能找到。

**Q：edge-tts 出现 403？**
三种原因：① 微软服务故障（自动识别 + 快速失败 + 明确提示）；② 本地时钟偏移（自动校正，会话级，最大 ±5 分钟）；③ 凭证错（罕见，3 次退避重试）。

**Q：断点续跑怎么用？**
三条主流水线（简化版短剧 / 红果 Pro / 数字人口播）每步都落盘，重启后回到面板会看到「断点续跑」按更新时间倒序列出历史运行。

**Q：怎么更新插件？**
到 Release 页面下载最新版覆盖 `.obsidian/plugins/agnes-ai-studio/` 下的 4 个文件，重启 Obsidian 即可。**注意**：设置存于 `data.json`，不在 zip 包内，覆盖不会丢配置。

---

## 作者与支持

- **作者**：xiaosong8584
- **GitHub**：<https://github.com/xiaosong8584>
- **推广 / 支持**：<https://afdian.com/a/xiaosong8584>
- **B 站**：<https://space.bilibili.com/505631203>
- **License**：MIT（详见 [`LICENSE`](./LICENSE)）

---

## 发布说明

- 版本历史：[`CHANGELOG.md`](./CHANGELOG.md)
