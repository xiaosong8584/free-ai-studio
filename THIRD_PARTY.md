# Third-Party Notices

本插件包含或调用以下第三方组件。**权重、运行时依赖均不在本仓库分发**，用户自行按需下载或安装。

## MOSS-TTS-Nano（Apache-2.0）

- **来源**：https://github.com/OpenMOSS/MOSS-TTS-Nano
- **权重（Apache-2.0，不随本插件分发）**：
  - `OpenMOSS-Team/MOSS-TTS-Nano-100M-ONNX`（HuggingFace / ModelScope）
  - `OpenMOSS-Team/MOSS-Audio-Tokenizer-Nano-ONNX`（HuggingFace / ModelScope）
- **许可证全文**：Apache License, Version 2.0
- **引用来源**：本插件 `scripts/moss/sidecar.py` 通过 `PYTHONPATH` 外部调用 `onnx_tts_runtime.OnnxTtsRuntime`，不复现其推理核心代码。

许可证摘要（完整条款见上游 `LICENSE`）：

> Copyright (c) 2026 OpenMOSS Team
>
> Licensed under the Apache License, Version 2.0 (the "License");
> you may not use this file except in compliance with the License.
> You may obtain a copy of the License at
>
> http://www.apache.org/licenses/LICENSE-2.0

## FastAPI（MIT）

- 来源：https://github.com/fastapi/fastapi
- 用途：MOSS-TTS 本地 sidecar 的 HTTP 服务壳

## Uvicorn（BSD-3）

- 来源：https://github.com/encode/uvicorn
- 用途：MOSS-TTS 本地 sidecar 的 ASGI 服务器

## onnxruntime（MIT）

- 来源：https://github.com/microsoft/onnxruntime
- 用途：MOSS-TTS ONNX 模型推理引擎（用户端 venv 依赖）

## numpy（BSD-3）

- 来源：https://github.com/numpy/numpy
- 用途：音频数组处理

## soundfile（MIT / Apache-2.0 dual）

- 来源：https://github.com/soundfile/soundfile-python
- 用途：WAV 读写

## scipy（BSD-3）

- 来源：https://github.com/scipy/scipy
- 用途：`scipy.signal.resample_poly` 采样率转换

## Obsidian API（MIT）

- 来源：https://github.com/obsidianmd/obsidian-api
- 用途：本插件运行环境

---

## 合规声明

本插件为本地桌面 TTS 工具，用户自行承担《互联网信息服务深度合成管理规定》下的内容合规义务：

1. **声纹授权**：使用他人声音作为参考音频（`prompt_audio_path`）须获得**授权人单独书面同意**（《民法典》第 1023 条、《个人信息保护法》第 28–29 条）。
2. **生成标识**：向公开渠道发布合成内容时，须按《深度合成管理规定》§16–§17 与《人工智能生成合成内容标识办法》**显著标注"AI 生成"**；本 sidecar 已在输出 WAV 文件头部嵌入 `LIST/INFO/AIMD` 隐式标识作为元数据标记。
3. **禁止用途**：不得用于冒充他人、电信诈骗、造谣抹黑、伪造公共人物言论等违反法律法规的场景。
4. **备案**：本插件为本地工具，不作为"深度合成服务提供者"对外提供服务；如用户二次开发为云端/多租户服务，须自行履行《深度合成管理规定》§19 备案义务。

## 反馈与授权问题

关于上游许可证、权重来源、合规条款的问题，请优先阅读：
- 上游仓库 LICENSE：`https://github.com/OpenMOSS/MOSS-TTS-Nano/blob/main/LICENSE`
- 本仓库 LICENSE（MIT）：`./LICENSE`
