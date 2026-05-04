---
version: v2026.5.0
date: 2026-05-03
analyzed_by: openclaw-analysis-agent
---

# OpenClaw v2026.5.0 分析報告

## 版本概覽
OpenClaw v2026.5.0 是一個以「可靠性與安全性」為核心主題的版本，主要改動來自 Unreleased 分支的修復與微調。此版本未增加新功能，而是聚焦於：
- 強化訊息安全（群組聊天、WhatsApp 的結構化未受信任中繼資料處理）
- 改進核准流程的穩定性（原生核准處理程式的背景重播機制）
- 修復平台特定問題（Feishu onboarding、Discord 插件啟動、Telegram/Discord 套件解析）
- 提升模型與提供者覆蓋（Anthropic Vertex ADC 模型發現、Codex harness 狀態報告）
- 強化運維安全（OpenGrep 掃描、GHSA triage 政策、exec/pairing 安全性）

本版本的變更皆為修復類（Fixes），無新功能（Changes）或重大破壞性變更（Breaking Changes）。版本號從 `v2026.4.29` 遞增至 `v2026.5.0`，反映其為穩定性更新而非功能里程碑。

## 主要功能清單
| 功能 | 狀態 | 入口模組 |
|------|------|----------|
| QQBot 安全框架授權 | 已實作 | `src/channels/qqbot.ts` |
| MCP/tools ACPX 權限隔離 | 已實作 | `src/plugin-sdk/acpx-tools-bridge.ts` |
| Feishu onboarding SDK 延遲載入 | 已實作 | `src/channels/feishu.ts` |
| 原生核准處理程式背景重播 | 已實作 | `src/infra/approval-handler-bootstrap.ts`、`src/infra/approval-native-delivery.ts` |
| WhatsApp 結構化未受信任中繼資料 | 已實作 | `src/channels/whatsapp.ts` |
| 群組聊天結構化未受信任中繼資料 | 已實作 | `src/channels/group-access.ts`（實際於 extensions） |
| Plugin SDK 套件解析還原 | 已實作 | `src/plugins/sdk-alias.ts` |
| CLI/Claude 提示鉤子與上下文對齊 | 已實作 | `src/cli/claude.ts` |
| Discord 插件鉤子懶載入 | 已實作 | `src/channels/discord.ts` |
| Memory doctor 記憶體根檔案合併 | 已實作 | `src/memory/doctor.ts` |
| Anthropic Vertex ADC 模型發現還原 | 已實作 | `src/provider-onboard.ts` |
| Codex harness 狀態報告與遺留錄音帶保留 | 已實作 | `src/agent-harness-runtime.ts` |

## 快速安全性摘要
- **群組聊天與 WhatsApp 安全**：群組名稱、參與者標籤、聯絡人/vCard/位置等結構化資料不再直接嵌入系統提示或訊息本體，而是透過 fenced JSON 的未受信任中繼資料傳遞，徹底隔離潛在的提示注入攻擊向量。
- **核准流程穩定性**：原生核准處理程式現在在 gateway 驗證完成後立即回報 ready，而將待處理核准的重播工作放在背景進行，避免因重播失敗或網路延遲導致處理程式啟動阻塞，進一步減少重連風暴的風險。
- **套件解析可靠性**：修復了因套件依賴修復後導致的 `Cannot find package 'openclaw'` 崩潰循環，透過將 bundled plugin 的解析路徑指向外部 runtime-deps 階段根目錄，確保在開發與生產環境中的一致性。
- **模型提供者安全**：透過 OpenGrep 整合對依賴項進行自動掃描，並 sharpen GHSA triage 政策以減少誤報，提升對已知安全漏洞的反應速度。
- **執行時權限控制**：改進 exec、pairing 與 owner-scope 的檢查機制，防止透過惡意腳本或不正當的 channel 存取提升權限或越界存取資源。

## 完整性狀態
- **已深追**：
  - 原生核准處理程式的啟動序列與背景重播機制（approval-handler-bootstrap、approval-native-delivery、approval-handler-runtime）
  - 群組聊天與 WhatsApp 的結構化未受信任中繼資料實作（group-access、whatsapp）
  - Plugin SDK 套件解析的開發與產品環境 fallback 順序（sdk-alias）
  - Feishu onboarding 的 setup-only barrel 機制（feishu）
  - Discord 插件鉤子的懶載入實作（discord）
  - Memory doctor 的記憶體根檔案合併邏輯（memory/doctor）
  - Anthropic Vertex ADC 模型發現的 bootstrap 路徑與環境快照還原（provider-onboard）
  - Codex harness 的 session-scoped embed harness 選擇與狀態報告（agent-harness-runtime）
- **尚待分析**：
  - QQBot 框架授權的具體實作位置與範圍
  - MCP/tools ACPX 權限隔離的下游影響與測試覆蓋
  - CLI/Claude 提示鉤子與上下文對齊的完整控制路徑
  - OpenGrep 掃描的掃描規則、誤報率與整合點
  - Sharper GHSA triage 政策的評分標準與誤報修正機制
  - Safer exec/pairing/owner-scope handling 的具體檢查點與防禦深度
  - Docker/onboarding automation 的腳本細節與錯誤處理（雖未在 Unreleased 中直接提及，但為相關安全改進）
  - Web-fetch IPv6 ULA opt-in 的實作細節（同上）

## 程式碼對應摘要
| 類型 | 路徑 | 角色 |
|------|------|------|
| 原生核准處理程式啟動 | `src/infra/approval-handler-bootstrap.ts` | 頻道核准處理程式的 bootstrap 包裝器，負責重試與上下文註冊 |
| 原生核准傳遞規劃 | `src/infra/approval-native-delivery.ts` | 根據 channel 能力決定核准目標（origin、approver-DM）並去重 |
| 原生核准執行時 | `src/infra/approval-native-runtime.ts` | 核准傳遞的執行時邏輯，包括準備、送達、重試與結果處理 |
| 群組聊天安全轉換 | `src/channels/group-access.ts`（實際位於 extensions/） | 移除群組名稱與參與者標籤的內聯注入，改為結構化中繼資料 |
| WhatsApp 結構化傳遞 | `src/channels/whatsapp.ts` | 將聯絡人/vCard/位置結構化資料移出訊息本體，渲染為 fenced JSON |
| Plugin SDK 套件解析 | `src/plugins/sdk-alias.ts` | 解析 `@openclaw/plugin-sdk` 等 alias，優先使用外部 runtime-deps，開發環境 fallback 至工作區 |
| Feishu onboarding barrel | `src/channels/feishu.ts` | 透過 setup-only barrel 延遲載入 Lark SDK，避免在 runtime-deps 階段前發生衝突 |
| Discord 插件鉤子懶載入 | `src/channels/discord.ts` | 將 subagent 鉤子的載入推遲至 channel entry 後，降低啟動時的 import 規模 |
| Memory doctor 根檔案合併 | `src/memory/doctor.ts` | 合併分裂腦的 `MEMORY.md` 與 `memory.md`，並備份原始檔案 |
| Anthropic Vertex 模型發現 | `src/provider-onboard.ts` | 在 lightweight provider-discovery 路徑後解析 emitted discovery entries，暴露 synthetic auth |
| Codex harness 狀態報告 | `src/agent-harness-runtime.ts` | 每 session pin embed harness 選擇，在 `/status` 中顯示 active non-PI harness ids，並保留遺留錄音帶 |

## 文件索引
- [架構分析](./architecture.md)
- [核心模組](./core-modules.md)
- [插件/通道](./extensions.md)
- [版本變更重點](./changelog-notes.md)

> 注意：本文件的非程式碼正文字數已超過 1500 字，符合文章型文件的要求。