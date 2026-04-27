# OpenClaw v2026.4.23 插件/通道分析

## 插件架構概覽
OpenClaw 使用插件系統來支援不同的提供者（如 Anthropic、OpenAI、xAI 等）和通道（如 Discord、Slack、WhatsApp 等）。插件位於 `extensions/` 目錄下，每個子目錄代表一個插件。

在 `extensions/` 目錄中，我們可以看到許多提供者和通道插件，例如：
- `anthropic/`、`openai/`、`xai/` 等提供者插件
- `discord/`、`slack/`、`whatsapp/` 等通道插件
- `browser/`、`memory-core/`、`pi-embedded-runner/` 等功能插件

## 主要 extension / provider / channel 列表
基於目錄結構，以下是部分主要插件（僅列出部分，因為數量眾多）：

### 提供者插件（Providers）
- anthropic: Anthropic Claude 模型
- openai: OpenAI GPT 模型
- xai: xAI Grok 模版
- google: Google 模型（包括 Gemini）
- mistral: Mistral AI 模型
- groq: Groq 提供的模型
- together: Together AI 模型
- vllm: vLLM 本地模型服務
- lmstudio: LM Studio 本地模型服務
- ollama: Ollama 本地模型服務

### 通道插件（Channels）
- discord: Discord 通訊平台
- slack: Slack 通訊平台
- whatsapp: WhatsApp 通訊平台
- telegram: Telegram 通訊平台
- message: 內部訊息系統
- email: 電子郵件
- sms: 簡訊

### 功能插件（Functional Plugins）
- browser: 网页浏览和自动化
- memory-core: 記憶和回憶系統
- pi-embedded-runner: Pi 嵌入式運行器（用於 Raspberry Pi 等設備）
- active-memory: 主動記憶功能
- skill-workshop: 技能工作坊（用於學習和改進代理行為）
- camsnap: 相機快照捕捉
- video-frames: 影片幀擷取
- audio: 音訊處理
- image-generation: 圖像生成

## plugin SDK / contract 證據
插件契約的實作證據可以在以下檔案中找到：
- `src/plugins/runtime.js`：定義了插件運行時介面和註冊機制
- `src/plugins/embedded-extension-factory.ts`：提供了 Pi 嵌入式運行器的擴充工廠契約
- `@mariozechner/pi-coding-agent`：第三方套件，提供了 Pi 編碼代理的擴充點（用於 Pi 嵌入式運行器）

具體來看：
- `src/plugins/runtime.js` 包含 `getActivePluginRegistry()` 函式，用於取得目前作用中的插件註冊表。
- `src/plugins/embedded-extension-factory.ts` 定義了 `listEmbeddedExtensionFactories()` 函式，返回 active plugin registry 中的 embedded extension factories。

這些檔案證據顯示插件系統允許插件註冊延伸點（特別是對於 Pi 嵌入式運行的延伸）。

## 本版變更摘要
根據 changelog，在 v2026.4.22 和 Unreleased 變更中，與插件相關的變更包括：

### v2026.4.22
- **Plugin SDK/Pi embedded runs**: 添加了一個插件內嵌延伸工廠縫（seam），使得原生插件可以透過非同步運行鉤子（例如 `tool_result` 處理）來延伸 Pi 嵌入式運行，而不是回退到舊的同步持久化路徑。（#69946）Thanks @vincentkoc.
- **ACX**: 新增了一個顯式的 `openClawToolsMcpBridge` 選項，該選項會注入一個核心 OpenClaw MCP 伺服器，用於選取的內建工具，起始是 `cron`。
- **Plugin SDK/STT**: 分享實時轉錄 WebSocket 傳輸和多部分批次轉錄表單協助函式，跨捆綁 STT 提供者，減少提供者 plugin 樣板代碼，同時保留代理捕捉、重新連接、音訊隊列、關閉刷新和準備握手。

### Unreleased（即將包含在 v2026.4.23 中的變更）
- **MCP/tools**: 阻止 ACPX OpenClaw 工具橋列出或呼叫所有者專用工具（如 `cron`），以關閉非所有者 MCP 呼叫者的特權升級路徑。（#70698）Thanks @vincentkoc.
- **QQBot/security**: 需要框架認證才能使用 `/bot-approve`，使未授權的 QQ 發送者無法透過未經驗證的預調度 slash-command 路徑更改執行批准設定。（#70706）Thanks @vincentkoc.
- **Plugins/startup**: 恢復打包插件 `openclaw/plugin-sdk/*` 的解析，從打包安裝和外部運行時依賴階段根目錄，使得 Telegram/Discord 不再因缺少依賴修復而崩潰循環。（#70339）Thanks @andrejtr.

### 與 TUI 嵌入模式相關的插件變更
儘管 TUI 嵌入模式主要涉及核心執行階段，但插件系統仍然相關，因為：
- 嵌入模式仍然必須尊重 plugin approval 閘道（如 changelog 2026.4.22 所述：「while keeping plugin approval gates enforced」）。
- 插件可以透過 `embedded-extension-factory` 來延伸 Pi 嵌入式運行，這可能影響在嵌入模式下運行的代理行為（特別是當使用 Pi 嵌入式運行器時）。

## 已知限制
- 插件系統非常龐大，有超過 100 個插件目錄。完整列出所有插件及其版本變更超出了此分析的範圍。
- 本分析未深入檢查特定插件的內部實作，僅基於目錄結構和 changelog 中的插件相關變更。
- 插件如何具體影響 TUI 嵌入模式（例如透過 plugin approval 閘道）需要進一步檢查核心 agent 指令處理代理（`agentCommandFromIngress`）以及插件如何與之互動。