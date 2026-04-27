# OpenClaw Source-Backed Learning Primer

這個 repo 不再只是一組版本分析報告，而是要逐步長成一份像 system-design-primer 那樣、可持續擴寫的學習架構。

核心原則只有一個：

- 以 OpenClaw 在 source code 中的實際功能切片為主題
- 以原始碼、測試、第一方 docs、changelog 為證據
- 讓資深工程師可以沿著文件快速理解 OpenClaw 怎麼運作、哪些模組決定行為、怎麼安全地下手改功能

## Repo 用途

這份資料庫同時維護兩層內容：

1. 穩定的學習知識層
   - 以功能主題整理，而不是以名詞 glossary 整理
   - 說明功能入口、控制路徑、設定面、測試邊界、改寫風險
2. 版本證據層
   - 每個版本一份可回溯的分析快照
   - 用來支撐穩定知識層中的結論，並記錄版本演進

## 閱讀入口

- [學習指南](./study-guide.md)
- [功能地圖](./feature-map.md)
- [子系統地圖](./reference/subsystem-map.md)
- [短路徑](./paths/short.md)
- [中路徑](./paths/medium.md)
- [深路徑](./paths/deep.md)

## 功能主題

- [Run A Chat Session](./features/01-run-a-chat-session/README.md)
- [Configure Runtime Behavior](./features/02-configure-runtime-behavior/README.md)
- [Use Local TUI And Terminal Chat](./features/03-use-local-tui-and-terminal-chat/README.md)
- [Serve Gateway And Remote Clients](./features/04-serve-gateway-and-remote-clients/README.md)
- [Register Plugins And Extensions](./features/05-register-plugins-and-extensions/README.md)
- [Route Models And Providers](./features/06-route-models-and-providers/README.md)
- [Approve Tools And Guard Sensitive Actions](./features/07-approve-tools-and-guard-sensitive-actions/README.md)
- [Run MCP And Tool Bridges](./features/08-run-mcp-and-tool-bridges/README.md)
- [Persist Sessions Memory And State](./features/09-persist-sessions-memory-and-state/README.md)
- [Run Cron And Background Jobs](./features/10-run-cron-and-background-jobs/README.md)

## 子系統

- [Subsystem Overview](./subsystems/README.md)
- [Entrypoints And CLI](./subsystems/01-entrypoints-and-cli/README.md)
- [Config And Runtime Policy](./subsystems/02-config-and-runtime-policy/README.md)
- [Agent Execution Pipeline](./subsystems/03-agent-execution-pipeline/README.md)
- [Gateway And Remote Transport](./subsystems/04-gateway-and-remote-transport/README.md)
- [Session Memory And Persistence](./subsystems/05-session-memory-and-persistence/README.md)
- [Provider And Model Routing](./subsystems/06-provider-and-model-routing/README.md)
- [Plugin And Extension Runtime](./subsystems/07-plugin-and-extension-runtime/README.md)
- [Tool Approval And Security Guards](./subsystems/08-tool-approval-and-security-guards/README.md)
- [MCP And Bridge Surface](./subsystems/09-mcp-and-bridge-surface/README.md)
- [Cron And Background Work](./subsystems/10-cron-and-background-work/README.md)

## 參考索引

- [Source Map](./reference/source-map.md)
- [Testing Map](./reference/testing-map.md)
- [Version Index](./reference/version-index.md)
- [Version Delta Log](./reference/version-delta-log.md)
- [Subsystem Progress](./reference/subsystem-progress.md)
- [Diagram Index](./reference/diagram-index.md)

## 版本證據層

現有版本資料夾維持在 repo 根目錄，以 `v<version>/` 表示，每個版本目錄都是一份工程證據快照，而不是完整學習入口。

目前已存在：

- [v2026.1.5](./v2026.1.5/README.md)
- [v2026.1.5-1](./v2026.1.5-1/README.md)
- [v2026.1.5-2](./v2026.1.5-2/README.md)
- [v2026.4.23](./v2026.4.23/README.md)

## 內容擴寫原則

- 每次只深追 1 至 3 個高價值功能切片
- 每次至少同步檢查 1 個相關子系統是否需要補強
- 先補版本證據，再更新穩定學習文件
- 對功能主題與子系統主題都要維護版本異動紀錄
- 每次實質更新都要補 Mermaid 圖，讓讀者能先看圖再看文字
- 若證據不足，直接標記尚待補完
- 不用常見架構名詞替代實際控制路徑

## 下一步

後續 agent 執行時，應依照 [openclaw-analysis-prompt.md](./openclaw-analysis-prompt.md) 同時維護：

- 功能主題文件
- 版本分析文件
- 功能地圖與測試地圖等索引文件