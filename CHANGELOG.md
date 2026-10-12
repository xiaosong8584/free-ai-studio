# Changelog

本文件是**发布说明的事实来源**：GitHub Release 的说明、社区插件市场的版本记录都以这里为准。
每版一屏内读完。

格式遵循 [Keep a Changelog](https://keepachangelog.com/)；版本号与 `manifest.json` 严格一致。

## 0.16.3 — TTS 交叉探测 + 版本升级

**主题**：edge-tts 后端整体宕机时给用户准确错误提示。

### Fixed

- edge-tts WebSocket 握手 403 时**交叉探测 HTTP 端点** `/voices/list`：微软整体宕机时 WS 端点 body 为空，
  原来只靠 WS body 里的指纹匹配不上，用户会看到误导性的"WebSocket 握手被拒绝"。加上对 HTTP 端点的
  交叉探测后（60 秒缓存，5 秒超时），同样的故障会明确抛"微软 TTS 服务暂时不可用，稍后重试或改用豆包配音"。
- `CHROMIUM_FULL_VERSION` 143 → 155：用户报"11 镜全部 403"排查中发现，本机 Edge 已从 133 升到 155，
  代码里写死的 143 是超过一年的过期版本号，会被微软服务端静默拒绝。

### Docs

- `doc/20-tts.md` 新增 §2.4「版本号维护」+ §6.1 从「3 条路径」扩展到「4 条路径」+ §6.2「诊断速查表」
- `doc/14-changelog.md` v0.16.2 条目更新为「三条路径 + 版本升级 + 交叉探测」
- **新增** `doc/36-tts-alternatives.md`：TTS 引擎替代方案评估（阿里云 NLS / ChatTTS / CosyVoice / GPT-SoVITS / 剪映 / Web Speech API）

### 排查套路沉淀

edge-tts 403 三条岔路：
1. **IP 级阻挡** → 配代理（探测：`https://speech.platform.bing.com/consumer/speech/voices/list`）
2. **Chromium 版本过期** → 更新 `CHROMIUM_FULL_VERSION`
3. **服务端整体宕机** → 等恢复（探测命中"服务不可用"HTML 指纹）

**踩坑**：别拿 `/readaloud/edge/v1` 当探测地址——那是 WebSocket 端点，浏览器发不了 Upgrade 头，
永远打不开。用 `/voices/list`（HTTP 端点）才是有效信号。

---

## 0.16.2 — 品牌改名 Free AI Studio + TTS 403 韧性 + UI 阶段归属修复

**主题**：TTS 韧性 + UI 归属正确性 + 品牌迁移——三个都是**用户视角看到「错」**但代码本身没错的类别。

### TTS 403 三条路径

用户报「11 镜全部 403」，排查后确认是**微软后端整体宕机**。三种触发路径各有独立处理：

1. **真凭证错**（罕见，`Sec-MS-GEC` 签名不合法）：走原有 3 次退避重试
2. **服务故障**（`<h2>Our services aren't available</h2>`）：`containsServiceUnavailable`
   短路重试，直接抛 `err.tts.serviceUnavailable`（i18n 有明确文案）
3. **时钟漂移**（本地时钟与服务器偏差）：从 `Date` 响应头自动校正，会话级变量
   `clockSkewSeconds`，最大容忍 ±300s（对齐 `Sec-MS-GEC` 5 分钟窗口）

**测试**：`media-protocol.test.mjs` 新增 6 例覆盖 offset 解析、边界、跨窗口、宕机指纹。

### UI 阶段归属

「4·生成素材」按钮视觉上误触「3·生成分镜」——`refreshGroups` 把 `frames` 归到了 Group 3，
`generateAssets` 的图片进度写到了 Group 3 的 `storyboardProgressEl`。新增 `framesProgressEl`
挂到 Group 4 body，`refreshGroups` 把 `frames/audio/bgm/subtitle` 全部归 Group 4。

### 品牌迁移

- `Agnes AI Studio` → `Free AI Studio`（15+ 用户可见字符串）
- 作者：`xiaosong8584`
- 链接：`authorUrl` = github.com/xiaosong8584、爱发电（主 CTA）、B 站
- `manifest.id` `agnes-ai-studio` 保留（社区插件不可改）
- 顶部广告位（TopAdSlot）代码预留：支持文字/图片两种形态，默认 `mode:"off"` 不显示，
  上线改配置即可；VIP 强制不显示，非 VIP 与底部 AdBanner 共用 24h 节流计数器

### 解说片「文本模型」标签

ExplainPanel 的「模型」字段改标签为「文本模型」，加一行 hint 说明"用于调研/口播稿/分镜，
不走视频生成"——用户误认为是「视频模型」并询问能否删除，实际是核心 LLM 环节。

### 测试

- 累计 1137/1137 pass（TopAdSlot 20 例 + TTS 403 韧性 6 例新增，无回归）

## 0.16.1 — UI 优化整轮：把「下一步该做什么」从隐性推到显性

**这一轮 5 项**（P0 删除安全网、P1-3 主按钮层级、P1-4 折叠态角标、P1-5 顶部状态 chip、P2 键盘可访问）
的共同主线索：把当前状态、下一步该做什么、有快捷键，**从隐性（tooltip / hover / 点开折叠）推到显性（常驻可视）**。
总改动量小（+6 i18n key、~100 行 CSS、~100 行 TSX），全部**复用现有 CSS token**，不发明新概念。

### 删除安全网

- **5 处危险删除按钮加 Modal 二次确认**：成片（10-14 分钟生成成本）、配音 / BGM / 字幕 / 清单。
  `confirmDeleteFile(path, name?)` 统一入口——标题带文件名、正文带完整路径 + 回收站恢复提示、
  `mod-warning` 红按钮、`settled` flag 防重复 resolve。缩略图（ffmpeg 内部缓存）不加，误删代价低。

### 视觉层级

- **runBtn 追加 `.agg-btn-lg`**：pipeline 面板已用作主按钮尺寸规范（大 padding + 15px + 700），
  这里直接复用。`.agg-explain-bar:has(.agg-btn-lg) { gap: 14px }` 只作用于顶部 bar（组内保持 8px），
  stop/save 按钮视觉上退到第二梯队；主按钮加轻 box-shadow 立体感。

### 折叠态可见

- **分组 summary 右侧角标**：`is-running`（蓝底 + 圆点 pulse）/ `is-error`（红底白字）/
  `is-warn`（橙底白字）/ `is-ok`（绿底白字）/ `is-hidden`。5 个分组各自一个角标，
  `refreshBadges()` 单一入口算状态——research / narration / storyboard 直接映射 stage；
  assets 合并 frames / audio / bgm 三个 stage；remotion 优先 running → error 数 → warn 数。
  角标**只做减法**：idle / 无 QC 一律隐藏，只显示「跑着 / 有错 / 有告警」三种。

### 顶部状态 chip

- **顶部常驻 chip 取代 tooltip 的禁用原因信息**：优先级 跑中 > QC 有错 > 待生成。
  跑中：橙 chip 「正在生成：{stage 名}」（复用 `stageLabel`）；
  QC 有错：红 chip 「QC 有 N 个阻断问题」；主题非空：灰 chip 「待生成」；主题空：`is-hidden`。
  触发点：`refreshButtons` 尾 + `renderQcReport` 尾 + topic 输入监听。

### 键盘可访问

- **runBtn + stopBtn 内嵌 `<kbd>` chip**：Ctrl+Enter / Esc 直接可见，不必 hover 才看到 tooltip。
  主按钮变体用半透明白底 + 白字反色。其他按钮（各分组按钮）没绑定快捷键，不加 chip——不虚标。
- **`.agg-btn:focus-visible` 内嵌 accent 环**：Tab 键盘循环到按钮时明确视觉锚点；
  用 `:focus-visible` 而非 `:focus`，避免鼠标点击后按钮留一圈蓝框。

### 基础设施

- **`.gitattributes`** 钉 `* text=auto eol=lf` + 二进制白名单。之前 Windows `core.autocrlf=true`
  下，一次 i18n 提交曾让 git 显示 3758/3760 全改（实际只加 2 key，是 CRLF/LF 差量）。
  钉住后新文件走 LF；历史 CRLF 文件不动，避免一次性大 diff。

### 测试

- 累计 1137/1137 pass（无回归）。

### 补一句 v0.16.0

上一版（VIP 门禁 / 广告 / Explainer P3+P4）出 tag 时没更新 CHANGELOG。这里一并记：
P3（Explainer 质量对齐）+ P4（Remotion 渲染 P4-a/b 高光时刻 / heroPos / haloRing）+ VIP 门禁 + 广告系统。

## 0.15.20 — Explainer 真机十项排障 + 渲染稳定性

这一轮是被真机截图推着走的，十条问题全部修复。共同点：**每条在 patch 之前，至少四条我都先判断错了方向**，
所以逐条记的是「怎么判断出根因」，而不只是改了什么。

### 稳定性

- **点「生成分镜」无反应**（点了完全没动静，「生成素材」却能连带跑出分镜）：根因不是逻辑错，是
  `generateStoryboard()` 同时服务流水线（期望 `throw`）和单步按钮，而按钮把 promise 丢了 →
  `throw` 变成 unhandled rejection 被静默吞掉。引入 `singleShot = !signal` 双分支：
  单步路径弹 Notice 返回 false，流水线照旧 throw。
- **主题填仓库链接却调研出 0 份文档**：不是 URL 解析错，是 `raw.githubusercontent.com`
  （Fastly CDN 段）整段不可达，而 `github.com` / `api.github.com` 同期都正常。
  新增 **GitHub Contents API** 兜底通道，未认证限额 60 req/h。
- **Remotion 渲染跑到 68% 失败**（Chrome out of memory or disk space）：报错把「内存」和
  「磁盘」并列，实测才分得清——C 盘 96% 满才是元凶，RAM 空闲 10 GB 完全没问题。

### 功能

- **独立「渲染并发」设置**：发现面板原有的「并发」下拉**根本调不到渲染**——它只管出图/配音，
  渲染并发此前写死 `'50%'` 且调用处压根没传这个字段。现为真正的旋钮，档位按本机核数现算
  （Remotion 传入超过核数的数字会直接抛错）。
- **「渲染临时目录」设置**：把 `os.tmpdir()` 从快满的 C 盘挪走（帧缓存 / Chrome userDataDir /
  bundle 三处全走它）。实测 Windows 上只认 `TEMP`，`TMP` / `TMPDIR` 会被忽略。
  目录建不出来静默回落，不因挪盘失败挡住渲染。
- **配音 / BGM / 字幕 / 素材清单缩略图**：音频是波形图，字幕是文本行、清单是列表格（纯几何、
  形状只由类型决定）。点击缩略图或路径块在 Obsidian 标签页打开原文件。
- **「重新生成」取代「在 Obsidian 打开」**：字幕按当前配音时长现算；清单从 vault 现读
  `TFile.stat.size`（流水线那份用的是内存态 bytes，重载后就没了）。内存态为空时明确提示，
  不写残缺清单。
- **解说词时长偏差即刻报警**：不等 QC、不等配音实测，生成完那一秒就比 `plan.totalSec` 与
  `durationSec`，超 ±15% 报警并自动重试一次。边界与 QC 的 `AUDIO_TOTAL_TOL` 同方向。

### 体验

- 分镜缩略图改为**自动换行**，不再挤一行拖横向滚动条。
- **BGM 响度提升 12.2 dB**：根因不是音量小，是 97.7% 能量压在 200 Hz 以下、被 A 计权等响曲线
  扣掉 13–26 dB。修法是声部整体抬八度到 130–390 Hz + 默认音量 0.35→0.5。
  （失败过一版：给分音加随机相位会让同和弦内的撞频分量互相抵消。）

### 回归

- `npm test` **1024/1024** 全绿，`tsc -noEmit` 0 错误。
- i18n 全量静态扫描：1856 处 `T()` 调用点 vs 两套语言包各 1772 键，真实缺失 0 处。
- 环境事实已收进内部工程手册（GitHub 三域名连通性、磁盘余量、Windows `os.tmpdir()` 只认 `TEMP`、本机工具链坑）。
  磁盘余量、Windows `os.tmpdir()` 只认 `TEMP`、本机工具链坑）。

### 升级注意

- 「渲染临时目录」需在设置里手动填（如 `J:/agnes-tmp`），留空则回落到系统 Temp。
- 必须**重启 Obsidian / 重载插件**才生效。

## 0.15.19 — 视频参数侧车（生成参数与成片成对）

- **`core/videoMeta.ts`**：新增视频参数侧车，与图片侧车（`imageMeta.ts`）完全同款。提示词 / 模型 / 分辨率 / 帧率 / 时长 / 参考图清单以 JSON 落在成片同目录、同基名（`x.mp4` → `x.json`），片在参数就在。
- **参数快照挂进任务记录**：`PersistentTask.meta` 新增 `VideoMetaSnapshot`。成片文件名下载前未知，而任务可能跨重启续跑 —— 参数不落进 `_tasks.json`，重启后的完成回调就补不出侧车。
- **删除成对清理**：任务卡片删除同时 Trash 视频与侧车（`fileManager.trashFile`，尊重「删除到回收站」偏好），不留没有片的参数孤儿。
- **参考图不塞 dataURI**：本地上传只记名 + 体积，手输 URL 保留原串可复现。
- **回归测试**：新增 `test/video-meta.test.mjs` 27 条用例（路径契约 / 写入读回 / modify 覆盖 / mkdir 竞态 / 脏数据容错 / 前向兼容），套件 694 条全绿。

## 0.15.18 — 图片 CDN 下载错误友好提示

- ERR_CONNECTION_RESET / ERR_NAME_NOT_RESOLVED 修复提示：当系统代理拦截 CDN 域名时，给用户明确的修复建议（NO_PROXY / --proxy-bypass-list）
- i18n 双语支持：新增 zh-CN / en-US 对应错误文案

## 0.15.17 — 定妆物品纠偏修复

- **ASSETS_SYSTEM 硬性要求新增**：明确「物品类（车、家具、武器等）必须归 props，不得归 supports」
- **PROP_NAME_RE 扩充**：新增车/轿/伞/刀/手机/电脑/钟表/桌椅/鞋/帽等物品特征词
- **coerceCategory 名称启发式纠正**：即使 LLM 将物品错归 supports/characters，自动纠为 props
- **回归测试**：追加 3 条用例（黑色轿车→props / 手机→props / 李娜不误纠）

## 0.15.16 — 短剧流水线二十一迭代 + 声画字幕同步 + 分镜去指代

本版本累计了自 0.15.15 之后的二十一轮改动，一次收口。

### 核心功能

- **短剧面板 4 Bug 修复 + 全阶段日志**：各阶段写 `pipeline.log`，排查不再翻检查点 JSON。
- **定妆四套模板 + 智能参考图**：出图时自动挑匹配角色 / 场景 / 道具的参考图（`matchShotRefs`），400/404/422 降级文生图，其余原样抛。
- **成片归位 + 分镜时长与配音对齐 + 分角色 ASS 字幕**：`core/subtitle.ts` 纯逻辑层（`planSubtitleCues` / `buildAss` / `estimateSpeechSec`），分角色配色，字号跟分辨率走。
- **自动生成创意**：`genPremiseNow()`，「换一条」/「自动生成」入口。
- **② 生成流程步骤条点亮创意 / 大纲**：`core/stageFlow.ts` 纯模块（`STAGE_FLOW_IDS` 9 项），面板 `markPremiseStage()` 默认态点亮第一项。
- **⑦ 断点续跑 → 流水线资产列表**：面板第 7 分区展示真实目录清单，每项 = 检查点卡片（资产摘要 + 续跑 + 删除项目）。
- **删除项目 = 删整个流水线目录**（`pipelineStore.removeCheckpoint`）：优先 `trashFile`，`adapter.remove` 兜底递归删目录；只删本项目，不碰共享产物。
- **定妆区「重新生成定妆照」**：`generateCastingPhotos(force=true)` 整阶段重出，面板按钮 `force=true`，流水线续跑保持默认（不重复花钱）。
- **⑥ 成片区「重新合成」**：`recomposeFinal` 强制重出 + 失败必 throw，素材认面板当前 shots。
- **面板「重新生成视频」轮询 429 修复**：`PipelinePanel.regenShotVideo` 旧实现无 sleep（截图实锤 2 ms 连发 → 429 静默失败），改为 `planPollSchedule + delayAfterPoll + sleep` 阶梯退避，绝对时间窗口判超时。
- **doubao TTS 可中止**：`delay(ms, signal)` 接入重试退避，中止不再干等。
- **LLM JSON 加固**：`sanitizeJsonText` 字符串感知洗白 + `finish_reason === 'length'` 翻倍重试 + 原文落 `_debug/*_raw.txt` 后收紧提示词重出。
- **错误色块红压红可读性**：有色底一律配 `--text-on-accent`，不再用 `--background-modifier-*`（底色淡染，同色不可见）。
- **run 目录名带日期**：`run_YYYYMMDD_HHMMSS_xxxx`，同日多次运行可区分。
- **面板默认勾选对齐短剧全流程**：7 项 checkbox 全默认开启（`doubaoAudio` 除外）。

### 声画字幕同步修复（第十九轮根因 → 本版本落地）

- **主因 A**：`planSubtitleCues` 每镜时间窗口改用**画面时长**（`durationSec`），与 `fitSegment` 口径对齐。配音 ≤ 画面时 cue 只占配音实际朗读区间（不延伸到尾部静默区）；配音 > 画面时 cue 铺满整个窗口。
- **次因 B**：`estimateSpeechSec` 高估幅度 16%（刻意保留缓冲）已被测试钉住，不再往下调。
- 测试：`subtitle.test.mjs` 用例对齐新口径（末条 endSec = 画面总时长 34s，不再 = 配音总时长 39.7s）。

### 分镜去指代（第二十一轮）

- `STORYBOARD_SYSTEM` 新增 `characters` 必填字段规范 + 禁用指示代词（`他/她/它/这人/那人/对方/两人…` 一律写成角色真实名字，与 `characters` 字段一致）。
- `coerceShot` 宽松解析 `characters`（英文键 / 中文键 `角色`/`人物`/`cast` / 数组 / 逗号串 / 顿号串），空数组 / 全空白省略为 `undefined`。
- 新测试：`test/shot-characters.test.mjs`（5 例）+ `test/build-shot-characters-test.mjs`。

### 测试与门禁

- tsc 0 error · `npm test` **609/609**（26 个 test 文件）· eslint 真实缺陷 **0**
- `esbuild production` **702.9 KB** · `validate` 11 OK / 4 WARN / 0 ERROR
- i18n：zh-CN / en-US 各 **1,486 键**对称

## 0.15.15 — 短剧面板五分区重构 + 剧本/定妆资产 + 取消真正可中止

用户一次提了六项，这版全部落地。

- **取消立刻生效**（`core/videoSlot.ts` / `core/videoKeySlot.ts` / `core/client.ts` / `core/video.ts` / `core/pipeline.ts`）：旧实现里 65 秒限流等待、429/5xx 退避、12 秒轮询间隔全用裸 `setTimeout`，`AbortSignal` 只管网络请求本身——点「取消」后界面最长要干等 65 秒。现在睡眠统一走 `client.sleep(ms, signal)`，`submitVideo` / `submitVideoWithKeys` / `pollVideoStatus` / `downloadVideoToVault` / `pollPoolVideo` 全部接 `signal`；`isTransientPollError()` 加一道 `isAbortedError` 短路，避免中止被误判成「网络抖动」继续轮询。底层 `RequestAbortedError` 在阶段 catch 里转成 `PipelineAbortedError`，检查点落 `aborted` 而非 `failed`。Pro / 锚点 / MTV 三条兄弟链路同步对齐（不传 `signal` 时行为不变）。
- **面板五分区重构**（`ui/PipelinePanel.ts` 重写，1255 行）：① 配置区 ② 生成流程显示区（阶段胶囊 + 进度条 + 日志）③ 剧本区（可编辑 + 保存 + 打开资产文件）④ 定妆区（人物/场景/道具分组卡片，单张重出 / 整组重出 / 换图）⑤ 分镜区（每镜横向四列：参考图 → 分镜提示词 → 关键帧 → 视频片段，每步可单独重跑）。
- **剧本阶段 + 资产落盘**（`pipeline.ts::generateScript()`）：大纲之后、分镜之前插入剧本阶段，LLM 按资产围栏输出「正文 + 定妆资产清单」，`parseScriptAssets()` 解析后写 `02_剧本.md`；`assignShotCharacters()` 把资产名回填进各镜 `characters`，供分镜提示词引用。
- **定妆阶段**（`generateCastingPhotos()` / `recastAsset()` / `saveCastingAssets()`）：按人物 / 场景 / 道具分组出 1024x1024 定妆照，落 `images/casting/`，元数据写 `03_定妆.json`（含 styleAnchor）。单张失败只标 `failed`，不阻断整批。
- **图片 / 视频放大查看**（新增 `ui/MediaViewer.ts`）：图片滚轮缩放 + 拖拽平移，视频原生控件播放；定妆照、关键帧、分镜视频、成片都可点开。
- **资产注册表与旧文件兼容**（`core/pipelineStore.ts`）：`PIPELINE_TEXT_FILES` 现为 `00_创意.txt` / `01_故事大纲.md` / `02_剧本.md` / `03_定妆.json` / `04_分镜脚本.json`。分镜脚本从 `02_` 顺延到 `04_`，`readStoryboardAsset()` 对老项目做一次 `02_分镜脚本.json` 兜底读取，「续跑合并用户手改」不会静默失效。
- **顺手收掉豆包 TTS 重试退避的取消延迟**（`core/doubaoTts.ts`）：`delay(ms)` 原本不接 `signal`，第 3 次重试退避最长干等 4.5 秒才中止。改为 `delay(ms, signal?)` 可中止版本，`callDoubaoApi` 的 `DoubaoCallOptions` 增加 `signal` 字段并透传到重试循环。至此「新加的 `sleep` / `setTimeout` 都必须能带 signal」通则覆盖到最后一处。
- i18n：zh-CN / en-US 各 **1366 → 1421** 键（+55），双向键集合零差异。
- 测试：新增 `test/script-assets.test.mjs`（14 例：剧本围栏解析、定妆提示词、资产回填、可中止 sleep）与 `test/build-script-assets-test.mjs`（pipeline 打包 + obsidian 桩件）；`pipeline-checkpoint.test.mjs` +8 例（剧本 / 定妆读写、脏数据收敛、注册表、旧文件名兜底）。
- 门禁：tsc 0 error · `npm test` **506/506**（+22）· eslint 真实缺陷 **0** ·
  `esbuild production` **639.7 KB**（655,100 B）· `validate` 11 OK / 4 WARN / 0 ERROR。

## 0.15.14 — 成片音频对齐修复 + 普通流水线资产化

- **成片「只合成了一部分」根因修复**（`core/ffmpeg.ts` + `core/pipeline.ts`）：用户反馈成片尾部出现大段冻结帧。实测根因是 `sharedFormat()` 只比 `hasAudio` 布尔值，把「24kHz 单声道混音段」与「32kHz 立体声原声段」误判成参数一致走 `-c copy`，concat demuxer 静默把音轨时间轴错位（视频 31.008s / 音轨被拉成 41.398s，尾部 10 秒全是冻结帧）。修法三层：① 一致性判定纳入**采样率 / 声道数**；② 逐镜统一用 `muxSegment()` 把音轨归一为 aac/24kHz/单声道（无配音镜补等长静音，不再塞原始视频）；③ `concatVideos()` 拼接后 `sumDuration` + `assertDuration` 时长闸门——不符合就抛错而非静默成功。
- **成片失败不再静默跳过**（`core/pipeline.ts`）：新会话直接跑流水线时 `getFfmpegSync()` 缓存冷启动会误判「未检测到 ffmpeg」而整段跳过成片。改为 `runMergeStage` 主动 `detectFfmpeg`；且「没装 ffmpeg / 可用片源不足两个」改为抛 `MergeUnavailableError` 上抛，让检查点落 `failed`、面板显示失败态与「续跑补合成」入口（已生成的图/片不重跑）。
- **视频提交统一口径**（`core/pipeline.ts`）：普通流水线改走 `submitVideoWithKeys()`（与 Pro / 锚点 / MTV 三兄弟一致）——KEY 池提交 + 65s/次限流窗口 + 把返回的 `apiKey` 透传给轮询（任务归属在 KEY 上，换 KEY 查必 404）；空池不再是「退化裸提交」，而是 `[undefined]` 走同一套退避循环，撞 429/5xx 照样退避。
- **普通流水线资产化**（`core/pipelineStore.ts` + `core/pipeline.ts`）：对齐 Pro 短剧，把创意提示词 / 故事大纲 / 分镜脚本落成 Vault 内**可编辑**文件 —— `_Agnes/dramas/pipeline/{id}/` 下的 `00_创意.txt` / `01_故事大纲.md` / `02_分镜脚本.json`。分镜脚本只存创意字段（imagePrompt/videoPrompt/dialogue 等），运行时路径留检查点；并在断点续跑时读回用户在 `02_分镜脚本.json` 里手改过的分镜（只覆盖创意字段、保留 imagePath/videoPath），实现「改了某镜提示词 → 续跑只重出那一镜」。
- i18n：zh-CN / en-US 各 **1366 键**（+2：`err.ffmpeg.concatMismatch` / `err.pipe.mergeNoFfmpeg`），双向键集合零差异。
- 门禁：tsc 0 error · `npm test` **484/484**（+11）· eslint 真实缺陷 **0** ·
  `esbuild production` **610.2 KB**（624,878 B）· `validate` 11 OK / 4 WARN / 0 ERROR。

## 0.15.13 — 短剧容错、资产归拢与界面整理

对照 Flask 版（`agnes-ai-studio_full/Agnes-AI-Studio-完整包-20260918`）把短剧流程的四类缺陷一次修完：

- **重试按钮补齐**（`ui/DramaProPanel.ts`）：分镜脚本、定妆资产、首尾帧、视频片段、配音、成片
  六个阶段各留自己的「重跑」入口，失败态卡片同时显示错误文本与按钮。已完成的镜也保留
  单镜重跑（`rerunShot`）—— 不满意某一镜不该被迫整集重来。
- **合成失败可重试**（`core/proPipeline.ts`）：`stageCompose` 的错误不再被吞成 `compose = null`，
  改为抛 `ComposeError` 上抛到阶段循环，用户点了「重新合成」才会真正重跑，而不是每次都从零开始。
- **资产统一到 `_Agnes/dramas/`**：新增 `pipelineStore.pipelineProjectDir()`，普通流水线产物
  （关键帧 / 分镜视频 / 配音 / 字幕 / 成片）改落 `{outputFolder}/{proSubfolder}/pipeline/{id}/`，
  与 Pro 短剧的 `{outputFolder}/{proSubfolder}/{id}/` 同级。`generateImage` / `imgToImg` 增加
  `opts.dir`、`downloadVideoToVault` 增加 `dirOverride`、`outputDir` 增加 `baseDir`，
  不传时行为完全不变（独立面板仍写 `_Agnes/pictures`、`_Agnes/videos`）。
- **界面整理**（`styles.css` +112 行）：补上 TS 里一直在用、CSS 里却没写的样式 ——
  错误 / 告警升级为带左边框的色块，首尾帧缺失态改成虚线占位框，分镜视频与成片同款观感，
  配音列表改成两列网格，窄面板（≤640px）网格自动降为单列不再溢出。
- i18n：zh-CN / en-US 各 **1353 → 1364** 键（+11），双向键集合零差异。
- 门禁：tsc 0 error · `npm test` **473/473** · eslint 真实缺陷 **0** ·
  `esbuild production` **606 KB**（620,330 B）· `validate` 11 OK / 4 WARN / 0 ERROR。

## 0.15.12 — 面板快捷键与生成中止

Ctrl+Enter 触发当前面板的主生成动作，Esc 中止当前任务。

- 新增 `core/hotkeys.ts`：`matchHotkey()` 纯函数（Ctrl/Cmd+Enter → generate，Esc → cancel），只读按键的 4 个字段，Node 里直接单测。
- `ui/StudioView.ts` 监听挂在 `containerEl`（capture 阶段）—— 焦点不在本面板就不触发，不与编辑器 / 其它视图按键冲突；`registerDomEvent` 注册，关视图自动解绑。
- Esc 只在**确实中止了任务**时才拦截：无可中止任务时按键原样放行，Obsidian 的「Esc 关 modal / 退出编辑」保持不变。输入法组词中（`isComposing`）回车 = 确认候选词、Esc = 退出输入法，同样不拦截。
- 中止贯通到 HTTP 层：新增 `RequestAbortedError` + `isAbortedError()`；`requestRaw` 与退避 `sleep` 接受 `signal`（`requestUrl` 无 signal 参数，用 `Promise.race` 中止），`withRetry` 对中止**绝不重试**。
- i18n：zh-CN / en-US 各 +2 键（`err.client.aborted` / `common.cancelled`）；顺手修一处 lint 缺陷 `Array<Promise<T | never>>` → `Array<Promise<T>>`。
- 门禁：tsc 0 error · `npm test` **473/473**（+8）· eslint 真实缺陷 **0** · `esbuild production` **599 KB**（613,374 B）。

## 0.15.11 — 图片会话历史持久化

把图片生成历史从「内存数组」改为「落盘 Vault 内 `_image_history.json`」，关视图 / 插件重载 / 重启均不丢：

- 新增 `core/imageHistory.ts` 的 `ImageHistoryStore`：MAX=200 条 + 脏数据过滤 + `fileManager.trashFile()` 删除，与 `_tasks.json` 独立（不塞 `data.json`）。
- `ui/ImagePanel.ts`：启动异步读回历史、生成成功后 `append` / 批量生成 `appendMany`；历史区改为「标题行（标题 + 清空按钮）+ 缩略图网格」。
- i18n：zh-CN / en-US 各 +4 键（`image.historyTitle` / `image.clearHistory` / `image.historyEmpty` / `image.historyCleared`）。
- 顺手修一个 lint 缺陷：历史清理改用 `fileManager.trashFile()` 而非 `vault.delete()`（尊重用户删除偏好）。
- 门禁：tsc 0 error · `npm test` 465/465（+6）· eslint 真实缺陷 0 · `esbuild production` 596 KB。

## 0.15.10 — core 层 UI 文案 i18n 收口

v0.15.8 把设置页与面板的硬编码 UI 出口清零，但 `src/core/` 里还有一批**会进 UI** 的中文
（厂商/模型/预设标签、过程日志、状态文案、错误提示、探测结果）。英文界面下它们仍是中文。
本轮按「提示词 / 参数 / 路径 / 标识符不译」的既有契约逐条分流后收口：

- **常量字典 `labelKey` 化**：厂商名、模型名、预设名、下拉标签不再在模块级直接 `T()`
  —— 模块级 `T()` 在 import 时定型，切语言不会更新 —— 改为只存 `labelKey`，渲染期再取译文。
- **过程日志 / 状态 / 错误（335 条）**：`ctx.log` / `report` / `ApiError` 等 UI 可达文案
  全部 `T()` 化。
- **剩余 UI 出口（70 条）**：状态栏、`mergeNote`、轮询诊断细节、探测结果、阶段说明、ffmpeg 提示。
- **顺带修一个隐蔽缺陷**：4 处「日志级别嗅探」正则原本按中文判级别（`/失败|异常|不可用/`），
  **英文界面下失败行会退化成普通行，红/绿色与告警级别一起丢失**。抽出 `core/logLevel.ts`
  用中英并集正则集中判定，补 4 条用例守住。
- **语言包**：zh-CN / en-US 各 **883 → 1346** 键（+463），双向键集合与占位符集合零差异。

严格不译（按 i18n 契约）：喂给模型的提示词（`proPrompts` / `coverPrompt` / `anchorSeg` 等）、
决策层纯函数的自校验信息、trace 日志与 `console.*`、生成的 Markdown 正文与文件名、持久化数据。

门禁：`tsc --noEmit` 0 error；`npm test` **459/459**（+4）；`eslint`（`test/lint-real.mjs` 口径）
真实缺陷 **0**；`esbuild production` 通过（main.js **594 KB** / 607,998 B）。

## 0.15.9 — 视频任务卡片：缩略图 / 用时与百分比 / 提示分级

真机反馈三件事，逐条落地：

1. **产物缩略图**：下载写入 Vault 后，卡片在文件名**前面**渲染一张视频缩略图
   （`<video preload="metadata">`，`loadedmetadata` 后 seek 到 0.1s 取第一帧 ——
   不 seek 只有一块黑底）。缩略图与文件名都可点击打开成片；非视频扩展名、或路径
   还没进 Vault 索引时退回原来的文件夹图标，不显示破图。
2. **已用时长 + 完成百分比**：卡片右侧保留「开始 17:27:20」，其右新增
   「已用 2 分 30 秒 · 约 12%」，终态改显示「用时 1 分 05 秒」。百分比
   **远端自报优先**（新增 `PollResult.progress`；Agnes 2.5 的 `progress` 经
   `normalizeProgress` 归一：0–1 小数换算、>100 夹回、读不出来返回 `undefined`
   而不是假的 0），读不到才退回按**绝对时间**估算（封顶 95%，不让进度条先于
   成片满格）。秒级刷新走独立轻量 ticker，只改两个文本节点与条宽、**不重建卡片**
   —— 重建会让缩略图重新加载元数据（肉眼可见闪一下），并换掉用户正在点的按钮。
3. **「过程提示」与「终态错误」分级**：原先把「连续 N 次读到相同响应」借 error
   通道渲染成红色错误，用户看到红色以为任务已经废了 —— 实际轮询仍在继续、任务
   照样完成。现在拆成 `hint`（警示色，非致命）与 `error`（红色，终态）两档：
   `running` 说「上游仍在生成中…会继续等待（不影响完成）」，`pending` 说
   「任务仍在上游排队…会继续等待」，指纹一变自动消失。顺带把「暂时无响应」
   「轮询连续 N 次失败」等原本硬编码中文的过程提示一并 `T()` 化。

门禁：`tsc --noEmit` 0 error；`npm test` **455/455**（+5）；`eslint`
（`test/lint-real.mjs` 口径）真实缺陷 **0**；`esbuild production` 通过
（main.js **537 KB** / 550,277 B）。


## 0.15.8 — 设置页与面板全量本地化：硬编码中文 UI 出口清零

把需求 A 遗留的「设置页硬编码 UI 文案」一次性收口，并把 4 个面板里散落的 Notice 一并 `T()` 化：

- **settings.ts（200 处自动 + 4 处手工）**：端点 / Ollama / 运行日志 / Key 加密 / 账号池 / 配音 TTS /
  ffmpeg / 输出目录 / 封面 / 短剧 Pro / 数字人 / 默认模型 / 轮询 / 自定义模型 —— 涵盖
  `setName` / `setDesc` / `setButtonText` / `setTooltip` / `addOption` / `createEl({ text })` 段落 /
  `setText` 状态行 / 下拉 `textContent` / `textSetting` 名称与说明 / `row()` 表单标签。
- **面板 Notice（15 处）**：`main.ts` 6 / `CoverPanel.ts` 4 / `DramaProPanel.ts` 4 / `PipelinePanel.ts` 1。
- **顺带补齐**：`CoverPanel` 参照图来源标签；`DramaProPanel` 分镜卡标题、状态标签、七阶段名称、统计行、
  确认弹窗；`PipelinePanel` 检查点元信息；`settings` 封面/ffmpeg 预览等动态 tooltip。
- **严格排除（功能数据，非 UI 文案）**：trace 日志（`tInfo` / `tOk` / `tWarn` / `tError`）、`console.*`、
  面板 `appendLog` 进度流、喂给模型的提示词（如 `DEFAULT_STYLE`）、生成的 Markdown 正文与文件名、
  持久化的自定义模型名。
- **语言包**：zh-CN / en-US 各新增 **251** 条（其中 6 条复用既有 key），双向键对称 0 差异；
  en-US 译文本次人工补齐。

门禁：`tsc --noEmit` 0 error；`npm test` **450/450**；`eslint`（`test/lint-real.mjs` 口径）真实缺陷 **0**；
`esbuild production` 通过（main.js **533 KB** / 545,633 B）。

## 0.15.7 — 视频流水线收尾：多 KEY 续跑 404 与「成功但无文件」

延续 v0.15.6 的排查，把上一轮记录里**遗留未修**的两个真实缺陷一并解决，并复核了
轮询/下载的鉴权链路：

- **多 KEY 轮换任务重启后续跑会 404**（`PersistentTask` 缺字段）：任务归属在 KEY 上，
  提交用的那把 KEY 之前没存进持久化任务，重启续跑的轮询与下载只能拿默认 KEY 去查，
  必然 404。修法：`types.ts` 给 `PersistentTask` 加 `apiKey?`；`VideoPanel.doSubmit`
  在创建任务时记下实际用的 KEY；`startPolling` 与 `handleTaskSuccess` 把 KEY 透传给
  `pollVideoStatus` 与 `downloadVideoToVault`。单 KEY（视频 Tab）时恒为 undefined，
  行为不变。
- **`downloadVideoToVault` 下载 GET 不带鉴权头**：Agnes 2.5 的「直接吐视频字节流」分支，
  成片地址就是轮询端点本身，不带 `Authorization` 会被 401/403。新增可选
  `apiKeyOverride` 参数，仅在多 KEY 时注入 `Bearer`，单 KEY 不额外加头（保持原行为）。
- **`pickVideoUrl` 只认 4 个字段名 → 「状态机对、日志全 200、产物却凭空消失」**：
  服务端换字段名（`download_url` / `result.file` / `play_url` …）时拿不到视频。新增
  递归扫描兜底——优先 http(s) 地址、其次 data: URI，裸 base64 交给既有的
  `pickVideoBase64`；已知字段名仍优先，避免误抓嵌套里的缩略图 URL。

复核结论：v0.15.6 的 `Cache-Control: no-cache` + `Pragma: no-cache` 已覆盖**所有** GET
出口（含 MiniMax / 默认分支的 `getJson` 都走 `requestRaw`），Chromium 缓存冻结问题从根
上解决；`&_t` 缓存破坏参数仍作为「服务端/CDN 侧缓存」的备选，待真机复测若仍命中再启用。

`npm run build` 通过（tsc 0 error）、`npm test` **450/450 · 1.7s**（新增 3 条）、
eslint 真实缺陷 0（改动文件全 CLEAN）。

## 0.15.6 — 视频轮询卡满 40 分钟：读得懂的「冻结响应」

v0.15.3 修的是「读不懂的错误码」，v0.15.4 修的是「200 但读不懂的响应体」。
这次实测暴露出第三条路：**响应完全读得懂，但它永远不变**。

真机证据（用户截图）：任务 14:35:07 创建、15:15 标失败，文案「视频轮询超时（已等待
40 分钟，超过上限 40 分钟）」；日志每 30 秒一次
`→ GET /v1/agnesapi?video_id=task_i9s…&model_name=agnes-video-2.5-flash` 配
`← 200 · 3ms`，近百次无一例外。两条推论：

- **3ms 是本地 HTTP 缓存命中的量级**。一个到 `api.agnes-ai.cn` 的 HTTPS 往返不可能
  这么快 —— 拿到的是**冻结的旧响应**，而不是服务端刚查出来的结果。
- **跑满 40 分钟、而不是第 20 次就提前失败** → 状态词在闭集里、`transient` 恒为 false。
  v0.15.3 的「连续 20 次提前失败」与 v0.15.4 的「读不懂就留痕」两条安全网**同时失效**。

- **GET 统一带 `Cache-Control: no-cache` + `Pragma: no-cache`**（`client.ts`）。
  `requestUrl` 走 Electron 的 net 栈、会命中 Chromium 的 HTTP 缓存；轮询是同一个 URL
  反复查同一个任务，一旦被缓存就永远读到第一份响应。堵在唯一出口而不是调用点 ——
  `tts.ts:92` 早就为 edge-tts 单独加过同样的两个头，但那是局部补丁，视频轮询一条都没沾到。
  POST 等带体请求不受影响，调用方显式给的头优先。
- **新增响应体指纹**（`video.ts` 的 `digestOf` / `clipSnippet`）：每次轮询带回
  `bodyDigest`（djb2 + 长度）与 `bodySnippet`（压平空白、截 200 字）。四个协议分支
  （账号池除外）全部对齐，指纹变了才写一行日志，不逐条刷屏。
- **连续读到同一份响应时卡片给明示**（`VideoPanel.ts`，阈值 10 次）：文案
  「⚠ 连续 N 次响应完全相同，可能命中本地缓存或服务端卡住」。**不终止轮询** ——
  服务端真只是排队慢时，进度一变这条提示会自动消失。超时文案也会带上这条线索，
  同样等 40 分钟，用户下次知道该去查缓存而不是再来一轮瞎猜。

`npm run build` 通过（tsc 0 error）、`npm test` **447/447 · 1.9s**（新增 6 条）、
eslint 真实缺陷 0。

## 0.15.5 — 语言切换不刷新的三个漏点

i18n 核心里 locale 确实换过去了，但 UI 只重绘了工作台视图，剩下三处一直是旧语言。

- **设置页自身不重建**（主因）：`display()` 的文案是一次性写死的，只刷工作台会出现
  「下拉框已经是新语言、下面的设置项还全是旧语言」。现在 onChange 里追加
  `window.setTimeout(() => this.display(), 0)` —— 延迟到本次事件结束后重建，避免正在
  触发事件的那个下拉节点被同步移除时抛异常。
- **命令面板的命令名不跟着变**：`addCommand` 只在 onload 时解析一次 name。新增
  `addLocalizedCommand(id, key, cb)`，注册时记下 i18n key；切换后
  `applyLocaleLabels()` 就地改写 `app.commands.commands[id].name` 再 `refreshCommands()`。
  只改 name、不动 id，用户已配的快捷键按 id 存储，不会丢。
- **ribbon 悬停提示不跟着变**：`applyLocaleLabels()` 顺带把
  `.workspace-ribbon [data-icon="sparkles"]` 的 title 刷成新语言。
- 切换提示从「只在工作台刷新成功时才弹」改成**总是弹**（3 秒），带上新语言名。

`npm run build` 通过（tsc 0 error）、`npm test` **441/441 · 1.7s**、eslint 真实缺陷 0。

## 0.15.4 — 视频链路二次排障（200 黑洞 / base64 产物 / 面板文案）

v0.15.3 的安全网只认 HTTP 错误码。还有一条 **200 但插件读不懂**的路径：既不是错误
也不算成功，返回裸 `pending` 并把连续失败计数器清零，一直等到 40 分钟窗口耗尽。
四个缺陷互不冲突，一起修了。

- **不认识的状态词不再静默 pending**：状态映射的 switch 改成表（`REMOTE_STATUS_MAP`），
  新增 `isKnownRemoteStatus()` 做闭集判定。词不在表里时返回 `transient: true`，
  诊断文案把原词与响应字段全带出来（「远端 status="…" 不在已知词表里（响应字段：…）」）
  —— 这是「服务端换了措辞」唯一的线索，会计入连续异常次数并提前失败。
  词表从 7 个扩到 35 个（`complete` / `finished` / `generated` / `ready` / `failure`
  / `timeout` / `expired` / `stopped` / `generating` / `queued` / `submitted` /
  `initializing` 等），四个协议分支（账号池 / MiniMax / Agnes 2.5 / 默认）全部对齐。
- **200 但响应体不是 JSON 不再静默**：Agnes 2.5 走 `requestRaw()`，`resp.json` 为
  `undefined` 时原先一律当 pending，WAF / 企业代理 / CDN 的 200 + `text/html`
  正好落进去。现在按 Content-Type 分「空体 / 非 JSON / 非法 JSON」三种文案，
  各带原文截 160 字符，同样标 `transient`。
- **base64 成片能落盘了**：原先只走 URL 兜底链，产物是 base64 时 `videoUrl` 为
  `undefined`，任务标成功但 Vault 里没有文件。新增 `extractVideoArtifact()`：
  URL 优先，URL 不像地址就当裸 base64 包成 data URI；否则走 12 个 base64 字段名
  （`b64_json` / `video_base64` / `b64` / `base64` / `video_data` / `data` 等），
  嵌套对象与数组也认。`downloadVideoToVault()` 新增 data URI 分支（复用 `pool.ts`
  的 `dataUriToBytes()`），扩展名按响应声明的 MIME 推（`quicktime` → `.mov`）。
  本机路径与云端 URL 两条旧路径不动。
- **提交后面板文案复位**：`outEl` 原先一直停在「正在提交…」，任务跑完 20 分钟
  它也没变。提交成功后改为显示「已提交，进度见下方任务中心。」（新增 i18n 键
  `video.submitted`，中英文各一条）。
- 新增 11 条回归用例；全套 **441 用例**全绿，`tsc` 0 error、esbuild 通过、
  eslint 真实缺陷 0。

## 0.15.3 — 视频生成链路排障（超时 / 切页丢输入 / 双轮询 / 零诊断）

用户报「视频任务很慢，最后超时；切一次页面输入框就空了」。四条链路过一遍，查出四个互不相关的缺陷。
根因、状态机与验证见本文件下方条目。

- **修「查不到任务」的 404 死循环（唯一会让任务超时的真凶）**：Agnes 2.5 提交响应里的
  `video_id` 可能为空，插件用空串拼轮询 URL → 网关 404 → 被容错表当成「后端可见性延迟」继续等
  → 40 分钟绝对窗口耗尽后标超时，全程零日志。现在提交侧 `videoId = resp.video_id || resp.id`、
  轮询侧 `vid = videoId || taskId` 双保险，与 Flask 版 `drama_pro.py` / `mv.py` 对齐。
  Flask 自己的视频 Tab（`video.py:283`）也有同一个 bug，插件就是从那里继承的。
- **轮询不再清空输入框**：此前每次轮询回调都调 `render()` → `root.empty()` 全量重建，
  等于每 10 秒抹一次你正在输入的内容。现在把任务中心拆成独立子树（`renderTaskCenter()`），
  只重建那一个 div；提示词 / 模型 / 尺寸 / 参考图完全不在重建路径上。
- **修双轮询**：首次 `poll()` 跑在 `pollTimers` 写入之前，`refreshTasks` 的
  `!pollTimers.has(t.id)` 守卫会判定「没人在轮询」再开一条循环，同一任务两套定时器抢写状态。
  现在进入即占槽再发请求，并加重入守卫；终态统一走 `stopTimer()`。
- **暂时性轮询失败不再静默**：404 / 429 / 5xx / 超时 / 网络异常此前被吞成 `{status:'pending'}`，
  界面只显示「等待中」，直到 40 分钟超时才知道失败。现在带 `transient` 标记与
  `transientDetail`（`HTTP {状态码}：{网关原文}`），界面显示「暂时无响应（连续 N 次）…」，
  连续 20 次提前标失败而非等到窗口上限。401/403 仍立即失败（KEY 问题不会自愈）。
- 新增 11 条回归用例覆盖 Bug A / D；全套 **430 用例**全绿，`tsc` 0 error、esbuild 通过、
  eslint 真实缺陷 0。

## 0.15.2 — 视频提交限流改为按 KEY 分桶

- **每把 KEY 各自一个 65 秒提交窗口**，不再共用一个全局窗口（新增 `src/core/videoKeySlot.ts`）。
  上游配额挂在账号上，换一把 KEY 等于换一个独立的桶；旧的全局单槽让 2 把 KEY 也排队等同一个 65s。
- **提交吞吐上限随 KEY 数量提升**：当镜间自然间隔低于 65s 时，2 把 KEY 约 2 倍、3 把（池上限）约 3 倍。
  串行「提交 → 轮询 → 下载」且单镜生成耗时 >65s 的流水线不受影响 —— 那种情况下本来就等不到窗口。
- **仍然严格串行提交**：只放宽「窗口归属」，不引入并行。任何时刻打到上游的提交请求数 ≤ 1。
  参考实现（Flask 版）也没做并行，上游究竟按「账号」还是「平台」限流并不确定，真并行在平台级限流下会立刻 429。
- **可一行退回旧行为**：`sharedBucket: true` 让所有 KEY 共用一个桶，等价于改造前的全局单槽。
- 401/403 换 KEY 不再跳过新 KEY 自己的窗口（省掉的是「为失败 KEY 退避」，不是限流窗口）；
  429/5xx 仍退避后重试**同一把** KEY，因为换 KEY 解决不了限流。
- 新增 `test/video-key-slot.test.mjs`（14 用例）；全套 **419 用例**全绿，`tsc` 0 error。

## 0.15.1 — 修复 v0.15.0 两处布局回归

- **Tab 栏不再是 10 根满宽横条**：v0.15.0 的 CSS 约定「一个 `.agg-tabs` 包两行 `.agg-tabs-row`」，
  TS 却建了两个 `.agg-tabs` 并把按钮直接挂上去，被 `flex-direction: column` 顶成满宽竖排。
  现在宽侧栏是一整行 10 个等宽按钮（回到 v0.12.0 的单行观感），窄侧栏（<600px）自动换行成两排各 5 个。
- **切 Tab 不再叠加面板**：`attachPanel()` 漏掉「先摘下上一个活跃面板」—— `appendChild`
  只挪走被追加的那个节点，旧面板一直留在 DOM 里，点几个 Tab 就把全部面板挤成一屏。
  修复后任一时刻只有一个面板可见；缓存语义不变，输入内容、选中模型、进行中的视频轮询都不丢。
- 全套 **405 用例**全绿；`tsc` 0 error。

## 0.15.0 — 面板缓存 + 全流水线打点 + 资产清单

- **切 Tab 不再丢输入、不再杀掉进行中的视频轮询**：面板改成槽位缓存（`slots: Map<TabId, PanelSlot>`），
  切 Tab 只做 DOM 迁移（`appendChild` 自动脱离旧父级），**不 destroy、不 new**。此前 `VideoPanel.destroy()`
  会 `pollTimers.clear()`，切一次 Tab 正在轮询的任务就永远停在「提交中」。
- **Tab 栏拆两排**：10 个 Tab 按使用频率分两组各 5 个，修 300px 侧边栏横向溢出。
- **三条流水线接入 trace**（短剧 / 红果 Pro / 数字人）：不逐个函数埋点，而是包住既有漏斗
  （`progress` 回调与 `ctx.log`），阶段推进、逐镜失败、哪一把 Key 报错从此可见。补上后又把
  **edge-tts / 视频下载重试 / 图片压缩降级 / 视频轮询异常** 四处的裸 `console.warn` 迁入 trace ——
  免费配音走 WebSocket，**完全绕过 HTTP 漏斗**，此前整条链路零打点。
- **新增 Vault 资产清单命令** `open-asset-index`：扫描插件产出的产物并写成可点击的笔记。
- **新增 trace 纪律闸门** `test/trace-discipline.test.mjs`：把「业务模块不得裸写 `console.warn`」
  这条约定变成硬检查，任何人再漏都会被 `npm test` 拦下。
- 全套 **405 用例**（17 个测试文件）；`tsc` 0 error、eslint 真实缺陷 0、发布校验 0 ERROR。

## 0.14.0 — 社区插件市场发布就绪

- **manifest 合规修复**：description 263 → 140 字符（官方上限 250，原值会被审核直接拒）；删除空 `fundingUrl`
  键（官方明令：不接受捐款时**必须删键**，不能留空串）；`isDesktopOnly` false → **true**。
- **isDesktopOnly 决策**：代码实际使用 `child_process`（ffmpeg.ts 调用 ffmpeg 子进程）与 `@electron/remote`
  （secureStore.ts 取 safeStorage 加密 API Key）—— 两者都是 Node/Electron API，按官方政策必须声明 true。
  移动端不再出现在可选列表里，这是诚实的取舍而非遗漏。
- **新增 LICENSE**（MIT）：仓库原本只有 package.json 里的 `"license": "MIT"` 声明而无文件，
  而社区插件审核强制要求「可检测的许可证」。
- **新增发布工具链**（零运行时依赖）：
  - `release/validate.mjs` —— 把官方《Submission requirements for plugins》与审核实际检查项固化为可执行闸门，
    含 description 长度/结尾/emoji/Obsidian 字样、id 命名、非法额外键、`isDesktopOnly` 与源码
    Node/Electron API 用法的一致性交叉校验、命令 ID 不得内嵌插件 id 等。
  - `release/zip.mjs` —— 纯 Node 实现的 ZIP 读写（store 方法），因为 registry 不可达装不了 archiver。
  - `release/package.mjs` —— build → 合规校验 → 打包 → 自校验，同时产出 `dist/assets/` 三个独立文件
    （审核器认这个）与 `dist/<id>-<version>.zip`（手动下载 / 第三方更新器用）。
  - `.github/workflows/release.yml` —— push tag 自动打包并发布 GitHub Release，含 Python 独立 CRC 校验。
- **新增发布手册**：完整发布手册，含官方要求逐项对照、首次提 PR 的步骤、后续版本更新流程、常见被拒原因，并明确标注哪些步骤只能人工完成。
  后续版本更新流程、常见被拒原因，并明确标注哪些步骤只能人工完成。

## 0.13.0 — 简化版短剧流水线断点续跑

- 简化版短剧流水线接入检查点持久化（`pipelineStore.ts`），跑一半关掉 Obsidian 重开可续跑。
- 新增 i18n 对称性测试，两个语言包 596 键强制对称。
- 全套 **380 用例**；修复 `listCheckpoints` 把 `adapter.list()` 的完整路径当 ID 用的生产 bug。

## 0.12.0 — i18n 国际化

- 6 个大型面板全部转 `T()` 调用，中英双语包各 580 键，设置页可即时切换。
- `src/i18n/index.ts` 零 obsidian 依赖（可 Node 单测），`bridge.ts` 是唯一 import obsidian 的文件。

## 0.11.0 — 视频轮询阶梯退避

- 阶梯退避 10s×12 → 20s×30 → 30s，窗口 40 分钟，设置页可配。
- 窗口按 `createdAt` 绝对计算（修面板重开重置计数、实测拖到 27 分钟才标失败）。
- 401/403 立即失败并给明因，不再空转。

## 0.10.0 — 阶段 E 工程化主体

- E3 批量出图 / E4 连通性测试 / E5 日志落盘 / E6 schema 迁移 / E7 API Key 加密（safeStorage）。

## 0.9.1 — MTV 作曲修复

- 修复 Gradio SSE 完成事件解析错误导致作曲永远失败（`event: complete` 的 data 是返回值数组本身）。

## 0.9.0 — MTV 一键成片 + 多参考图

- D3 MTV（ACE-Step 免 Key 作曲）；E2 多参考图参考板；P1 文本测试扩展。

## 0.8.1 — Agnes 视频轮询修复

- 修复 Agnes 视频生成永远转圈：轮询端点写成了不存在的路径，且漏读完成态的 `url` 字段。

## 0.8.0 — 运行期追踪与日志面板

- 新增 trace 模块（环形 1000 行）与日志 Tab；6 项体验修复。

## 0.7.0 — 数字人口播

- 阶段 D2：文稿 → 分段 → 配音 → 形象图 I2V → 段成片 → 字幕 → 成片。

## 0.6.0 — 红果短剧 Pro

- 阶段 D1：七阶段编排（剧本 → 分镜 → 定妆 → 首尾帧 → 视频 → 配音 → 成片），KEY 池轮换 + 断点续跑。

## 0.5.0 — 小说封面生成器

- 阶段 C：Canvas 裁 3:4 并排版书名作者名，替代原版 PIL。

## 0.4.0 — TTS 配音 + ffmpeg 成片

- 阶段 B：edge-tts 免费配音 + ffmpeg 拼接混音，短剧流水线能出成片。

## 0.3.0 — 账号池视频提交

- 阶段 A：打通本机免费额度池，视频生成成本趋近于 0。

## 0.2.0 — 视频生成增强

## 0.1.0 — 首次发布

- 图片 / 文本 / 视频 / 短剧基础版 / 设置。
