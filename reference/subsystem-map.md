# Subsystem Map

這份地圖用來把功能主題映射到較穩定的內部子系統，方便 agent 知道分析某個 feature 時應同步回寫哪份 subsystem 文件。

| 子系統 | 說明 | 相關功能主題 |
|------|------|------|
| Entrypoints And CLI | CLI / terminal / TUI 等使用者入口與命令分流 | Run A Chat Session, Use Local TUI And Terminal Chat |
| Config And Runtime Policy | defaults、parse、validate、override、policy gate | Configure Runtime Behavior |
| Agent Execution Pipeline | agent turn、payload、runtime dispatch、response stream | Run A Chat Session, Use Local TUI And Terminal Chat |
| Gateway And Remote Transport | remote ingress、RPC、gateway session path | Serve Gateway And Remote Clients |
| Session Memory And Persistence | session history、memory、state save/load | Persist Sessions Memory And State |
| Provider And Model Routing | provider selection、model listing、routing policy | Route Models And Providers |
| Plugin And Extension Runtime | plugin registry、extension loading、embedded extension seams | Register Plugins And Extensions |
| Tool Approval And Security Guards | approval gate、owners-only behaviors、framework auth | Approve Tools And Guard Sensitive Actions |
| MCP And Bridge Surface | MCP server, tool bridges, exposed capability surfaces | Run MCP And Tool Bridges |
| Cron And Background Work | cron, async jobs, background execution | Run Cron And Background Jobs |

## 使用規則

- 每次 feature 更新後，至少要確認它對應的 subsystem 是否需要同步更新
- 若一個 feature 橫跨多個 subsystem，優先更新真正決定行為的那個 subsystem，其他寫在尚待補完或關聯欄位