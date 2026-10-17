# Free AI Studio — Obsidian Plugin

> **Language / 语言**：[English](./README.md) · [中文](./README-cn.md)

Generate **text / images / videos / voiceover / novel covers / talking-head avatars** right inside Obsidian, and compose **short-drama films** (with a seven-stage Red Fruit Pro pipeline). Unified multi-vendor API management, all artifacts written directly into your Vault with Markdown wiki-link support and resume-after-restart.

- Current version: **v0.16.3** (2026-10-12)
- Requires Obsidian **≥ 1.7.2**
- Desktop only (`isDesktopOnly: true`; requires Node/Electron and ffmpeg)
- Release notes: [`CHANGELOG.md`](./CHANGELOG.md)

---

## Core Capabilities

| Panel | Capabilities |
|------|------|
| **Text** | 4 modes (free-form writing / short-drama outline / storyboard script / prompt optimization) + **6 script-quality modes** (health check / rewrite / 70-episode split / lock-code design / 8-step storyboard / first-last-frame inheritance); JSON output auto-rendered as structured blocks; saved as notes with frontmatter |
| **Image** | Text-to-image / image-to-image; reference images can be picked from **the Vault**, **uploaded locally**, or pasted as a URL; auto-compressed and normalized; concurrency guard against 429s |
| **Video** | Text-to-video / image-to-video; four-branch protocol; tasks persisted — **auto-resume on restart** |
| **Voiceover** | **Doubao Seed-Audio full-episode dubbing** (multi-role dialogue + ambient + BGM + word-level subtitles) + **Audio8 local engine** (self-hosted, free) + **MTV** music-video montage. *The edge-tts engine (9 voices / 6 emotions / 3 speeds / punctuation pauses) is **VIP*** |
| **Short Drama** | Outline → synopsis → storyboard JSON → per-shot images → per-shot videos → **voiceover → final render** → Markdown note, one-click orchestration |
| **Red Fruit Pro** | Script → storyboard → character art → first/last frames → video → voiceover → final render, **seven-stage orchestration**; multi-version selection + rewrite + pinpoint correction; **per-shot API-key rotation**; resume-from-breakpoint + per-stage re-run |
| **Cover** | Title + author + synopsis (**imports TXT/DOCX**) → text model produces a prompt → image → **crop to target aspect** → **lay out title/author** → writes three files into the Vault; live layout preview |
| **Talking Head** | Script → **segmentation** → voiceover → **portrait-to-video** (auto-fallback to Ken Burns) → per-segment clips → subtitles → final render, **four-stage orchestration**; 5 performance styles; per-segment re-run |
| **Explainer** | Research → narration → storyboard → frame images → voiceover → **Remotion render**; **BGM bed** (C-major + arpeggio) + static QC report; three-stage system prompts are user-overridable |
| **Log** | **Runtime trace panel**: a full timeline of HTTP requests, ffmpeg subprocesses, and pipeline-stage transitions; module filter, auto-follow-latest, one-click export to a note |

**Cross-cutting features**

- 8-vendor unified routing + key fallback chain (custom model → single-vendor key → global text key → Agnes primary)
- **Account pool (local free-quota pool)**: domestic doubao-account-pool / overseas dola-pool; URL and model-ID decoupled, with connectivity check
- **Video KEY pool**: up to 3 keys rotated per shot with a process-level submit window, so "one key can't cover 10 shots" stops being a problem
- **ffmpeg post-production**: concat (auto-handling mismatched resolution/frame-rate), subtitle burn-in, audio/video mux; graceful degradation when the binary is missing
- **Canvas layout**: cover cropping and text layout run entirely on Canvas 2D — no PIL / ffmpeg dependency
- **Idempotent stages**: every stage of the three flagship pipelines is an independent idempotent function, each step flushed to disk → **resume-from-breakpoint for free**
- **VIP gate**: `deviceFingerprint` + gate in two layers; VIP users never see ads
- **Top ad slot**: reserved, off by default, one config flag to enable later
- **Resilience**: exponential-backoff retryer + concurrency semaphore + edge-tts clock-drift self-correction + Microsoft service-outage detection
- **Ollama local-model autodiscovery** (on startup + every 5 minutes)
- **Task cleanup**: 30-day auto + manual button
- **Popout window and theme switching support**
- **UI accessibility**: 5-second-confirmation modals for destructive actions, status chips, collapse badges, visible keyboard shortcuts, focus-visible styles

---

## VIP Features & Disclosure

This plugin is free to use for its core generation features. The following **advanced pipelines are VIP-only** (require activation; the tab/command is hidden for non-VIP users and appears after activation):

| VIP-only feature | What it does |
|------------------|--------------|
| **Red Fruit Pro** (`pro`) | Seven-stage short-drama film orchestration |
| **Talking Head** (`anchor`) | Four-stage portrait-to-video avatar pipeline |
| **Cover** (`cover`) | Novel-cover generation with title/author layout |
| **Let Her Dance** (`dancer`) | Wan-Dancer music-driven dance video generation (Aliyun DashScope `wan2.2-dancer`) |
| **B-roll Background Video** (`broll`) | Five-stage idempotent pipeline: material → plan → storyboard → keyframe images → clips, with the three-step character flow (profile / design / anchor) |
| **InkOS Writing Desk** (`inkos`) | Vault-native novel projects: chapters, assets, AI continuation, chapter → storyboard JSON handoff to Red Fruit Pro |
| **Infinite Canvas** (`canvas`) | Node-graph orchestration: text / image / video nodes, dependency links, parallel generation, first-frame inheritance |
| **edge-tts engine** (Voiceover → engine) | Microsoft edge-tts: 9 voices / 6 emotions / 3 speeds / punctuation pauses. Non-VIP users still get **Doubao** (API key) and **Audio8** (self-hosted, free), and the option is disabled (not silently broken) |
| **8 extra UI languages** (Settings → Language) | Japanese, Korean, Russian, Arabic (RTL), Hindi, Spanish, French, German. Incremental translation packs covering the core UI; untranslated strings fall back to English. **Simplified Chinese and English stay free forever** |

All other panels — **Text, Image, Video, Drama, Voiceover, Explainer, MTV, Gallery, Log** — plus unified multi-vendor API management, account pools, and ffmpeg post-production — remain **free forever**. Script-quality tooling (script check / rewrite / 70-episode split / lock-code design / 8-step storyboard / first-last frame inheritance) ships inside the free **Text** panel.

> **Obsidian Developer Policies compliance:** this plugin offers *optional payments*. No feature is removed from the free tier to force payment — the VIP panels are additive enhancements.

---

## Installation

### Download from GitHub Release (recommended)

1. Open the GitHub Release page and download the **`.zip`** for the latest version
2. Extract it to `<YourVault>/.obsidian/plugins/agnes-ai-studio/` so the folder looks like:

```
<YourVault>/.obsidian/plugins/agnes-ai-studio/
├── main.js
├── manifest.json
├── styles.css
└── versions.json
```

3. In Obsidian: Settings → Third-party plugins → enable **Free AI Studio**
4. Click the ✨ icon in the left ribbon to open the workbench

### Prerequisites

- **ffmpeg** (required for voiceover, final render, and Red Fruit Pro's render stage). Not bundled with the plugin — install it separately. Missing binaries cause related features to hide automatically.
  - Windows: `winget install ffmpeg` or [gyan.dev](https://www.gyan.dev/ffmpeg/builds/)
  - macOS: `brew install ffmpeg`
  - Linux: `sudo apt install ffmpeg`

---

## Quick Start

1. Open the workbench → click "Open settings" in the bottom-right
2. Add at least one vendor API key (Agnes primary key is recommended — it anchors every fallback chain)
3. Switch to any tab, enter content, hit generate
4. Artifacts land directly in the Vault at the current folder, notes come with frontmatter for wiki-linking and search

---

## Vendor Support

| Vendor | Text | Image | Video | Base URL |
|------|:---:|:---:|:---:|----------|
| Agnes AI | ✅ | ✅ | ✅ | `api.agnes-ai.cn/v1` |
| DeepSeek | ✅ | ❌ | ❌ | `api.deepseek.com/v1` |
| OpenAI GPT | ✅ | ❌ | ❌ | `api.openai.com/v1` |
| Qwen (Tongyi) | ✅ | ✅ | ✅ | `dashscope.aliyun.com/...` |
| Doubao | ✅ | ✅ | ✅ | `ark.cn-beijing.volces.com/api/v3` |
| MiniMax | ✅ | ✅ | ✅ | `api.minimaxi.com/v1` |
| SenseNova | ✅ | ✅ | ❌ | `token.sensenova.cn/v1` |
| Ollama | ✅ | ❌ | ❌ | `localhost:11434/v1` |

### API Key Fallback Chain

```
custom model → {vendor}_key → text_api_key → agnes global key → agnes_extra_keys[0]
```

### Privacy & Security

- API keys are **encrypted at rest on desktop via Electron safeStorage** (`enc:v1:` prefix); they remain in memory as plaintext
- On mobile, or when no system credential store is available, they fall back to plaintext; the fields are cleared and the user is notified if the credential store breaks on a machine move
- **Local models (Ollama)** never leave your machine; other vendors' traffic goes to their respective APIs

---

## Known Limitations

- **Edge-TTS voiceover uses an undocumented endpoint**: Microsoft upgraded the read-aloud
  protocol in Oct 2026 (`Sec-MS-GEC-Version` `1-` → `2-`, token format changed). This plugin
  implements the v2 token and keeps v1 as a fallback — but `speech.platform.bing.com` is not a
  public API, has no SLA, and Microsoft may tighten it again. For business use, assess this
  yourself or use the **Audio8 local engine** / **Doubao** routes. Analysis:
  [`doc/40-edge-tts-v2-and-compliance.md`](doc/40-edge-tts-v2-and-compliance.md)
- **Desktop only**: voiceover, final render, Red Fruit Pro's render stage, and Talking-Head's render stage all require Node + an external ffmpeg binary
- **No streaming text output**: `requestUrl` does not support SSE
- **Large videos (>100 MB)**: read fully into memory, may OOM
- **Cover generation**: fonts are not bundled; Chinese rendering depends on the system font stack
- **Multi-reference images**: multiple URLs supported, but no "reference-board merge"
- **B-roll / InkOS / Canvas panels**: unit-tested and ffmpeg-chain verified, but not yet click-verified inside the Obsidian UI

---

## FAQ

**Q: The plugin enabled but nothing happens.**
Make sure your Obsidian version meets `minAppVersion` in `manifest.json` (currently ≥ 1.7.2).

**Q: ffmpeg not found.**
Run `ffmpeg -version` on the command line to confirm it is installed and on your PATH. Obsidian desktop inherits the system PATH — if it works on the CLI, it works inside Obsidian.

**Q: edge-tts returns 403.**
Three causes: ① Microsoft service outage (auto-detected, fails fast with a clear message); ② local clock drift (auto-corrected, session-scoped, up to ±5 minutes); ③ genuine credential error (rare, retried 3× with backoff).

**Q: How do I resume a broken pipeline?**
All three flagship pipelines (short-drama, Red Fruit Pro, Talking-Head) flush every step to disk. After a restart, the panel shows a "Resume from breakpoint" section listing past runs, newest first.

**Q: How do I update the plugin?**
Download the latest release and overwrite the four files under `.obsidian/plugins/agnes-ai-studio/`, then restart Obsidian. **Note**: settings live in `data.json`, which is not part of the zip — overwriting never loses your configuration.

---

## Third-party Notices

This plugin bundles no third-party source code. It does, however, reimplement one
protocol that is documented publicly and originated in an LGPL-licensed project, plus
it calls one undocumented Microsoft endpoint. Both deserve to be stated plainly.

**Edge TTS — protocol reimplemented, project under LGPLv3**

The free voiceover engine speaks the Edge browser's read-aloud WebSocket protocol.
The reference implementation is [rany2/edge-tts](https://github.com/rany2/edge-tts),
licensed **LGPLv3** (relicensed 2024-11-22; its README badge still says MIT — that is
stale). This plugin implements the protocol **independently** from the publicly
documented behaviour: the signing step calls Node's built-in `crypto.createHash`
rather than any code taken from that project, and the tick arithmetic is dictated by
Microsoft's own endpoint. Attribution is recorded in `src/core/tts.ts`.

If a future release ports the DRM/token algorithm from that project, it must ship as
LGPLv3 with the full GPL + LGPL texts — see `doc/40-edge-tts-v2-and-compliance.md`
for the full analysis and the three implementation options.

**Edge TTS — undocumented endpoint, no SLA**

`speech.platform.bing.com` is not a public API. Microsoft's Edge EULA forbids reverse
engineering and working around technical limitations. There is no SLA, no quota
commitment, and no guarantee of continued availability; the endpoint has been tightened
several times (403 waves, endpoint changes, and a token-format upgrade in October 2026).
**Using it is a fact of the moment, not a contractual grant.** Commercial deployments
should assess this themselves, or use the Audio8 local engine / Doubao routes instead.

For the complete analysis, see [`doc/40-edge-tts-v2-and-compliance.md`](doc/40-edge-tts-v2-and-compliance.md).

---

## Author & Support

- **Author**: xiaosong8584
- **GitHub**: <https://github.com/xiaosong8584>
- **Support / tips**: <https://afdian.com/a/xiaosong8584>
- **Bilibili**: <https://space.bilibili.com/505631203>
- **License**: MIT (see [`LICENSE`](./LICENSE))

---

## Release Notes

- Version history: [`CHANGELOG.md`](./CHANGELOG.md)
