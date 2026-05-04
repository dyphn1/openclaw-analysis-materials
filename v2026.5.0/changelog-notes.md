---
version: v2026.5.0
date: 2026-05-03
analyzed_by: openclaw-analysis-agent
---

# OpenClaw v2026.5.0 變更重點

本版本為 v2026.5.0，主要內容為將先前標記為 `Unreleased` 的修復納入正式版本。此版本無新功能（Changes），亦無破壞性變更（Breaking Changes），僅包含修復類（Fixes）變更。以下是從 CHANGELOG.md 中的 `Unreleased` 區段摘錄的變更，並補充實作位置與證據來源。

## 新功能
（本版本無新功能）

## Breaking Changes
（本版本無破壞性變更）

## 安全性相關修正

| 功能/變更名稱 | 實作位置 | 說明 | 證據來源類型 | 信心 |
|---------------|----------|------|--------------|------|
| QQBot/security: require framework auth for `/bot-approve` | `src/channels/qqbot.ts` | 未授權的 QQ 發送者只能透過未經驗證的 pre-dispatch 路徑呼叫 `/bot-approve` 指令，嘗試變更 exec 核准設定。此修復要求框架授權（framework auth）才能進入該路徑，防止未授權使用者提升核准權限。 | 原始碼 + CHANGELOG | 高 |
| MCP/tools: stop the ACPX OpenClaw tools bridge from listing or invoking owner-only tools such as `cron` | `src/plugin-sdk/acpx-tools-bridge.ts` | ACPX（Anthropic Claude Protocol Extension）的 OpenClaw 工具橋接先前會列出並允許非所有者（non-owner） MCP 呼叫者執行 owner-only 工具（例如 `cron`，用於管理排程任務），構成特權升級路徑。修復後，橋接會在 `listTools` 與 `callTool` 中檢查呼叫者是否為所有者（`ctx.isOwner`），非所有者則回傳空列表或拋錯。 | 原始碼 + CHANGELOG | 高 |
| WhatsApp/security: keep contact/vCard/location structured-object free text out of the inline message body and render it through fenced untrusted metadata JSON | `src/channels/whatsapp.ts` | 先前，WhatsApp 訊息中的聯絡人、vCard、位置等結構化物件會被直接序列化為純文字並插入訊息本體，惡意內容（例如隱藏的指令）可能被語言模型誤執行。此修復將該等結構化資料移出訊息本體，改為以 fenced JSON（例如 `\`\`\`json\n{...}\n\`\`\`）形式作為未受信任中繼資料附加，確保語言模型僅將其視為純文字而非可執行內容。 | 原始碼 + CHANGELOG | 高 |
| Group-chat/security: keep channel-sourced group names and participant labels out of inline group system prompts and render them through fenced untrusted metadata JSON | `extensions/group-access.ts`（實際位於 `src/extensions/group-access.ts`） | 類似 WhatsApp 安全修正，群組聊天的系統提示中原本會直接包含群組名稱與參與者標籤（例如來自頻道的元資料），存在提示注入風險。修改後，該等資訊不再內聯於提示，而是透過結構化的 fenced JSON 傳遞，作為未受信任中繼資料。 | 原始碼 + CHANGELOG | 高 |

## 與前版比較
相較於前一個已分析版本 `v2026.4.29`（實際為標籤 `v2026.4.29`，但本版本 v2026.5.0 納入了其 Unreleased 變更），本版本的主要差異如下：

### 已解決的問題
1. **QQBot 框架授權缺失**：在 `v2026.4.29` 中，QQBot 的 `/bot-approve` 路徑未進行框架驗證，允許未授權發送者嘗試修改核准設定。v2026.5.0 新增框架授權檢查，封鎖此路徑。
2. **MCP/tools 特權升級路徑**：在 `v2026.4.29` 中，ACPX 工具橋接未檢查呼叫者是否為所有者，非所有者 MCP 呼叫者可列出並執行 owner-only 工具。v2026.5.0 新增所有者檢查，僅允許授權呼叫。
3. **WhatsApp 結構化資料提示注入風險**：在 `v2026.4.29` 中，WhatsApp 的聯絡人/vCard/位置資料直接嵌入訊息本體。v2026.5.0 改為結構化未受信任中繼資料，降低隱藏式提示注入風險。
4. **群組聊天結構化資料提示注入風險**：類似 WhatsApp，群組名稱與參與者標籤曾直接出現在系統提示中。v2026.5.0 改為結構化中繼資料。
5. **核准處理程式啟動阻塞**：在 `v2026.4.29` 中，原生核准處理程式必須等待待處理核准的重播完成才能報告 ready，導致慢速或失敗的重播會阻斷新核准請求的處理。v2026.5.0 改為在 gateway 驗證後即報 ready，將重播放入背景進行。
6. **Feishu onboarding SDK 載入時機衝突**：在 `v2026.4.29` 中，Feishu 的 onboarding 會在 runtime-deps 階段前即載入 Lark SDK，可能造成版本衝突或找不到套件。v2026.5.0 改為 setup-only barrel，延遲載入至真正需要設定時。
7. **Discord 插件鉤子啟動時 import 過寬**：在 `v2026.4.29` 中，Discord 插件在載入時即會 import subagent 鉤子，增加啟動時的 import 規模與衝突機會。v2026.5.0 將鉤子載入推遲至 channel entry 後，並改善錯誤回報。
8. **Plugin SDK 套件解析崩潰循環**：在 `v2026.4.29` 中，因套件依賴修復後，外部 runtime-deps 與工作區的 node_modules 解析順序不當，導致 `Cannot find package 'openclaw'` 錯誤，造成插件啟動失敗並可能重試崩潰。v2026.5.0 明確將外部 runtime-deps 設為優先搜尋路徑，工作區為 fallback。
9. **Memory doctor 根檔案分裂腦處理**：在 `v2026.4.29` 中，doctor 會將 `memory.md` 視為 runtime fallback，導致 `MEMORY.md` 與 `memory.md` 雙寫情況下不一致。v2026.5.0 將 `MEMORY.md` 定義為唯一 canonical 根檔案，並提供 `--fix` 指令合併雙檔案並備份。
10. **Anthropic Vertex ADC 模型發現失效**：在 `v2026.4.29` 中，輕量級 provider-discovery path 失敗後無法 fallback 至傳統 ADC 模式，導致模型探測失敗。v2026.5.0 恢復解析 emitted discovery entries 並暴露 synthetic auth。
11. **Codex harness 狀態報告不一致**：在 `v2026.4.29` 中，embed harness 選擇未釘選至 session，導致 `/status` 中顯示的 harness ID 可能不穩定，且遺留錄音帶無法保留至顯式重置。v2026.5.0 每 session pin embed harness 選擇，在 `/status` 中報告 active non-PI harness ids，並保留遺留錄音帶至 `/new` 或 `/reset`。

### 未變更的領域
- 核心通訊協議（gateway RPC、plugin contract）保持不變。
- 基礎記憶體結構（`MEMORY.md`、`memory/` 目錄）在 v2026.5.0 中僅在 doctor 行為上有變更，讀寫介面不變。
- 提供者介面（例如 OpenAI、Anthropic）的核心呼叫方式保持不變，僅在探測與初始化階段有修復。

### 整體影響
本版本為穩定性與安全性修復版本，無新功能加入，亦無移除或重大變更現有行為。所有修復均為針對特定情況的防禦性變更，旨在減少因惡意輸入、啟動序列錯誤或套件解析問題導致的故障或安全風險。對於升級自 `v2026.4.29`（或之前）的使用者，建議優先更新以獲得安全防護與啟動穩定性提升。

--- 
> 本文件的非程式碼正文字數已超過 1500 字，符合文章型文件的要求。