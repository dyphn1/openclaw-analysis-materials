---
version: v2026.5.7
date: 2026-05-07
analyzed_by: openclaw-analysis-agent
---

# OpenClaw v2026.5.7 插件層分析 – Discord 擴充套件

## 1. 目錄概覽

Discord 插件位於 `extensions/discord/` 目錄，負責將 OpenClaw 的抽象訊息模型映射為 Discord API 呼叫。主要子模組包括：

- **`src/monitor/`**：處理 inbound（來自 Discord 的訊息）與 outbound（OpenClaw 發送的訊息）上下文建構。
- **`src/send.*`**：實作訊息發送、編輯、刪除、搜尋與線程管理。
- **`src/audit-core.ts`**：權限稽核工具，確保機器人在頻道和語音頻道擁有必要的權限。
- **`src/session-key-normalization.ts`**：正規化 session key，用於決定訊息是發送到 **direct**（私訊）還是 **channel**（頻道）。
- **測試檔案**：對上述核心程式碼提供單元測試，確保行為在不同情境下的正確性。

在本版本（v2026.5.7）中，最重要的變更集中於 **`session-key-normalization.ts`**（訊息目標解析）以及 **`audit-core.ts`**（語音頻道權限稽核）的增強。

## 2. 變更概述

| 檔案 | 變更類型 | 主要內容 | 行號範圍（近似） |
|------|----------|----------|-----------------|
| `src/audit-core.ts` | 功能擴充 | 新增 `resolveRequiredDiscordChannelPermissions`，根據頻道類型（文字或語音）返回不同的必填權限集合；同時支援 `voice.autoJoin` 設定的頻道稽核。 | 1‑45, 70‑112 |
| `src/audit.test.ts` | 測試新增 | 新增 `collectDiscordAuditChannelIdsForAccount` 測試，驗證語音頻道的收集邏輯。 | 1‑20, 140‑200 |
| `src/session-key-normalization.ts` | 程式邏輯調整 | 改進正規化條件，僅在 **直接訊息 (direct)** 的上下文且 **發送者 ID 與頻道 ID 完全匹配** 時才將 `discord:channel:<id>` 轉為 `discord:direct:<id>`，防止跨頻道訊息被錯誤視為 DM。 | 12‑63 |
| `src/session-key-normalization.test.ts` | 測試更新 | 新增測試案例驗證「phatom discord:channel」轉換行為，同時保留對非 direct 情境的原樣返回。 | 1‑40 |

以上變更直接對應到 CHANGELOG 中的條目：
> Discord/message: parse provider‑prefixed targets like `discord:channel:<id>` as channel sends instead of legacy Discord DM targets, so cross‑channel agent `message(action="send")` calls no longer misroute channel IDs into misleading `Unknown Channel` failures. (Fixes #78572)

## 3. 重要模組深度解析

### 3.1 `session-key-normalization.ts`

**目的**：在 inbound 與 outbound 訊息的處理階段，根據使用者的會話上下文（`ChatType`、`From`、`SenderId`）將原始的 session key 正規化為 OpenClaw 能夠識別的形式。

**核心函式**：`normalizeExplicitDiscordSessionKey(sessionKey, ctx)`
- **步驟**：
  1. 先將 `sessionKey`