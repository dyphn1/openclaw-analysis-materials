---
version: v2026.5.3
date: 2026-05-04
analyzed_by: openclaw-analysis-agent
---

# OpenClaw v2026.5.3 核心模組分析：File Transfer Plugin

## 職責定義

File Transfer Plugin（`extensions/file-transfer`）負責在 OpenClaw 系統中的節點（paired nodes）之間提供安全、受控的檔案操作能力。其核心職責包括：

1. **節點層級檔案讀取**：透過 `file_fetch` 工具，從指定節點的絕對路徑讀取檔案內容，並根據 MIME type 自動決定是內嵌回傳（小文字檔案、圖片）或保存至 Gateway 媒體存儲。
2. **節點層級目錄列舉**：透過 `dir_list` 工具，取得節點上特定目錄的結構化清單（檔名、大小、修改時間、MIME type 等），不傳輸實際檔案內容。
3. **節點層級目錄打包下載**：透過 `dir_fetch` 工具，將節點上的目錄打包為 gzipped tarball，在 Gateway 上解壓縮並返回儲存的檔案 manifest，適合一次性取得原始碼樹或資產資料夾。
4. **節點層級檔案寫入**：透過 `file_write` 工具，將位元組（可來自先前 `file_fetch` 的媒體 ID 或直接 Base64）寫入節點上的絕對路徑，支援 atomic write（暫存檔案＋重新命名），並預設禁止覆寫既有檔案。
5. **安全政策與操作員核准**：所有節點層級操作必須經過雙層防護：（a）節點與路徑的政策評估（可讀/可寫路徑清單、符號連結限制）；（b）操作員即時核准（一次性或永久允許）。
6. **審計與追蹤**：每一次檔案操作都會寫入附加式審計日誌（`~/.openclaw/audit/file-transfer.jsonl`），記錄決策、路徑、節點、大小、SHA256 等資訊，支援事後追蹤與異常偵測。

該插件不負責節點的發現或連線維護，這些功能由節點管理子系統（`gateway-and-remote-transport`）提供；插件僅假設節點 ID 已經被解析為可達的節點，並透過 `node.invoke` RPC 向節點端的節點主機命令伺服器發出指令。

## 關鍵介面與型別定義

### 插件入口點（`extensions/file-transfer/index.ts`）

插件透過 `definePluginEntry` 註冊四個 `nodeHostCommands`（`file.fetch`、`dir.list`、`dir.fetch`、`file.write`）與四個對應的 agent 工具（`file_fetch`、`dir_list`、`dir_fetch`、`file_write`）。節點主機命令的實作位於 `src/node-host/` 目錄下的四個 TypeScript 檔案，每個檔案實作單一命令的處理