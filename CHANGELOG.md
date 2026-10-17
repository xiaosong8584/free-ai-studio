# Changelog

本文件是**发布说明的事实来源**：GitHub Release 的说明、社区插件市场的版本记录都以这里为准。
每版一屏内读完。

格式遵循 [Keep a Changelog](https://keepachangelog.com/)；版本号与 `manifest.json` 严格一致。

## 0.17.18 — 多国语言（8 种 VIP 增量语言包）（2026-10-17）

### Added — 日语 / 韩语 / 俄语 / 阿拉伯语 / 印地语 / 西班牙语 / 法语 / 德语

- 新增 8 个语言包（`src/i18n/<locale>.ts`），每个覆盖 **195 条核心 UI 键**
  （通用动作 / 通知 / 设置分区名 / 标签栏 / VIP 门禁 / 状态行 / 命令名）。
  设置 → 界面语言 下拉里选，切换即时生效、无需重启 Obsidian
- **渐进式翻译**：这 8 个包只翻核心键，其余键走回退链
  `locale → en-US → zh-CN → key`。回退链里**刻意插了一层 en-US** ——
  对日语/俄语用户，未翻译项显示英文比显示中文更好懂
- **阿拉伯语 RTL**：`RTL_LOCALES` 标记 ar-SA，切换后给**插件自己的视图容器**
  设 `dir="rtl"`（绝不动 `document.body`，避免波及整个 Obsidian 与第三方插件）
- 语言名在下拉里用**该语言的自称**（日本語 / Español / العربية …），
  用户是"认得自己的语言"来选的，翻译成中文反而找不到

### Added — VIP 门禁：多语言为 VIP 专属

- `gate.ts` 新增 `VIP_ONLY_LOCALES`（8 项）+ `isLocaleLocked(locale, vipActive)`。
  **简体中文 / English 永久免费**，非 VIP 的那 8 个下拉项直接 **disabled**，
  且下方有说明：不是静默失效，是明确告知
- 三重护栏（与 edge-tts 同款设计）：
  1. 设置页下拉 disabled；2. `onChange` 里再判一次（防脚本 / 旧配置绕过）；
  3. **运行闸** `bridge.resolveLocale` —— 挡住"升级前已把 `uiLocale` 存成 VIP 语言"的
     老配置，回退为跟随 Obsidian 并给出可行动提示
- 启动顺序调整：`initI18n` 从 `restoreVipState` **之前**挪到**之后** ——
  VIP 状态未知时没法判定语言能否选用；而命令名只在注册时解析一次，
  提前解析会让 VIP 用户的语言被误挡回且事后无法补救

### Changed — i18n 核心回退链

`lookup()` / `hasKey()` 从「当前 locale → zh-CN → key」改为
「当前 locale → **en-US** → zh-CN → key」。对 zh-CN / en-US 用户行为不变
（两个全量包键集合严格对称），只是给增量包多铺一级台阶。

### Tests — 8 项新增

增量包的三条**专属**失败模式（全量包靠"两侧键集合相等"就能抓，抓不到这些）：
键拼错（永远取不到）/ 占位符抄错（插值静默失效）/ 整包被误清空（覆盖量下限）。
外加：回退到 en-US 而非中文、VIP 清单两处引用一致、中英永不被锁、只有阿拉伯语是 RTL。

## 0.17.15 — explainer 成片质量专项优化（2026-10-16）

### Fixed — explainer BGM 旋律化重制（2026-10-16 第三轮）

用户反馈 BGM「没有旋律、像噪音」。根因是 pad 为纯持续长音（无音头、无音高走向，
人耳归类为机器嗡声），恒定 523 Hz 的 spark 更接近耳鸣音。本次重制声部结构：

- **旋律声部**：C 大调五声音阶（C5–C6）40 槽固定乐句（每槽 0.5s，含休止呼吸），
  音乐盒音色（基音 + 二次谐波，软起音 + 指数衰减 + 平滑收尾）。五声音阶对四个
  diatonic 和弦任意搭配都协和，chordOffset 轮转不打架
- **低音脉冲**：每和弦起点 + 半程各一个根音心跳（0.03s 软起音 + 指数衰减），
  给节奏支点
- **spark 移除发声**：恒定 523 Hz 正弦在人声下就是啸叫；字段保留（网格测试不变量
  仍覆盖），波形不再渲染
- **声部平衡**：旋律增益 3.0 / 低音 1.6 / pad ×4 分音 0.4→0.25——实测增益 1.1 时
  旋律音槽与休止槽 RMS 几乎无差（0.106 vs 0.099，被 pad 淹没），声部要站到前台
  必须显著大于 1
- **测试**：新增「旋律音阶频率在网格上」与「音槽 RMS 显著高于休止槽」（阈值 1.15，
  被淹没时实测仅 1.07）两条守卫；无缝循环/确定性/峰值归一化等 18 条原有不变量全过
- 频谱目视验证：旋律频带（500–1000 Hz）音符攻击点规律分布，循环点（20s/40s）无断裂

### Fixed — explainer 成片去雾 + text 镜恢复（2026-10-16 第二轮）

真实生产 run（27 镜 cine-violet）反馈"主体发毛模糊不清、不时尚"的修复：

- **风格锚点降雾度**：`cine-violet` 锚点文案去掉"柔和体积光，环境氛围"这类
  生雾词，改为"干净深色背景，克制体积光，高对比低雾度，简洁构图"
  （zh-CN / en-US 同步；只影响之后新生成的 run，已有静帧不追溯）
- **text 镜提示词**：`MG_TYPES` 中 `text` 挪至末位并补配额说明，
  非 text 镜必须含 `heroGlow` 的强引导（渲染侧配套 MgText 金句镜已实现）
- **测试**：explain-prompts 断言同步（drive→gpu 实体、fx 计数）
- 渲染侧（外部工程）配套：全局辉光收紧为紧裹 rim light（5 文件 34 处）+
  `MgSetPiece` 时序按镜头时长缩放（短镜主词/caption 不再缺席），
  详见该工程 `docs/05` 第 9 节

### Added — explainer MG 动效契约增强（2026-10-16）

对照 skill `/anything2explainer` 补强 remotion-explainer 渲染侧的联动契约更新
（渲染侧改动在外部工程 `I:\Components\remotion-explainer`，本仓库是契约的源头）：

- **MG 类型**：`MgType` 新增 `rail`（顶部流程轨：2–5 步胶囊 + 进度线，
  不给 `active` 按镜头前 75% 逐帧推进；步数 <2 清洗时丢弃）
- **显式运镜**：`MgData` 新增 `cam`（`MgCam`）——`zoom:[from,to]`（clamp 0.7–1.4，
  超 1.4 描边显粗）+ `panX/panY`（±180）；宽松清洗：写错整项忽略不牵连 mg
- **语义实体白名单**：`MG_ICONS` 30→43，新增 13 个可点名实体
  （doc/db/chunk/llm/gpu/rack/person/code/repo/gauge/cluster/timeline/trap），
  size≥90px 时渲染为有内部结构的语义图元而非通用 path
- **fx 体系**：`MG_FX` 9→10，新增 `dirBlur`（各向异性方向模糊，入场 14 帧解焦到 0）；
  提示词 fx 清单 / 用法说明 / 推荐组合同步
- **幽灵 fx 治理**：`bigNumber`（metric 默认注入）与 `ghostText` 此前在
  白名单但渲染侧无消费点（模型点名即静默丢弃），本次由渲染侧真正消费

## 0.17.17 — MTV 转免费 + edge-tts 转 VIP（2026-10-17）

**收费范围调整**（`src/vip/gate.ts` 是唯一权威清单，两份 README 的 Disclosure 已同步）：

- **MTV 由 VIP 转免费**：`mtv` 移出 `VIP_ONLY_TABS`，非 VIP 用户可直接看到并使用
  MTV 面板（ACE-Step 作曲 / 静帧特效成镜 / 本地音轨全部开放）。注：MTV 标签
  **不在默认可见标签里**，需到设置 → 标签可见性勾选一次（此前被锁，勾不了）
- **edge-tts 引擎转 VIP**：新增 `VIP_ONLY_TTS_ENGINES` + `isTtsEngineLocked()`
  （引擎级门禁，不动整个配音 Tab）。非 VIP 的表现是**明确禁用**而非静默失效：
  - 设置页「默认引擎」与配音面板引擎下拉的 edge 选项 **disabled**（文案带 🔒）
  - 设置页 **edge 参数组整组禁用**（与 Remotion 等 VIP 分区同款，附锁定提示）
  - 运行闸：升级前已把默认引擎存成 edge 的非 VIP 用户，生成时被拦下并提示
    改用豆包（需 Key）或 Audio8（本机部署，免费），不再合成到一半才报错
- 免费替代路径**必须留着**（测试用例钉死）：`doubao` / `audio8` 对非 VIP 永不被锁

### 说明
- 两处 i18n 文案同步：设置项 `edge-tts（免费）` → `edge-tts（VIP）`
- 回归：`test/vip-ad.test.mjs` 新增 2 例（MTV 转免费 / edge-tts 引擎锁 + 免费替代不被锁），
  25 例全过；`npm test` **1612/1612**

## 0.17.16 — MTV 项目制与成镜双模式 / ZeroGPU 额度兜底 / 成片回验（2026-10-17）

### Added — MTV 项目制：一键创作 + 静帧特效成镜双模式（2026-10-16）

MTV 面板重做为**项目制**（七大分区：项目条 / 一键创作 / 参数 / 成镜模式 /
生成 / 进度 / 资产），配套四个新模块：

- **一键创作**（`mtvCreate.ts` 纯函数）：创作意图 → LLM → 歌词 / 曲风 / 时长 /
  质量自动填充。**时长不靠 LLM**——`estimateDurationSec()` 按歌词行数 × 4.2s
  确定性估算（LLM 数数不可靠）；必填字段缺失整份丢弃并映射到 `mtv.create.bad*`
  文案；可勾选「创作后自动生成成片」全流程接跑
- **项目制存储**（`mtvStore.ts`）：`{outputFolder}/mtv/projects/<id>.json`
  走 `vault.adapter`（中间物料不进 Vault 索引）；参数改动 800ms 防抖落盘、
  生成前强制 flush；**删项目只删 json，不动资产目录**（歌曲/成片是花钱产物，
  与短剧 `removeCheckpoint` 对齐）
- **静帧特效成镜双模式**（`mtvStill.ts`）：面板开关切 I2V / still。still 走本地
  ffmpeg Ken Burns 推拉，每段按时间窗**真实时长**生成 → 拼接天然等于歌长，
  顺带消除 I2V 对长歌的十倍慢放问题；参数与 anchor 模块 `kenBurns()` 完全一致
  （四效果轮转 / zoompan 具体宽高 / 显式 fps=24 / 失败退静态图），各段参数一致
  → concat 走 `-c copy` 秒级拼接。agnes 视频限流时的降级路径，新项目默认开
- **资产区分组**（`planAssetGroups` / `mtvAssetSummary` 纯函数）：按歌曲名分组
  的缩略图卡片墙，逐段关键帧生成完即时出现，每张卡带重出 / 预览 / 删除
- **进度模型**（`mtvStages.ts` 纯函数）：ETA 预估 / 阶段步骤条 / 计时行
- **`onPersist` 回调**：每落一个资产（歌/帧/镜头/拼接/成片）回调一次 → 面板
  即时写 json + 刷资产区，**半途崩溃不丢已花钱的成果**

### Added — ZeroGPU 额度用尽：三道兜底（2026-10-17）

截图反馈「ZeroGPU 额度用尽」。根因是 ACE-Step 走 HuggingFace ZeroGPU，
匿名请求共享**全网 2 分钟/天**的池子（最易被抢空），且额度按「当日首次 GPU
使用后 24 小时」重置——长退避无意义。三道防线：

- **① Token 鉴权**：设置页填 `hf_` Token 后请求带 `Authorization: Bearer`，
  额度记个人账号（免费 5 / PRO 40 分钟 / Team 60），不再抢匿名共享池
- **② 端点池故障转移**：设置页可填多个 Space 地址（逗号 / 分号 / 换行分隔，
  非法项忽略，全非法回落内置端点）。失败按 `quota` / `rateLimit` / `waking`
  分类：额度类直接换端点，**退避刻意做短**（5s / 15s，不睡几十分钟），
  全失败才抛**可行动错误**（明确指引填 Token / 加端点 / 用本地音轨）
- **③ 本地音轨直通**：MTV 面板新增「本地音轨」行（`AudioPickerModal` 选 Vault
  内音频），完全跳过在线作曲直接合成 MV；`readVaultAudio` 只 `readBinary` 拷
  字节，**不写用户原文件**
- 8 条纯函数单测（端点解析 / 鉴权头 / 失败归类 / 退避 / 可行动错误）

### Added — 成片回验 renderVerify：参数 / 帧覆盖 / 坏帧（2026-10-17）

「日志成功 ≠ 画面正确」——成片要能被机器校验，新增 `src/core/renderVerify.ts`：

- **三项检查**：① ffprobe 参数回验（fps / 时长 / 可选宽高 / pix_fmt，
  实测缺失项跳过不误报）；② 帧覆盖断言（场景时间轴 → 帧区间，逐场景抽帧，
  断言每场景真的进了片）；③ 全片坏帧扫描（`tblend=difference` +
  `signalstats` YAVG，超阈值段合并成一处）
- **接入 `renderMedia`**：在 `child.on('close')` 成功分支、确认产物存在之后
  `resolve` 之前 `await verifyRenderedRender(...)`。口径遵循 doc/43：**回验不
  阻断主流程**（产物已生成就不因回验翻车），自身异常吞掉，只 trace + Notice
- **真机 E2E**（`npm run test:verify-live`）：ffmpeg 7.1.1 合成纯蓝 480×270@24
  成片，再 `drawbox` 插一帧纯黑实测 —— clean 逐帧亮度差峰值 **0 / 0 处坏帧**，
  corrupt 峰值 **25 / 检出 1 处（帧 23，单帧，1000–1042ms）**；两份真实
  metadata 已固化成 `test/fixtures`，不依赖运行时 ffmpeg 进 CI
- **设置项**（新增免费分区「成片回验」）：总开关 + 坏帧扫描单独开关（它是三项
  里唯一「重」的，全片解码、高运动画面可能误报，允许只关它）+ 期望宽高
  （填 0 = 跳过该项）
- **修复**：`detectFrameCorruptions` 原实现把进入帧与「第一个」高差异帧配对，
  **连续坏帧只报第一帧、后续漏报**；改为扫到连续高差异运行的最后一帧，整段
  合并成一处

### Fixed — VIP 命令与标签锁不一致 + Disclosure 补全（2026-10-17）

- `main.ts` 把 `open-cover` 注册成免费命令（`addLocalizedCommand`），但
  `cover` 标签在 `gate.ts` 仍被 `isTabLocked` 锁、且已在 `VIP_ONLY_COMMANDS`
  里——非 VIP 点命令被静默弹回。改为 `addVipCommand`（弹激活窗，与其它 VIP
  命令一致）。测试不变量从 `[broll, inkos, canvas]` 扩到全部 **7 个有命令的
  VIP 标签**，防止再被改坏
- 两份 README 的 VIP Disclosure「免费面板」枚举漏了 **短剧（drama）** 与
  **画廊（gallery）**，已补齐；VIP 表 8 行与 `VIP_ONLY_TABS` 逐条核对一致
  （`gate.ts` 注释要求改收费清单必须同步 README，否则社区目录审查会打回）

### Unreleased

### Added — explainer 分镜多样性 + 自动运镜（2026-10-16 第四轮）

针对真实成片「PPT 翻页感」（主体相机近乎静止）与「元素缺失」（65 镜里
flow/compare/rail 等结构化类型几乎为 0）两项短板：

- **运镜断链修复（本轮关键）**：`ExplainPanel.buildRemotionProps` 此前透传了
  `mg` / `transitionIn` 却漏了 `motion` —— 分镜卡上模型选的运镜从未进入渲染
  props，Remotion 侧永远全默认「缓慢推近」。本次补上 `motionByIndex` 映射与
  `RemotionShot.motion` 字段，运镜链路真正闭环（渲染侧配套改动见外部工程
  `docs/05` §10）
- **提示词自审动作**：分镜提示词（中/英双语）加输出前自审 —— 结构化类型
  ≥2/10 镜、text/icon 不超 50%，信息密度高的讲解镜强制改成结构化类型
- **`auditMgDiversity()`**：机器可校验的分镜丰富度审计（mg 类型直方图 /
  structured 计数 / textIconRatio / warn），把提示词里的配额规则变成可断言数据；
  +5 单测（压线值 / 无 mg 镜 / 空数组边界）
- **上述审计已真实接入渲染前质检门（本轮补上的关键一环）**：
  `explainQc.runExplainerQc` 新增 warn 级 `mgLowDiversity` 检查项。此前它
  是「写好 + 单测全绿 + 管线零调用」，短板「65 镜里结构化类型几乎为 0」
  无人拦。现生成完分镜即出预警，消息带结构化类型数、text/icon 占比与类型
  直方图，并提示「可重生成分镜，或把信息密度高的讲解镜改成结构化类型」。
  口径：**warn 级不阻断渲染**（守住「只报不改」纪律）；带 mg 的镜头少于 3
  镜不报（极短片按比例判必然误报）；`total` 只统计带 mg 的镜头，不等于
  全片镜数。+4 单测（触发 / 不触发 / <3 镜 / 全无 mg + 断言 warn 不改 ok）
- 渲染侧（外部工程）配套：`buildCamKeys` 跟随卡片 motion 派生主体相机
  （同向视差，幅度 ≤1.08），真实 run 复渲抽帧 + 差分图确认缩放生效

### 待办
- **未在 Obsidian 真实界面验证 v2 全链路**（最高优先）：目前只在 CLI 探针
  （`npm run test:tts-live`）层验证过，Obsidian 内的设置面板 → 生成 → 落盘全链路未跑。
  这是「保证功能可用、用来项目测试」前最后一块未验证的拼图。
  （面板层的代次展示逻辑已由 `npm run test:tts-live` 的探测段覆盖，
  但**按钮点击 → DOM 渲染**这一段仍未跑过。）
- **三个新面板（B-roll / InkOS / 无限画布）未在 Obsidian 真实界面点击验证**：
  纯逻辑层有单测、ffmpeg 滤镜链有真跑验证，但「按钮点击 → DOM 渲染 → 落盘」没跑过，
  建议与上一条合并为一次真机验收。
- ~~**降级路径无法自动化测试**~~ → **0.17.10 已解决**：
  `tts.ts` 暴露 `resetGecVersionForTest()` / `currentGecVersion()` /
  `currentGecDowngradeReason()`，集成测试新增 4 例覆盖代次迁移与复位。
  这三个接口不依赖 v2 实现，v2 摘除后仍保留（护栏 B2c 会检查别只留一半）。
- ~~**摘除流程有护栏但没脚本**~~ → **已交付**：`npm run remove:gec-v2`
  （默认 dry-run，`--execute` 真摘除），自验证见
  `test/gec-removal-script.test.mjs`（8 例）。**仍待人工的两件事**：
  ① 保留的 `gec-integration.test.cjs` / `gec-version-preference.test.cjs`
  里演练后又长出的 v2 时代断言需人工分诊（删 v2 专用例 / 改 v1 期望，
  脚本只打失败清单不替人改写测试语义）；
  ② `applyGecVersionPreference('2')` 摘除后语义未定（夹回 `'1'` 还是从
  settings / i18n / schema 摘掉该选项）。详见 doc/40 §5.5。

---

## 0.17.14 — 墨影 AI 工场移植：剧本质量工程 / B-roll / InkOS 写作台 / 无限画布

**主题**：把桌面端「墨影 AI 工场 v1.0」里插件缺失的能力搬进 Vault。四个模块分两条线落地 ——
**剧本质量工程**走免费档（挂在文本面板，任何人可用），**B-roll / InkOS / 无限画布**
走 VIP 档（增量增强，不挤占免费功能）。同版交付 GEC v2 一键摘除工具（见「工程工具」）。

### Added（免费档）

- **文本面板新增 6 个剧本质量模式**（`src/core/scriptQuality.ts`，永久免费）
  - `剧本体检` / `剧本改写` / `70 集拆分` / `锁定码设计` / `八步分镜` / `首尾帧继承`。
  - 全部为**去题材化**系统提示词：只保留结构铁律（单镜时长、台词字数、语速闸门），
    不含任何特定作者风格或角色卡，换题材即用。
  - 时长/字数/集数上限走**常量单一来源**（`SCRIPT_SHOT_SEC` / `SCRIPT_CHARS_PER_SEC` /
    `SCRIPT_SPLIT_EPISODES` …），提示词文本由模板插值引用同一批常量；
    测试反向把数字从提示词里抠出来比对，防止「改了常量忘了改提示词」。
  - `parseCheckVerdict()` 在模型**没给出判定行**时保守返回 `fix`（不是 `pass`）——
    宁可让用户再看一遍，也不能把没体检过的剧本当合格。
  - 文本面板为此加了**第二输入框** `extra`（如「目标集数」「改写方向」），
    只在需要的模式下显示，不增加其他模式的界面负担。

### Added（VIP 档）

- **B-roll 背景视频**（新 tab `broll`，五阶段幂等流水线）
  - 素材 → 计划 → 分镜 → 关键帧图 → 片段，任一阶段可单独重跑且结果落盘可续。
  - **角色三步**（建档 / 定妆 / 定锚）：先立人物卡，再定视觉锚点，最后才生成，
    避免「每镜一张脸」。
  - **自动分幕（零模型）**：`src/core/brollAlign.ts` 用 ffmpeg `silencedetect` 切出静音段、
    取反得人声段，再按字数加权把文稿摊到各段上 —— **不下载任何语音识别模型**。
  - **抽帧联系表**：`src/core/brollFinish.ts` 拼 ffmpeg 滤镜链一次吐出网格图，
    用于「一眼看完所有镜」而不必逐个点开视频。
  - **合成成片**：`concatVideos` → `fitSegment`（音乐**绝不** atempo 提速，
    画面冻结末帧去迁就音乐长度）→ `burnSubtitle`（保留原音轨）；字幕复用既有纯函数
    `subtitle.ts`，不新造一套折行逻辑。产物落 `<outputFolder>/broll/`。
- **InkOS 写作台**（新 tab `inkos`，Vault 原生小说工程）
  - 章节列表 + 资产库 + AI 续写 + 章节→分镜，全部落 Vault 文件，不依赖外部服务/iframe。
  - **「送入红果 Pro」**：把章节分镜 JSON 直接交给红果 Pro 七阶段流水线，
    打通「写 → 拍」两头的断点（此前只能手抄）。
  - 跨项目资产复用 `collectLibrary` / `libraryToSketch` / `libraryToProAssets`：
    旧项目里攒的角色/场景/道具能拎出来给新项目用。
- **无限画布**（新 tab `canvas`，节点图编排）
  - 文本 / 图片 / 视频三类节点 + 依赖连线 + 并行生成 + 视频节点**首帧反向继承**。
  - `canLink()` 禁自连 / 重复连 / 悬空连 / **成环**（成环会让依赖永远等不到上游）。
  - `sanitizeNode()` 对脏数据兜底，坏节点不会让整张图打不开。

### Changed

- 三个新 tab **不在**全新安装的默认可见集合里（默认仍是 6 核心），
  需到设置页手动开启 —— 避免标题栏一次性塞 15 个按钮。老用户勾选不受影响。
- i18n 双 catalog（zh-CN / en-US）新增 `text.mode.*` / `text.extra.*` /
  `broll.*` / `inkos.*` / `canvas.*` / `tab.*`，键集合对称由 `test/i18n.test.mjs` 卡。
- ** lint 真实缺陷 114 → 80**（新文件贡献的 34 条已清零）：
  - `BrollPanel` / `CanvasPanel` 的 `document.createElement` 全改 **`activeDocument`** ——
    popout（弹出窗口）里 `document` 指向主窗口文档，用它建的元素**挂到错误文档、肉眼不可见且不报错**。
  - 两个新面板的原生 `confirm()` 换成 Obsidian `Modal`，并把 `AnchorPanel` / `DramaProPanel`
    里各抄一份的确认弹窗**抽成 `src/ui/confirmDialog.ts` 的 `confirmModal()`**（四处漂移 → 一处定义）。
  - 静态样式直写改 `setCssProps()`；两处浮动 Promise 补 `void`；`JSON.parse` 的 `any` 收敛为 `unknown`。
- 新增 `test/ui-hygiene-invariant.test.mjs` 把这三条钉成契约：全库禁裸 `document.*`、
  禁原生 `confirm`、`confirmModal` 只允许定义一次且 `private confirm()` 必须是一行转发。

### Added（工程工具）

- **GEC v2 一键摘除脚本**（`npm run remove:gec-v2`，`scripts/remove-gec-v2.mjs`）：
  把 doc/40 §5.5 演练出的手动流程（删 5 文件 + `tts.ts` 4 处 + `package.json`
  三处）固化成一条命令。默认 dry-run 只打印计划；`--execute` 才动手，
  且 9 条 tts.ts + 2 条 package.json 替换规则必须各自唯一命中，任一漂移
  即整体放弃（一个字节都不改，防止半摘除态）；git 安全网 + 非 git 仓库
  拒绝 + `--force` 先建备份；幂等。摘除后自动跑静态扫描 + 护栏 + tsc +
  npm test，并按后果分级输出「剩余人工事项」。
- `scripts/check-gec-consistency.mjs` 导出 `runChecks(root)`（不打印不退出），
  供摘除脚本进程内直调；CLI 行为与退出码不变。
- 测试 1494 → **1502**（+8，`test/gec-removal-script.test.mjs` 已入测试链）。

### Fixed（护栏）

- 两处**摘除态误报**：B3 只匹配字面量 `'1'`，而 0.17.13 起默认值走
  `DEFAULT_GEC_VERSION` 常量，会把完好的摘除态误报成 BLOCKER（与 A4 同款坑，
  改为常量路径 + 字面量两条都认）；B2b 数 `gecVersion` 子串会把按设计保留的
  `gecVersionLocked` 与探测结果展示字段算成僵尸参数（改为只认合成路径上的
  三种形态）。护栏自己变噪声后，真问题就会被淹没 —— 这两条都是那个模式。
- 一处潜在崩溃：`package.json` 不合法时会在 JSON.parse 的 catch 里访问
  尚未初始化的 `problems`（TDZ ReferenceError），把「报告 JSON 坏了」本身
  变成第二次崩溃。声明已移到 catch 之前。

### 说明

- 本版同时给 `src/vip/gate.ts` 的两个清单补了三项（tab + 命令双重锁定），
  并同步了 README 的 VIP Disclosure 表 —— 门禁清单与文档**必须一起改**，否则会漏披露。
- 未完成事项见上方 `## Unreleased → 待办`。其中**最高优先**仍是
  「在 Obsidian 真实界面验证 v2 全链路」——目前只在 CLI 探针
  （`npm run test:tts-live`）层验证过，设置面板 → 生成 → 落盘未跑。
- 本版新增的三个面板同样**未在 Obsidian 真实界面点击验证过**（纯逻辑层有单测覆盖、
  ffmpeg 滤镜链有真跑验证），建议与上一条合并为一次验收。

---

## 0.17.13 — edge-tts 协议代次可手动锁定（自动 / v1 / v2）

**主题**：设置 → 配音 → edge-tts 组新增「协议代次」下拉。默认「自动」= 先试 v2、
被服务端 403 拒绝再退回 v1（原行为不变）；手动锁定某一代后**不再自动降级**。

### Added
- **设置项「协议代次」（自动 / 锁定 v2 / 锁定 v1）**（`settings.ttsGecVersion`）
  - 给两类场景：① 想确认某一代到底能不能通 —— 自动降级会把「v2 被拒」变成
    「静默退 v1 后成功」，结果被掩盖；② v2 有授权风险，想冻结现状只跑 v1。
  - 锁定后 403 **不降级**，改打一条明确日志告诉用户「你锁的是 v2，它被拒了」，
    而不是偷偷换成 v1 让人以为锁定生效。
  - 连通性检测结果会追加 `[已手动锁定 v2]` 徽章 —— 锁定期间若失败，不必翻控制台
    才知道锁的是哪一代。
  - 改动**立即生效**，不必重启 Obsidian（这类藏在协议层的配置最容易被误判成「没生效」）。
- **`tts.ts` 新增 `applyGecVersionPreference` / `isGecVersionLocked`**
  - `'auto'` 会**复位**回内置默认代次。只解锁不复位的话，用户从 v1 切回「自动」
    后当前代次还停在 v1，行为与 auto 的语义不符。
  - 但**已因 403 降级过的**不会顶回 v2 —— 否则连通性检测会
    「失败 → 重置 v2 → 再失败」地抖。区分「用户主动切 auto」与「已降级态」两种来源。
- **schema v8 + `migrateV8`**：非法代次一律夹回 `'auto'`。这个值直接决定
  `generateSecMsGecToken()` 走哪条签名路径，写歪成 `'3'` 既不报类型错也没有分支接，
  会**静默按 v2 发**——用户以为锁了代次，实际根本没锁。

### Fixed
- **GEC 一致性护栏 A4 误报**（`scripts/check-gec-consistency.mjs`）
  - 默认值改由 `DEFAULT_GEC_VERSION` 常量承载后，旧正则
    `/gecProtocolVersion:\s*GecVersion\s*=\s*'2'/` 把「v2 路线完好」报成 BLOCKER。
    **护栏自己变成噪声后，真正的 v2 缺失就会被忽略** —— 这是比误报本身更坏的后果。
    现在两条路径都认（常量路径查常量值 + 初始值引用它；字面量路径仍放行）。
- **新增护栏 B2d**：`applyGecVersionPreference` 与锁定守卫必须成对存在。
  只留设置项不接运行时 = 用户改了没反应；只留守卫不留 apply = 守卫永不生效。
  两者都是「tsc 过、构建过、真机才发现设置是假的」的类型。

### Testing
- 新增 `test/gec-version-preference.test.cjs`（19 例）：迁移兜底、锁定/解锁语义、
  降级守卫、锁定态签名确实跟随模块级代次、auto 不顶回已降级态、i18n 中英对称。
- 为覆盖降级守卫新增测试钩子 `tryGecDowngradeForTest`：守卫藏在 `synthChunk` 的
  catch 里，触发它要真建 WebSocket 并撞回 403 —— 没网就永远测不到，
  而这恰恰是「手动锁定会不会被自动降级覆盖」的唯一判定点。
- 测试 1347 → **1366**（+19），0 fail。
- **反向破坏验证 6 项全部精准命中**（含「切回 auto 不复位代次」「锁定态仍降级」
  「migrateV8 不兜底」「启动不应用」「下拉不立即 apply」「复位函数不清锁定态」），
  还原后全绿。第一轮跑出 1 项 MISSED，据此补了降级守卫用例才达到全命中。

---

## 0.17.12 — 解说词「写太长」被误判为达标（129% 显示绿色 ✓）

**主题**：用户实机截图报「生成分镜有问题」。日志里那行
`模型返回 435 字 → 39 镜 / 154.3s = 目标 129%` 后面跟着的是
`→ 时长在容忍范围内（129%）`，且是绿色 `tOk` —— 但 129% 远超 ±15% 容忍带。

### Fixed
- **时长诊断缺上界盲区：写太长一律报成「在容忍范围内」**（本轮主 bug）
  - `ExplainPanel.reportDurationDiagnosis` 的 verdict 分支自己写了个**只有下界**的
    `ratio < 100 - EXPLAIN_NARRATION_DURATION_TOL * 100`，else 一律「在容忍范围内」。
    数值探针实测：**116% / 120% / 129% / 150% / 200% / 400% 全部落进 else**，
    越离谱越被说成达标。
  - 更糟的是**同一屏两套结论**：分支下方的 `judgeNarrationDuration` 判
    `outOfBand=true` 并弹 Notice（⚠ 偏差 +29%），于是用户同时看到
    「绿色 ✓ 达标」与「⚠ 报警」，且日志那行还是绿的。
  - 根因是**判定源被复制成两份**（一份在方法开头有上下界，一份在分支里只有下界）。
    现在统一由 `judgeNarrationDuration` 判定，日志颜色 / 面板 / Notice 三处全部消费它；
    分支里不再出现任何基于容差常量的阈值。
- **新增「原因 D：写太长」分支**：与「原因 B：写太短」分开，二者处置完全相反
  （B 要补参考料，D 要删字或调大目标片长）。
  - 文案只陈述**可核对的事实**（写了多少字 / 上限多少 / 超百分之几 / 镜头数是否也超），
    不编机制。原因见下条——本轮试过两个归因都被实测推翻。
  - 新增 Notice `explain.durationTooLong`，写太长时不再复用「写太短」的文案
    （旧文案会引导用户去补内容，而写太长的正确操作是删字）。

### 归因修正：129% 的主因是「字数超预算」，不是「块太碎」
本轮在写「原因 D」文案时连着错了两次，均被数值探针推翻，结论记在这里防止再犯：

| 假设 | 受控实验 | 结论 |
|---|---|---|
| 块太碎（39 块太多） | 字数**恒定 435**，只扫块数 22~42：144.0 / 148.8 / 134.5 / 142.0 / 144.0 s | ❌ 块数与时长**不单调**，39 块比 22 块还短 |
| 逗号太密（每块塞标点） | 每块 0/1/2/3 逗号 → 172.7 / 164.9 / 157.1 / 149.3 s | ❌ 逗号会被 `STRIP_CHARS` 去掉且不计字数，加逗号反而让块变少、时长**下降** |
| **字数超预算** | 435 字 ÷ 3.3139 字/秒 = **131.3s** > 120s 目标；且 435 字超 398 字预算上限 **9%** | ✅ 光朗读就已超时，与块数、标点无关 |

> 定则：**归因必须来自受控实验，不能靠「看起来合理」。**
> 两次错误归因都发生在「只读代码 + 读日志」之后——日志里的 39 镜太显眼，
> 很容易顺势归因到「块多」，但没做「固定字数、只扫块数」的对照就下结论必然出错。

### Added
- **分镜缺口诊断**：模型只给部分镜头卡时（原逻辑 `if (drafts.length > 0) break`
  —— 只要解析出 1 张就收工），现在核对镜头总数，缺口会
  ① 打印 `got/want/missing + finish_reason` 并触发重试，
  ② 两轮仍不齐时在**分镜组面板上**以 `is-error` 样式显示「模型只给了 N/M 个镜头的分镜卡」。
  此前缺口只表现为镜头表里一行行「缺画面提示」，用户无法判断是模型漏镜还是插件坏了。
- 新增测试 `test/explain-duration-verdict.test.mjs`（9 例，已挂入 `npm test`）：
  钉死 verdict 三态分流、判定源唯一性、分镜缺口两条路径、i18n 中英对称。

### 清理
- 删除孤儿 i18n key `explain.retryStoryboard`（已被 `retryStoryboardPartial` 取代，
  留着会让人以为那条路径还在）。

### 测试
1338 → **1347**（+9）。0 fail。tsc / 构建 / 产物内容校验通过。
反向破坏验证 5 项全部精准命中（含「恢复单侧阈值」「用文案前缀反推颜色」
「恢复 ≥1 张就 break」「面板不显示缺口」「改回块太碎归因」），还原后全绿。

---

## 0.17.11 — 切引擎后解说词音色下拉不跟随（面板收不到设置变更）

**主题**：用户在设置面板把 TTS 引擎切到 edge-tts 后，解说词面板里的音色下拉
仍列着上一个引擎的音色。配置生效了，**面板没跟上**。

### Fixed
- **解说词音色下拉不跟随 `settings.ttsProvider`**：面板实例跨 Tab 切换长期存活
  （`StudioView.attachPanel` 只迁移 DOM、不重建），而 `ttsProvider` 在全局 settings 里，
  改它**没有任何面板级通知**。`rebuildVoiceOptions()` 只在 `display()` 与载入档案时调用，
  `switchTab` 走的却是 `attachPanel` —— **不调 `render()`**，面板无从重算下拉。
  - 表现最糟的一种是 audio8 → edge：下拉里留着 `zh-female-qn-qingse` 这类 audio8 音色名，
    而 `synthVoice()` 在 edge 分支**原样返回** `this.voice`（不做归一化），
    用户真选了就会合成失败。
  - 修复：`PanelSlot` 新增 `onShow` 钩子，`attachPanel` 在复用槽位时调用；
    `ExplainPanel` 记住下拉是按哪个引擎渲染的，漂移就只重建这一个下拉（**不重建整个面板**）。
  - 方法幂等：引擎没变时直接返回，用户反复切 Tab 不触发任何重渲染；
    首帧（`voiceRenderedFor === ''`）豁免判定，避免一次无谓重建。
  - `createSlot` 里局部声明的 `onShow` 必须 `return` 出去 —— 漏了局部钩子永远不存在，
    且类型上不会报错（可选属性）。

### Changed
- `doc/08-ui-components.md` 新增「面板收不到设置变更 —— `onShow` 挂载自检」：
  含槽位缓存的必然后果、真实案例、两种修法（A 设置变更事件 / B 挂载自检）的取舍，
  以及排查口诀（grep 面板里 `plugin.settings.<字段>` 且该字段在设置页可改）。
- 排查过同类面板：`AnchorPanel` 恒用 edge 音色表（不按引擎分支），
  `WandancerPanel` / `PipelinePanel` 读的设置字段无对应控件 → **只有本处需补**。

### Added
- `test/explain-voice-engine-sync.test.mjs`（5 例，纯 UI 无 DOM环境，
  沿用 `explain-panel-ui.test.mjs` 的源码结构断言）：
  钩子声明 + 挂载时无条件调用（且在槽位创建分支之外）+ `createSlot` 透传 +
  漂移判定读全局 settings + 幂等守卫顺序。
  **已做反向验证**：分别破坏「移除 onShow 调用」「return 漏字段」「移除幂等守卫」，
  三次都精确让对应用例变红，证明不是空转。

### 验证
- tsc 0 错 · 全量 **1338 pass / 0 fail**（+5）· 生产构建 `free-ai-studio@0.17.11`
- 护栏自洽 · 反向破坏测试三次均精准命中（not ok 1 / 2 / 4），还原后 5/5

---

## 0.17.10 — Edge-TTS 协议代次可视化（降级不再静默）

**主题**：0.17.9 把 v2 签名接上了，但**代次是不可见的**——握手 403 会把
`gecProtocolVersion` 从 `'2'` 降到 `'1'` 且本次会话不可逆，而设置面板对此毫无提示。
本版让「当前在用哪一代」成为面板上看得见的事实。

### Fixed
- **「v2 被服务端拒绝」与「网络不通」在界面上完全一样**：
  `probeEdgeTts` 原本只返回「直连正常，返回 34 KB 音频」，不报代次。
  于是最常见的真实故障（v2 被拒 → 降级 v1 → v1 也失败）显示为
  「❌ 合成失败」，用户只能反复换代理去试，方向从一开始就是错的。
  现在成功/ 失败两条路径都回报 `gecVersion`，降级时附
  「这不是网络问题，重试代理无效；重启 Obsidian 可回到 v2」的明确指引。
- **探测的环境短路分支漏字段**：给返回结构加 `gecVersion` 时只改了 try/catch，
  漏了 `if (!hasNodeRuntime())` 的提前 return —— 移动端点检测唯独缺代次，
  而它恰恰最需要（移动端不发握手，代次来自上次降级状态）。真机跑一次才发现。
- **护栏的接口存在性检查静默失效**：新增的「三个可观测性接口是否齐全」用
  `countOf(fn) === 0` 判存在性，而 `probeEdgeTts` 里的调用点也被计入子串，
  **定义被删后计数仍非零** —— 检查在它最该生效的场景下完全不报。
  改为匹配函数定义后，构造的反例能正确报出。

### Added
- **`currentGecVersion()` / `currentGecDowngradeReason()`**（`tts.ts`）：
  读当前代次与降级原因。`gecProtocolVersion` 是 `export let`，
  打包成 CJS 后外部拿到的只是绑定、看不到变化，必须走函数。
- **`resetGecVersionForTest(version?)`**：测试用复位口。
  **顺带解除了「降级路径无法自动化测试」的历史限制** ——此前只能靠真机撞 403
  才能走到 `synthChunk` 的降级分支。
- **`TtsProbeResult.gecVersion` / `.gecDowngradedFrom`**，面板据此显示
  `✅ 直连正常，返回 34 KB 音频（约 5.8 秒） · 协议 v2`，
  降级时额外挂 `[已降级至协议 v1]` 徽标。
- **集成测试 +4 例**（13 → 17）：代次迁移、复位有效性、
  降级后 token 形态、URL 前缀跟随。
- **`test/edge-tts-live.mjs` 增探测段**：面板走的是 `probeEdgeTts` 而非
  `synthesizeEdgeTts`，这条路径只有真机能验（集成测试无网络，跑不到握手）。
  现在合成与探测两条路径一次跑完。
- **护栏 B2c 检查**：三个可观测性接口「别只留一半」。
  它们**不依赖 v2 实现**，摘除后应保留 ——
  只留 getter 不留复位口则集成测试无法复位，只留复位口不留 getter 则面板无从展示。

### Changed
- 护栏问题分级提示行补上 §5.6 索引。
- `doc/40-edge-tts-v2-and-compliance.md` 新增 §5.6「代次可观测性」
  （含「为什么不加 v1/v2 下拉」的三条理由）与 §6.9 / §6.10 两条新坑。
- 踩坑记录 9 条 → **11 条**。

### 不变的设计决定
设置下拉**仍然只有 `edge` / `doubao` / `audio8` 三项**，不加 v1/v2 选项：
那是**引擎路由**，与协议代次正交；且v1 已被微软拒绝，
给一个必然失败的选项没有意义。详见 `doc/40` §5.6。

### 验证
- tsc 0 错 · 全量 **1333 pass / 0 fail**（+4）· 生产构建 `free-ai-studio@0.17.10`
- 护栏双态实测：v2 存在态自洽；构造的摘除态反例报出 2 BLOCKER + 6 WARN
  （新增的第 6 处即「缺 `currentGecVersion` 定义」）
- 真机 `npm run test:tts-live`：合成 34848 字节 MP3（`ff f3` 帧同步 ✓），
  探测 `ok: true` / `gecVersion: 2` / 无降级
- i18n 三条新文案经真实 `registerMessages` + `setLocale` 插值验证，中英文均正确

---

## 0.17.9 — Edge-TTS 协议 v2 支持（恢复免费配音）

**主题**：修复 2026-10-14 发现的「协议代际过期」导致的全线 403。
v2 签名已实现并真机验证，v1 保留为降级路径。

### Fixed
- **edge-tts 免费配音全线 403**：微软于 2026-10-13 把 `Sec-MS-GEC-Version` 前缀
  从 `1-` 升到 `2-`，token 格式同步改为 `CS64-SHA256` 双段。本插件实现 v2 签名后
  真机合成恢复（34848 字节 MP3，帧同步正确）。**注意现象与微软宕机完全一样**
  （WS 403 + body 为空）但处置相反，详见 `doc/20-tts.md` FAQ。

### Added
- **`src/core/ttsGecV2.ts`**：v2 签名实现（约 200 行，零外部依赖、零 obsidian 依赖）。
  **隔离成单文件是有意的** —— v2 的魔数取自 LGPLv3 上游，属派生作品；
  隔离后「摘掉 edge 路线」的成本 = 删 5 个文件 + 改 `tts.ts` 4 处 + 清 `package.json` 3 处
  （**已实测演练**，非推演，见 `doc/40` §5.5）。
  文件头标 `@license LGPL-3.0-or-later` 与上游归属。
- **握手失败自动降级**：`synthChunk` 先走当前代次，403 时切 v1 重试一次。
  将来微软出 `3-` 前缀时不必发版（前提是 token 算法未变）。
- **`generateSecMsGecToken()`** 运行时唯一入口 + `gecProtocolVersion` 开关。
  要整体退回 v1：把该常量改回 `'1'`。
- **`test/gec-v2-parity.test.mjs`**（10 例）：与 Python `edge-tts==7.3.1`
  **逐位对拍**（8 个时间戳 × token 全等+ SHA/CS64 分段全等）+ 常量表逐个比对 +
  窗口恒定性 + 形态断言。
- **`test/extract-gec-v2-consts.py`**：从上游 `drm.py` 直接导出全部魔数，
  杜绝「手工抄常数漏一位」这类**静默失败**。
- **`test/gen-gec-v2-golden.py`** + `npm run test:gec-v2-golden`：重新生成对拍基准。
  `.tmp` 文件不入库时测试自动退回**内置基准**，CI / 干净克隆零依赖可跑。
- **`debugV2()`**：分段导出 salt 派生 / 压缩 / CS64 三段中间量。
  三段出错的表现都是 403，只能靠它区分。

### Changed
- `buildWssUrl()` 增加 `version` 参数；`generateSecMsGec` 标注 `@deprecated`
  （保留原因：v2 若因授权被摘除，这是唯一回退路径）。
- `test/edge-tts-live.mjs` 的诊断输出改打**实际使用的代次与 v2 token**。
  原先打的是 v1 样例（单段 32 hex），看着「有输出」但与真实请求无关，
  排查时会指向错误方向。
- `README` / `README-cn` 的 Third-party Notices 补上 v2 的派生作品声明。

### 已知边界
- 上游 `edge_tts` 的 `_sha256_rounds` 把 64 轮手工展开成「8 轮一组」，
  本实现用标准 64 轮循环 —— 语义等价、代码形态不同。
- v2 的 CS64 与魔改 SHA-256 都**不能靠协议描述自行实现**（无推导路径），
  必须以 `edge-tts` 为参考，故认定为 LGPLv3 派生作品。

### Added
- **`scripts/check-gec-consistency.mjs` + `npm run check:gec`**：v2 摘除一致性护栏，
  挂在 `npm test` 最前面。按 `ttsGecV2.ts` 是否存在走两条分支：v2 在则校验引用完整性
  （import / 分派 / 默认代次 / 降级函数是否被调用），v2 走了则校验无残余引用
  （`package.json` 悬空引用 / 死代码 / 僵尸参数）。
  **问题分两级**——`⛔ BLOCKER`（会让测试崩或功能静默失效）与
  `⚠ WARN`（死代码、悬空手动脚本），初版全标成「硬阻塞」是**自造误报**
  （Python 脚本不在 `npm test` 链里，删掉并不会崩）。
  护栏自身已用构造的摘除态目录实测：能报出2 BLOCKER + 5 WARN。
- **`test/gec-integration.test.cjs`**（13 例）：补齐**集成层**覆盖盲区。
  原先 10 个用例全打在纯算法上，而摘除演练证明真正易错的是 `tts.ts` 的胶水逻辑 ——
  v1/v2 分派写反、`buildWssUrl` 前缀没跟着代次变、降级函数只有定义没调用，
  这些**全都不报错**（tsc 过、算法单测过、真机才403）。
  现覆盖：分派形态、前缀拼接、URL 组装、clock skew 跨/不跨窗口语义、v1 回退入口。

### Fixed
- **`ttsGecV2.ts` 两处冗余乘数 `* 1`**（CS64 链 A/B）：值上 `x * 1 === x`，
  token 完全正确，但**形状与上游不符** —— 上游 `drm.py` 里此处根本没有额外乘数。
  看似无害，实则掩盖了「是否抄漏常数」这个本该一眼可见的问题。已改为与上游同形。
- **四处自相矛盾的注释**（`expandSchedule` / `smallSigma0` / `v2Digest` /
  `deriveInitialState`）：注释描述的是**被否决的错误写法**，或与自己下面的代码打架。
  这类注释不改结果，但会主动把后续改动引向错误方向—— 已在 `doc/40` §6.8 记录。

### 验证
- tsc 0 错 · 全量 **1329 pass / 0 fail**（新增 13）· 生产构建 OK
- 与 Python 逐位对拍 **10/10**；review 中另现场对拍 2 个时戳，与上游 7.3.1 **逐位全等**
  （`63DEFE91AD9FDACC-…` / `2CFA218E361E5B1A-…`）
- **真机合成成功**：34848 字节 MP3（`npm run test:tts-live`），GEC 代次 2
- 摘除流程实测演练：真删文件 + 改调用点，tsc 仍 0 错，但发现
  **`package.json` 的 `test` 链残留 2 处悬空引用会让 `npm test` 直接
  `MODULE_NOT_FOUND` 崩掉**、另有 4 处死代码/僵尸参数静默通过。
  完整残留清单见 `doc/40` §5.5 —— 现已固化为自动护栏（§5.6），不必再靠人记
- 顺手修掉 `tts.ts` 两处 JSDoc 残留（重复注释块、常量删除后悬空的注释）
- 订正 `doc/40` 交叉引用错误：踩坑节号从「第九节」改回「第六节」

### 排查证据存档（2026-10-14，v2 实施前的调研阶段）

> 以下是**定位协议升代时的原始证据**，保留备查。结论已落地到 0.17.9。

**当时的主题**：排查「edge-tts 全线 403」时发现**协议已升代**，插件 v1 实现已被
微软服务端拒绝。该阶段只做调研留档 + 合规声明，尚未修改 TTS 代码。

**排查结论（重要）**
- **不是宕机**：同机同时段对照实验 —— 官方 `edge-tts==7.2.8`（v1）403，
  `==7.3.1`（v2）成功产出 23328 字节 mp3。一次排除宕机 / 本地网络 / 环境问题。
- **不是改个版本号能修的**：7 组握手矩阵实验（含「真 v2 token 配 `1-` 前缀」
  「真 CS64 配假 SHA」等）**全 403** → 前缀代号与 token 格式是两个独立的必检项，
  且 CS64 与 SHA 两半都校验。
- **与 2026-10-12 宕机现象完全一样**（WS 403 + body 为空）但**处置相反**：
  前者等恢复，后者改代码。已在 `doc/20-tts.md` FAQ 加区分方法。
- 移植难度约 180 行 TS，但 v2 的 SHA-256 **不是标准算法**（前 7 字节当 salt 派生
  自定义 IV + 轮常数魔改），Node 内置 `crypto.createHash` 用不了。

**该阶段新增的文档（已全部落地）**
- **`doc/40-edge-tts-v2-and-compliance.md`**（新建）：v1→v2 差异、握手矩阵、
  证据链、移植要点，以及 MIT 插件的 LGPL 污染判定（三条路径）。
- **`README.md` / `README-cn.md`** 新增「第三方声明 / Third-party Notices」小节：
  edge-tts 为 LGPLv3 + 本插件 v1 为独立实现的归属声明 +
  **未公开端点无 SLA、商业使用需自行评估**的免责说明。
- **`doc/20-tts.md` §2.8「合规与风险」**（新建 5 个子节）、§2.4 补 GEC 前缀
  （`1-`→`2-`）与失效状态、FAQ 补「2026-10-14 起全线 403 是宕机吗」与
  「edge-tts 有法律风险吗」两条、`doc/README.md` 索引 + 快导行。

**澄清（避免误判，此结论依然成立）**
- 现有 v1 实现**是干净的**：`generateSecMsGec` 只用 Node 内置 `crypto`，ticks 算式由
  微软协议规定，未取用上游代码；`src/core/tts.ts` 注释里的「照 rany2/edge-tts 实现」
  是**归属披露**而非抄袭。污染风险只在移植 v2 token 算法时出现——
  **该风险在 0.17.9 中已实际发生**，处置方式见 §5.5 摘除演练。

## 0.17.8 — review 收口：失败清单去重 + 中文本地路径

**主题**：上一版发布后做一轮「报过但没修」的 review，抓出两个真缺陷——一个是我自己在 0.17.7
引入的、一个是一直潜伏在离线调研入口的。

### Fixed
- **失败条目翻倍**（0.17.7 引入）：加了 `onAudioFailed` 逐镜推送后，整批 `renderShotAudio()`
  resolve 的 `this.assetFailures.push(...audio.failures)` 把同一条失败**再推一遍** → 失败清单
  翻倍、警告条数虚高。抽出 `mergeFailures(newOnes)` 按 `(stage, index)` 去重，把 6 个写入点收拢成
  「1 处定义 + 5 处调用 + 1 处重置」；判据带 stage（S3 出图失败 ≠ S3 配音失败），用 `Set` 记键。
- **中文本地路径识别不到，深度调研被静默跳过**：`LOCAL_RE` 用 `\w`（只含 `[A-Za-z0-9_]`，不含
  任何非 ASCII），`I:\...\墨影AI工场V1.0.0\...` 整段匹配失败 → `detectSources` 返回空 → 面板弹
  「没有识别到可抓取链接」。改为 `[\p{L}\p{N}_./\\-]` + `u` flag。原本只有「函数存在」的结构断言、
  **零行为断言**，所以一直没人发现。

### Changed
- 补 5 条本地路径行为用例（中文目录 / Unix 与 `~/` 形态 / URL 不误判 / 本地压过远程 /
  owner 不一致时不去重）+ 2 条失败去重契约用例（去重键必须含 stage、整批回填不允许裸 `push(...)`）。
- 订正 `doc/34` 一处过期记录：调研 clone 已不在 `Temp/`（0.17.6 迁到 vault 共享缓存），
  但新缓存同样无回收策略。

中英各 **2060 键**（零增删）；测试 **1295→1302**（+7 行为与契约用例）全绿；
lint 与 HEAD 持平 **90**（中途因正则多余转义 `\/` 涨到 91，已修回）。

## 0.17.7 — 解说片：配音文件列表逐镜增量刷新

**主题**：出图早就逐镜点亮缩略图了，配音却还在等整批跑完才把列表一次填进来——用户看到
「配音进度 9/9」，下面「配音文件」却一直是 `—`。同一份 UI 里两种刷新语义。

### Changed
- **配音逐镜回调**：`RenderAudioOpts` 新增 `onAudioDone(asset)` / `onAudioFailed(failure)`，
  与出图的 `onFrameDone` 同构。worker 三个终态各触发一次（成功推 asset；缺文本早退与 catch
  各推一条 failure）；`bump()` 仍只在「缺文本早退」+「finally」两处，进度分母口径不变。
- **配音列表抽出独立刷新**：`renderAssetLists()` 里的配音块抽成 `renderAudioList()`，新增
  `absorbAudioAsset()`（按 index upsert + 排序）/ `absorbAudioFailure()`（去重判据带
  `stage === 'audio'`，`assetFailures` 里混着出图阶段的失败）；「自动跑完」与「分步」两处
  `renderShotAudio` 调用点都接上回调。只刷配音这一块——字幕 / BGM / 清单在配音阶段不变，
  逐镜重建它们只会闪一下。
- **删掉重复的配音进度行**：`renderAssetProgress()` 每调用一次就往 `assetAudioEl` **追加**
  一条进度行（带 `data-asset-progress`），而配音进度本来就有 `audioProgressEl` 一条，且两者
  都只在配音阶段可见。原先每帧只调一次没人注意，改成逐镜回调后会堆出 9 条。直接删掉（连同
  它唯一的调用点、以及只被它用的 `currentStage()`），配音进度只留 `audioProgressEl` 一条。

中英各 **2060 键**（零增删）；测试 **1286→1295**（+9 逐镜增量回归钉）全绿；
lint **92→90**（删掉的方法自带 2 条 `obsidianmd/no-static-styles-assignment`，无新增）。

## 0.17.6 — 解说片：静帧缩略并入「4 · 生成素材」，收口 0.17.3 的半迁移

**主题**：一次做完一半的迁移收口。0.17.3 把「出图进度」从分镜组搬到素材组（理由是「点
『生成素材』看到分镜组亮起，会误以为分镜被重跑」），但**产物**——静帧缩略图——留在了分镜组。
结果用户点一次按钮要开两个分组才能对上产物：进度条亮在 4、缩略图躺在 3。本版把缩略区搬到
出图进度正下方，进度与产物同组同序。

### Changed
- **静帧缩略区搬到素材组**：`thumbsEl` 由 `g3Body` 移到 `g4Body`，紧接 `framesProgressEl`
  下方。素材组顺序变为 生成素材按钮 → 出图进度 → **静帧缩略** → 配音进度 → 配音文件 → BGM
  → 字幕 → 清单。分镜组只剩分镜自己的产物：出镜进度行 + 镜头表。
- **素材组标题补上「图」**：原「4 · 生成素材（配音 / BGM / 字幕 / 清单）」没提图，
  但组内按钮文案是「生成素材（图/音/字幕）」——分组标题、按钮文案、产物归属三处口径不一致，
  统一为「（图 / 配音 / BGM / 字幕 / 清单）」。
- **交叉引用保留**：每张缩略图仍带 `S1/S2…` 标题，点击走 `openShotCard()` 弹大图 +
  实际提示词 + 单镜重出；代价只是多一次点击换来看镜头表与静帧的相邻对照。
- **过期编号注释清理**：`styles.css` 里四处「分组 0/1/2/3」是旧的 0-indexed 说法，现按内容命名。

中英各 **2060 键**（本轮零增删，仅改标题键的值）；测试 **1286/1286** 全绿；
lint 与 HEAD 基线持平（15 errors / 1 warning，全为既有）。

## 0.17.5 — ffmpeg 配置重做 + TTS 三引擎折叠分组 + Audio8 TN 协议名转中文全称

**主题**：三件事一起。① ffmpeg 面板「每次重载都要重新应用路径」其实是**错觉**——路径一直
保存在 `settings.ffmpegPath`，但探测结果是**内存态缓存**，重载后必然为空，状态条因此恒显示
「未找到」。本版改为**打开面板即自动探测**，并把「扫描全部安装 → 挑一个 → 应用」作为主流程，
两个输入框直接显示 ffmpeg 包的**完整安装路径**。② 「检测当前配置」与「扫描全部安装」职责重叠，
删掉前者。③ TTS 设置页 20 多个输入框一屏糊满，拆成 edge-tts / 豆包 / Audio8 三个**默认折叠**
的分组。另含 Audio8 TN 的协议名处理。

### Changed
- **ffmpeg 探测结果落成完整路径**：新增 `resolveCommandFullpath()`，PATH 上的裸命令名（`ffmpeg`）
  按 PATH 逐目录 `existsSync` 解析成绝对路径（win32 补 `.exe`），解析不到原样返回。UI 与日志
  直接看到真实安装位置，不再只显示一个裸名。
- **打开 ffmpeg 面板即自动探测**（`refreshFfmpegStatus`）：缓存命中直接回填、不起子进程；
  未命中则按已保存路径探测一次，路径有变化就 `save()` 并 Notice。是否回填由 `writeBack` 控制——
  自动探测回填，**手动改路径后的校验不回填**（否则用户没打完字就被另一条路径覆盖）。
- **删掉「检测当前配置」按钮**：职责并入「打开即探测」+「手改路径后 900ms 防抖自动校验」
  （`verifyFfmpegDebounced`；`display()` 时 `clearTimeout`，防旧节点起子进程）。
- **应用扫描结果不再整页重建**：`applyFfmpegPick` 改为原地 `syncFfmpegInputs()` 刷新两个输入框，
  保住滚动位置与其他分区展开状态；`textSetting()` 返回 `TextComponent | null` 以支持原地 `setValue`。
- **TTS 三引擎分组折叠**：`tts()` 只留跨引擎通用的「默认引擎」与「配音子目录」，edge-tts /
  豆包 Seed-Audio / Audio8 各自收进 `section(..., open: false)` 折叠组；Audio8 组保留
  `canUseFfmpeg` 门禁（依赖本机 Node 运行时，移动端整个分组不显示）。
- **Audio8 TN 协议名转中文全称**：`http` → 超文本传输协议、`https` → 超文本传输安全协议、
  `ftp` → 文件传输协议、`www` → 万维网；正文里的「http协议」联动剥掉冗余的「协议」二字；
  `urlToSpoken` 剥掉 URL 首部的 `www.`。套件 `tts.py` 同口径镜像。

中英各 **2060 键**（+7 新分组键、-3 废弃键，订正此前文档记的 2058）；测试 **1283→1286**
（+3 为协议名 TN 用例），全绿。

## 0.17.4 — Audio8 TN 修正：孤立 scheme 不再静默（保留协议名）

**主题**：0.17.3 的「孤立 scheme 剥离」把裸 `http://` **整段删除**，导致用户写
`http://   这是http协议` 时开头的「http」完全不读（静默）。本版改为**保留协议名、只剥 `://` 分隔符**
（`http://   这是http协议` → `http 这是http协议`），既消除 `://` 被逐符号念的高频噪声，又让协议名正常读出。

### Fixed
- `STANDALONE_SCHEME_RE` 替换逻辑由 `''` 改为保留协议名（`http`/`https`/`ftp`）+ 剥 `://`
  （`m.replace(/:\/\//g, '')`）。真实整 URL 仍由 `URL_RE` 口播化，不受影响。
- 配套：套件 `tts.py` 同步（`_STANDALONE_SCHEME.sub(lambda m: m.group(0).replace("://", "").rstrip("."))`），双端同口径。
  CLI 实跑 `http://   这是http协议` → `http 这是http协议`，模型读出「http」。
- 单测期望同步（孤立 scheme 用例由「整段删除」改为「保留协议名」）。

中英各 2058 键不变；测试总数不变（1283）。

## 0.17.3 — Audio8 TN 强化：孤立 scheme 剥离 + 超长拉丁串切块

**主题**：0.17.2 修好了 URL 口播化与预算公式，但用户实测仍有两类残留让 Audio8 读英文/混合文本
「非常奇怪」：① 裸 `http://` 后面没跟 URL 内容（只接空格/标点）原样进模型、被逐符号念；
② `datawhalechina` 这类 14 字母连写品牌词被模型逐字母念且中途 stall。套件 CLI `tts.py` 先补了
TN 这两步并实跑验证（分片输入 `hiss_frac` 0.491→0.106，NOISY→CLEAN），本版把**同口径**的两步
补进插件 `normalizeAudio8Text`，让解释器脚本里的长英文词同样不再 stall。

### Fixed

- **孤立 scheme 前缀剥离**（`STANDALONE_SCHEME_RE`，0.17.3 新增）：裸 `http://` / `ftp://` 后面
  紧跟空格或标点（非真实 URL）时，原样进模型会被逐符号念（`hiss_frac` 飙到 0.491）。
  在 URL 口播化**之前**剥离；lookahead 保证真实整 URL（如 `https://github.com/x`）仍由 `URL_RE`
  正常口播化、不被误剥。
- **超长连写拉丁串切块**（`LONG_LATIN_RE = /[A-Za-z]{9,}/` + `chunkLongLatin`，0.17.3 新增）：
  模型对 ≥9 字母连写串（如 `datawhalechina`=14）逐字母念且中途 stall。按 4 字切块插空格
  （`datawhalechina` → `data whal echi na`），把 stall 挪到词边界。阈值取 **≥9**（而非 8）以不误伤
  `cookbook`(8) 这类常见词——`datawhale`(9) 因 9≥9 仍切，验证 55/55 单测通过。
- 配套：套件 `tts.py` 的 CLI 同步镜像这两步（`_STANDALONE_SCHEME` / `_chunk_long_latin`，maxlen=8），
  双端同口径；CLI 实跑同 seed 对照，分片输入 hiss 0.491→0.106、整句 BORDERLINE→CLEAN。

测试 1281→1283（TN 两步各加 1 个用例）。中英各 2058 键不变。

## 0.17.2 — Audio8 合成：预算公式重写 + TN 预处理 + 字幕竖线泄漏修复

**主题**：用户报障「Audio8 读 URL、数字发音非常奇怪，会中断」。两轮合成探针 + 一段成品
解说片逐段扫描后，定位到**三个叠加问题**：预算公式对非中文严重低估（软件 bug）、URL 与
连字符词让模型坍塌成静音（模型分布外，软件侧用 TN 缓解）、解说词分隔符 `|` 漏进 TTS
（软件 bug，且在成片字幕里可见）。

### Fixed

- **token 预算公式按字符重写**（`computeAudio8Budget`）：旧公式「CJK×9 + 英文词×5 + 16」
  对非中文低估 2–4 倍——46 字符的 URL 只拿到 46 token，实测需要 93；口播形式 97 → 实测 202。
  根因是模型对非中文**逐字母/逐符号念**，词内字符数不封顶，而旧公式完全不给符号计价
  （`://` `/` `-` `.` 白念但照样烧 token）。新公式 **CJK×9 + 数字×9 + 拉丁字母×4 +
  ASCII 符号×4 + 16**：数字单独按 9 计价，因为模型把「11」念成「十一」（18 帧，与两个
  汉字同价），按词算只有 5 token，手机号 13800138000 会被砍成半句。
- **新增 TN 预处理层**（`normalizeAudio8Text`，分片前应用）：Audio8-0.1B 训练数据
  （套件 `docs/0.1B/data.json` 全部 70 条样本）**0 条含 URL**，模型遇到分布外文本不报错，
  而是坍塌成静音码且不吐 EOS，把剩余预算全烧成死寂——实测「这套教程叫jev-cookbook」
  静音率 0.498、1.9s 之后 1.88s 死寂。四步规范化：URL/邮箱 → 口播形式
  （`github dot com slash jev dash cookbook`）、字母间连字符 → 空格、字幕分隔符 `|` → 逗号、
  折叠重复标点。同一 seed 前后对照：URL 原样静音率 0.510 / 9 处 gap → 口播形式 0.255 / 1 处；
  `jev-cookbook` 0.498 死寂 → `jev cookbook` 0.158 无 gap。
- **解说词分隔符 `|` 漏进 TTS**（`mergeShortShots` 用 `|` 连接合并镜头，`renderShotAudio`
  把合并后的 text 原样送合成）：模型训练数据里没有 `|`，每处产生 0.3–0.66s 停顿——纯中文句
  「这就像让厨师直接给你一碗汤|你却只想尝咸淡」仅因竖线就在 3.24s 处卡停 0.52s，成品片
  4/4 精确复现。新增 `flattenCueText()` 在送 TTS 前压平；字幕渲染同步修复——
  **成片 ASS 的 Dialogue 行此前会显示字面竖线**（该片 20 行中招）。
- 预算给足后 URL 不再硬切：`frames == budget` 且 `tail_rms > 0.5` 的截断签名全部消失，
  探针所有用例自然 EOS（手机号 56/115、英文 68/108、URL 口播 228/292）。

测试 1270→1281；中英各 **2058 键**对称（0.17.1 文档记的 2057 比实测少一个，本版订正）。

## 0.17.1 — Audio8 服务：真机联调修复

**主题**：0.17.0 的服务管理器是照着套件脚本的**源码**写的，但从没在本机跑通过一次。
本版做真机联调，把「看起来对」和「真的能跑」之间的差距补上。

### Fixed

- **Start 死锁：陈旧 `.service.pid` 会让服务永远起不来**（真机联调发现，用户无法自愈）。
  `start_server.ps1` 的幂等检查是 `if (Get-Process -Id $existingId) { exit 0 }`，
  **只验 PID 存活性、从不看端口**。异常关机留下陈旧 pidFile 后，Windows 重启会把那个数字
  重新分配给任意无关进程 → 点 Start 时脚本 `exit 0` 说「已在运行」但服务根本没起 →
  插件二次 health 复检失败 → **用户再点一百次都一样**，因为 pidFile 从没被清过。
  现在 Start 前凡是「health 不通但 pidFile 还在」就先清掉它（`clearStaleAudio8PidFile`，
  删的是**下钻后脚本所在那层**，删错层等于没删）。
- **Stop 谎报「已停止」**：服务在运行但**没有** pidFile 时（用户手动 `run_server.ps1` /
  `python -m uvicorn` 拉起来的很常见），`stop_server.ps1` 第 5-8 行遇到无 pidFile 直接
  `exit 0` 说已停止。原来插件只报一句没信息量的「停止失败」，现在区分出
  `audio8Service.unmanaged`，明确告诉用户「不是套件脚本启动的，插件无法自动停止，请手动结束进程」。
- **监听地址与连接地址不一致**：脚本缺省只监听 `127.0.0.1`，而插件连的是用户填的 baseUrl。
  用户把 baseUrl 填成 `http://localhost:8024` 时，Windows 上 `localhost` 优先解析到 `::1`，
  脚本却只听 IPv4 → 「脚本说启动成功、插件永远连不上」。
  新增 `parseAudio8Host()` 并把 host 通过 `HOST` env 传给脚本，两边强制对齐
  （IPv6 方括号会被剥掉，怪字符走白名单回退）。
- **配音页切引擎后 edge-only 控件不隐藏**（CSS 层叠 bug，影响所有三引擎切换）。
  `.agg-hidden { display: none }` 定义在 `styles.css:1000`，而 `.agg-inline`
  （`display:flex`，1309 行）和 `.agg-check`（`display:inline-flex`，1323 行）定义在它**之后**；
  同特异性下后者胜出 → `.agg-hidden` 对同时挂这些布局类的元素**完全失效**。
  配音页三引擎的行 div（edgeRow / doubaoRow / audio8Row）与两个复选框
  （标点停顿 / 生成字幕）恰好全踩中：切到 Audio8 后云希音色、自然、正常、
  单段字符上限全部残留，界面混乱。改为 `display: none !important`
  （「强制隐藏」语义下使用 `!important` 是正当的），交叉验证：截图里被正确隐藏的
  参考音频区（`.agg-ref-area` 定义在第 491 行、在 `.agg-hidden` 之前）与此诊断完全吻合。
- **「规范化 [男]/[女]」按钮只在豆包模式显示**。该按钮产出的是 Seed-Audio 的
  text_prompt（`凌云霄（男性，xx音色）说道："…"`），只有豆包引擎消费这种自然语言提示词；
  edge（单音色 SSML）与 audio8（零样本克隆）都会把这段元信息**一字不差念出来**。
  切到非豆包引擎时按钮自动隐藏。
  **流水线不受影响**：短剧 Pro 逐镜配音合成前走 `toSpeechText()`（subtitle.ts），
  实测 `[男]凌云霄："这路，太慢。"` → `"这路，太慢。"`，性别标记与署名都被剥掉，
  audio8 拿到的已是纯台词。
- **音色管理页试听与显示文本不一致**（用户报障）：列表显示的是音色的
  `reference_text`（注册参考音频的转写），试听按钮却朗读固定短句
  `我是中国人，我爱我的祖国。`——「听到的」和「看到的」对不上，被当成 bug。
  修：新增 `pickAudio8PreviewText()`，试听改为朗读显示那段 reference_text 的
  **第一句**（句末标点断句，超过分片上限 120 字符时截断保证单分片合成最快；
  无标点截断到上限；空文本回退固定短句），按钮 tooltip 同步显示实际会念的句子。
- **配音页主面板「试听」无视输入框内容**（同一报障的第二处）：`run(preview=true)`
  原来永远读写死的 `明月出天山，苍茫云海间。`，输入框里打什么都不念 ——
  与音色管理页的试听内容还不一样，用户以为是两个 bug。
  修：抽通用 `pickPreviewSentence(text, cap, fallback)`（`pickAudio8PreviewText`
  改为其特例），主面板试听改为**朗读输入框文本的第一句**（截 120 字符保单分片秒回，
  空框才回退示例句），并给按钮加了行为说明的 tooltip（`tts.previewHint`）。
  两个试听入口职责不同但行为一致：**念的都是按钮旁边显示的文本**。
- **解说片模块无法切换到 Audio8 音色**（用户报障）：音色下拉写死 edge 的
  `TTS_VOICES` 九个音色，`ttsProvider` 切到 audio8 后用户既选不到 audio8 音色，
  面板还会把 edge 音色 id（`zh-CN-YunyangNeural`）直接传给 audio8 服务端（必然失败）。
  修：`rebuildVoiceOptions()` 按当前引擎分支——edge 列内置音色（顺带修掉
  「恢复项目只回填字段不同步下拉选中项」的老 bug）；audio8 异步拉服务端已注册音色
  （带竞态序号防过期响应，失败保留 default 占位不阻断）。新增纯函数
  `isEdgeVoiceId()` / `normalizeAudio8Voice()`（audio8Tts.ts）：合成与 manifest
  统一走 `synthVoice()`，老 profile 里存的 edge 音色自动归位到 `settings.audio8Voice`；
  profile 存档仍保留原始选择。解说片面板每次进入都会重建，配置页切引擎后回来即生效。

### Changed

- `.service.pid` 文件名收成常量 `AUDIO8_PID_FILE`（原来散在代码和注释里）。

### 真机联调记录

| 观察 | 结论 |
| --- | --- |
| uvicorn 起**两个** python 进程 | pidFile 记的是父进程（3.4 MB），真正加载 ONNX 的是子进程（**1.8 GB**） |
| 杀父进程（3.4 MB 那个） | 子进程跟着退、端口释放 → **stop 脚本不带 `/T` 也能停干净**，先前担心不成立 |
| `service.error.log` 里有 uvicorn INFO | 脚本把 stdout 合并进了 stderr，排查时别只盯 `service.log` |

## 0.17.0 — Audio8-TTS 本地配音引擎（edge-tts 托底）

> **未单独发布**：本版内容从未对外分发，也未单独提交，已与 v0.17.1 的修复合并在
> 同一个 release 提交（`18d5fa0`）里，因此 git 上**只有 `0.17.1` 这一个 tag**。
> `versions.json` 保留 `0.17.0` 仅为兼容映射。

**主题**：微软后端长期故障时的第三条免费路径 —— 本机跑一个 HTTP 服务，插件直接调用，不再依赖微软。

### Added

- **第三条配音引擎 `audio8`**：`src/core/audio8Tts.ts`（新增）对接本机 Audio8-TTS 服务（HTTP `127.0.0.1:8024`），
  免费、离线、不消耗任何厂商额度。服务需用户在本地另起一个进程，见 `doc/39-audio8-integration.md`。
- **音色管理弹窗** `src/ui/Audio8VoiceModal.ts`（新增）：列出服务端音色、试听、注册新音色。
  Audio8 是零样本克隆引擎，音色要「注册」而不是「选」—— 上传一段参考音频 + 逐字准确的转写文本即可。
- **TTS 引擎分派层** `src/core/ttsRouter.ts`（新增）：`synthesizeTts()` 按 `ttsProvider` 统一路由，
  `ttsContainer()` 给出落盘扩展名（edge → mp3，audio8 → wav）。
  逐镜配音的四处调用点（流水线阶段 5 / 数字人阶段 2 / 素材生成 / 单镜重出）全部改为走这一层。
- **流水线引擎回落**：短剧流水线未勾选豆包时原来硬编码 `edge`，现在回落设置里的默认引擎，
  选 `audio8` 也能生效；提示文案随之三态化。
- **服务管理器** `src/core/audio8Service.ts`（新增）：Config 页新增「Audio8 服务管理」子块，
  用户填一次 install-dir 即可，点 Start/Stop 按钮控制本机 Audio8 服务（子进程 spawn `start_server.ps1` /
  `stop_server.ps1`，POSIX 走 `.sh`）；状态来自 `/api/health` + `.service.pid`，5 秒缓存。
  Start 走 90s 超时 + 二次 health 探测（脚本 exit 0 但端口没起来也报失败）；Stop 走 15s 超时。
  **install-dir 会向下探测 ≤2 层**：套件把脚本放在 `…/Audio8_TTS/onnx_runtime_0_1b_int8/` 里，
  填套件根目录也能自动定位到真正那一层，并在状态 chip 上标「已定位到 xxx」。
- **schema v7 迁移**：`audio8InstallDir` 兜底为空串。前一版 v6 迁移（`audio8BaseUrl` / `audio8Voice` /
  `audio8ChunkChars` 三字段兜底，`audio8ChunkChars` 夹到 `[10, 1000]`，`ttsProvider` 非法值归位 `edge`）保留。
  顺手修掉 `DEFAULT_SETTINGS.schemaVersion`（4）与 `SCHEMA_VERSION`（5）长期不同步的问题。

### Changed

- **音频扩展名跟随引擎**：`shotAudioName(index, ext)` 增加可选参数。逐镜配音的落盘与
  Remotion 按名反查三处必须一致，否则会出现「合成成功但成片无声」的静默失败。
- **逐镜配音并发**：`audio8` 服务端一次只处理一条请求，流水线/素材阶段并发强制降为 1。
- `PipelinePanel` 引擎提示按实际生效的引擎给出对应说明。

### Fixed

- `Audio8VoiceModal.submit()`：注册成功会重建列表，`finally` 里对已销毁的按钮调 `setText`
  （打到 detached 节点，不抛异常但语义脏）。加 `domReplaced` 标志跳过。
- `audio8Tts.ts`：删掉 `req.destroyed = true`（`HttpRequestLike` 接口无此字段，TS2551；
  真正的竞态已由 `fail()` 的 `settled` 守卫挡住），并移除未使用的 `tError` import。

### Docs

- **新增** `doc/39-audio8-integration.md`：Audio8-TTS 服务端部署、协议对接、音色注册全流程
- `doc/20-tts.md` 升级为三引擎：总览表、数据流图、流水线分派、FAQ
- `doc/36-tts-alternatives.md` 新增 §3.6 Audio8-TTS（本次落地方案）
- `doc/29-feature-matrix.md` 配音行与 i18n 键数同步

### 测试

- 新增 `test/audio8-client.test.mjs` 35 例（token 预算公式、WAV 头解析、按句分片、
  multipart 构造、错误映射、i18n 缺键回退）
- 新增 `test/audio8-service.test.mjs` 21 例（端口解析含 IPv6、install-dir 检查含 Windows/POSIX
  分支、venv/脚本存在性校验、向下探测 1/2 层与跳过隐藏项、状态缓存按配置分键、
  超时常量合理性）
- 新增 `migrateSettings` v6 / v7 迁移用例；全量 **1253 例全绿**，`tsc --noEmit` 通过

**注意**：Audio8-TTS 服务需用户在本地自行启动，沙箱 / CI 环境内后台进程会被回收。
若启用服务管理器，Config 页里 Start/Stop 按钮会在后台 spawn 对应平台的启动脚本。

---

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
- **新增** `doc/37-ad-slots.md`：广告位系统总览（TopAdSlot 顶部常驻 + AdBanner 底部悬浮）+ 5 点修改广告位清单
- `src/ad/top-ad-slot.ts` 顶部注释升级为「📌 修改广告位清单」：从原来的"两处必改"扩展到 linkUrl / ctaUrl / imageUrl / 文案 / mode 五点，config 字段配 🔗 位置标注，未来换广告不用猜"改哪儿"

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
