# 學習指南

這份 repo 的目標不是把 OpenClaw 講成一串名詞，而是把它拆成工程師真的會修改、排錯、擴充的功能切片。

## 建議讀法

### 第一輪

- 先看 [README.md](./README.md) 了解 repo 分層
- 再看 [feature-map.md](./feature-map.md) 知道每個功能主題對應什麼問題
- 然後從 [Run A Chat Session](./features/01-run-a-chat-session/README.md) 開始，建立主流程感

### 第二輪

- 依序補上設定、TUI、Gateway、Plugins、Providers
- 對照 [子系統地圖](./reference/subsystem-map.md) 看這些功能分別落在哪些內部結構
- 遇到不確定的結論，回查對應版本目錄中的 changelog-notes、core-modules 與 architecture

### 第三輪

- 依照你要改的功能，直接跳到對應 feature 主題
- 再用版本證據層確認該功能最近一次重大變更與測試邊界
- 最後回到對應子系統文件，確認你改動的邊界是否會波及其他功能

## 三種學習目標

### 快速進入維護狀態

目標：能回答「入口在哪裡、哪裡決定行為、哪裡看設定」

建議路線：

- [paths/short.md](./paths/short.md)

### 能安全改功能

目標：能追控制路徑、理解驗證點與風險點

建議路線：

- [paths/medium.md](./paths/medium.md)

### 能規劃大改動或新擴充

目標：能看懂 plugins、providers、state、approval、MCP 等跨模組邊界

建議路線：

- [paths/deep.md](./paths/deep.md)

## 這份資料怎麼和版本分析配合

- 穩定學習文件回答「這個功能一般是怎麼運作」
- 版本證據文件回答「這個版本實際改了什麼、哪裡有證據、哪些地方還沒追完」
- 子系統文件回答「這組功能背後有哪些共享邊界與內部責任分工」

任何高階結論，都應能回到某個 feature 文件、某個 subsystem 文件與某個 `v<version>/` 文件。