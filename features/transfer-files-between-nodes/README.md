# Transfer Files Between Nodes（檔案在節點間傳輸）

## 這個功能在系統中的角色

`file-transfer` 插件為 OpenClaw 提供了跨節點的二進位檔案操作能力。它讓使用者可以 **從一個已配對的遠端節點讀取檔案、列出目錄、打包目錄或寫入檔案**，而所有操作都經過嚴格的安全政策與操作員批准流程。這個功能在以下情境中特別重要：

- **檢索截圖/螢幕錄影**：使用 `file_fetch` 把遠端節點的螢幕快照拉回 Gateway，然後在對話中直接顯示。
- **同步設定或日誌**：透過 `dir_fetch` 把遠端目錄匯出為 tarball，供後續分析或備份。
- **遠端部署或補丁**：使用 `file_write` 把新二進位或腳本寫入遠端節點，配合 `agent-runtime` 的指令執行。
- **安全審計**：所有檔案操作都會寫入審計日誌，並支援「一次批准、永久允許」的 `allow-always` 機制。

此功能把檔案傳輸提升到與聊天、工具、模型回覆同等的第一級別，讓開發者和運維人員可以在同一個交互式介面中完成跨機器的檔案管理，避免額外的 SSH、SCP 或第三方工具。它同時遵循 OpenClaw 的 **安全預設拒絕** 原則：除非在 `plugins.entries.file-transfer.config.nodes` 中明確允許，否則任何檔案讀寫請求都會被阻止並返回錯誤，確保節點不會因為錯誤的路徑或惡意指令而泄漏敏感資訊。

## 使用者入口

| 入口類型 | 觸發方式 | 相關指令或工具 |
|----------|----------|----------------|
| CLI | `openclaw tools file_fetch --node <node> --path <abs-path>` | `file_fetch`、`dir_list`、`dir_fetch`、`file_write` |
| Agent Tool (在對話中) | 透過 `file_fetch`、`dir_list`、`dir_fetch`、`file_write` 代理工具呼叫 | 同上，工具會自動顯示於 `tools` 清單 |
| API / Gateway RPC | `node.invoke` RPC 呼叫 `file.fetch`、`dir.list`、`dir.fetch`、`file.write` | `src/extensions/file-transfer/index.ts` 的 `nodeHostCommands` |
| UI / TUI | 在 TUI 中點選「Download」或「Upload」按鈕 (由 UI 組件呼叫相同的工具) | 內部使用 `callGatewayTool` 介面 |

## 對應子系統

- **Plugin And Extension Runtime