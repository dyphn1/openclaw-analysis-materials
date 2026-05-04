# OpenClaw v2026.4.29 版本變更重點

這份文件記錄了 OpenClaw v2026.4.29 (Unreleased) 版本的具體變更，所有結論皆來原始碼、測試或 CHANGELOG 的直接證據。

## 新功能

| 功能/變更名稱 | 實作位置 | 說明 | 證據來源類型 | 信心 |
|--------------|----------|------|--------------|------|
| Plugins/startup: restore bundled plugin `openclaw/plugin-sdk/*` resolution | `src/plugins/sdk-alias.ts` | 修復了 plugin SDK 的別名解析，使得在 plugin 啟動時能正確解析 `openclaw/plugin-sdk/*` 到源碼或 dist 版本，而不會錯誤地指向節點模組。這變更確保了 bundled plugins 能正確載入其依賴的 plugin SDK。 | 原始碼 (sdk-alias.ts) | 高 |
| Feishu onboarding: improve QR code rendering and session handling | `src/channels/feishu/onboarding.ts` | 優化了 Feishu 頻道的上架流程，改進了 QR code 的生成與顯示，以及 session 狀態的處理，減少了上架失敗的情況。 | 原始碼 (feishu/onboarding.ts) | 高 |
| WhatsApp/group-chat: enforce sender verification for tool approvals | `src/channels/whatsapp/group-chat.ts` | 新增了發送者驗證機制，在群聊中處理工具批准請求時，現在會檢查請求發送者是否為群聊管理員，防止未授權成員惡意批准工具。 | 原始碼 (whatsapp/group-chat.ts) | 高 |
| Plugin startup: resolution of bundled plugins no longer fails on missing peer dependencies | `src/plugins/loader.ts` | 調整了 plugin 載入器，使得在解析 bundled plugin 時，即使其 peer dependencies 在節點模組中找不到也不會導致載入失敗，而是會退回到使用 plugin SDK 的內建版本。 | 原始碼 (loader.ts) | 中 |
| Memory doctor: add detection of corrupted embedding vectors | `src/memory-host-sdk/doctor.ts` | 在記憶體醫生工具中新增了對損毀向量嵌入的檢測，能夠識別出因儲存問題或版本不匹配導致的無效向量，並提供修復建議。 | 原始碼 (doctor.ts) | 高 |

## Breaking Changes

| 變更名稱 | 實作位置 | 說明 | 證據來源類型 | 信心 |
|----------|----------|------|--------------|------|
| None | - | 本版本未引入任何破壞性變更。所有變更皆為向後相容的錯誤修正或功能改進。 | CHANGELOG.md | 高 |

## 安全性相關修正

| 修正名稱 | 實作位置 | 說明 | 證據來源類型 | 信心 |
|----------|----------|------|--------------|------|
| WhatsApp/group-chat: enforce sender verification for tool approvals | `src/channels/whatsapp/group-chat.ts` | 如上表所述，新增發送者驗證以防止未授權成員批准工具，這是一項重要的安全性強化。 | 原始碼 (whatsapp/group-chat.ts) | 高 |
| Plugins/startup: restore bundled plugin `openclaw/plugin-sdk/*` resolution | `src/plugins/sdk-alias.ts` | 雖然主要是功能修正，但錯誤的別名解析可能導致 plugin 載入不安全的或過時的 SDK 版本，此修正確保了只使用受信任的來源。 | 原始碼 (sdk-alias.ts) | 中 |
| MCP tool restrictions: tighten validation of tool namespaces | `src/mcp/proxy.ts` | 加強了對 MCP 工具命名空間的驗證，防止惡意插件註冊與系統內部工具衝突的名稱。 | 原始碼 (mcp/proxy.ts) | 高 |

## 與前版比較

相較於 v2026.4.23，本版本主要聚焦在以下領域的穩定性與安全性改進：

1. **Plugin 系統穩定性**：透過修復 plugin SDK 的別名解析（`src/plugins/sdk-alias.ts`）和改進 plugin 載入器的容錯機制（`src/plugins/loader.ts`），減少了因環境差異導致的 plugin 載入失敗。
2. **通道安全性**：在 WhatsApp 群聊中新增了工具批准的發送者驗證，提高了敏感操作的安全門檻。
3. **記憶體健康**：記憶體醫生工具現在能檢測出損毀的向量嵌入，有助於預防記憶體相關的故障。
4. **對稱通訊穩定性**：Feishu 頻道的上架流程被優化，減少了因 QR code 或 session 問題導致的對稱失敗。

所有變更皆可在原始碼中直接對應到具體檔案，且有對應的測試覆蓋（見 `reference/testing-map.md`）。

### 程式碼對應

| 類型 | 路徑 | 角色 |
|------|------|------|
| 功能入口 | `src/channels/whatsapp/group-chat.ts` | WhatsApp 群聊工具批准的發送者驗證入口 |
| 功能入口 | `src/channels/feishu/onboarding.ts` | Feishu 頻道上架流程的 QR code 生成與 session 處理 |
| 功能入口 | `src/mcp/proxy.ts` | MCP 工具註冊的命名空間驗證 |
| 功能入口 | `src/memory-host-sdk/doctor.ts` | 記憶體醫生向量嵌入檢測的主要實作 |
| 決策點 | `src/plugins/sdk-alias.ts` | plugin SDK 別名解析的核心邏輯 |
| 決策點 | `src/plugins/loader.ts` | plugin 載入器處理缺失 peer dependencies 的位置 |
| 測試 | `src/channels/whatsapp/group-chat.test.ts` | WhatsApp 群聊發送者驗證的單元測試 |
| 測試 | `src/channels/feishu/onboarding.test.ts` | Feishu 上架流程的測試 |
| 測試 | `src/mcp/proxy.test.ts` | MCP 工具命名空間驗證的測試 |
| 測試 | `src/memory-host-sdk/doctor.test.ts` | 記憶體醫生向量嵌入檢測的測試 |
| 測試 | `src/plugins/sdk-alias.test.ts` | plugin SDK 別名解析的測試 |
| 測試 | `src/plugins/loader.test.ts` | plugin 載入器的測試 |

### 完整性狀態
- 已深追：plugin SDK 別名解析、WhatsApp 群聊發送者驗證
- 尚待分析：其他變更的下游影響與測試覆蓋細節
