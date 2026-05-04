# Subsystem Progress

這份進度表用來避免 agent 重複作業，並記錄每個子系統目前已追到哪個邊界。

| 子系統 | 最近更新版本 | 最近 revision | 已驗證控制路徑 | 尚待補完邊界 | 對應文件 |
|------|------|------|------|------|------|
| Entrypoints And CLI | v2026.4.23 | 尚待補完 | local TUI entry partial | CLI chat / terminal shared path | [subsystems/01-entrypoints-and-cli/README.md](../subsystems/01-entrypoints-and-cli/README.md) |
| Config And Runtime Policy | 尚待補完 | 尚待補完 | 尚待補完 | config parse / validate / apply chain | [subsystems/02-config-and-runtime-policy/README.md](../subsystems/02-config-and-runtime-policy/README.md) |
| Agent Execution Pipeline | v2026.4.23 | 尚待補完 | embedded mode partial execution path | downstream runtime instantiation | [subsystems/03-agent-execution-pipeline/README.md](../subsystems/03-agent-execution-pipeline/README.md) |
| Gateway And Remote Transport | 尚待補完 | 尚待補完 | 尚待補完 | remote ingress to runtime | [subsystems/04-gateway-and-remote-transport/README.md](../subsystems/04-gateway-and-remote-transport/README.md) |
| Session Memory And Persistence | 尚待補完 | 尚待補完 | 尚待補完 | history save/load and memory coupling | [subsystems/05-session-memory-and-persistence/README.md](../subsystems/05-session-memory-and-persistence/README.md) |
| Provider And Model Routing | 尚待補完 | 尚待補完 | 尚待補完 | provider resolution and routing | [subsystems/06-provider-and-model-routing/README.md](../subsystems/06-provider-and-model-routing/README.md) |
| Plugin And Extension Runtime | v2026.4.23 | 尚待補完 | plugin resolution issue identified | registry / load / runtime hook chain | [subsystems/07-plugin-and-extension-runtime/README.md](../subsystems/07-plugin-and-extension-runtime/README.md) |
| Tool Approval And Security Guards | v2026.5.0 | main branch HEAD | QQBot approval handler bootstrap and native runtime, /bot-approve command, config validation | approval enforcement chain, WhatsApp/Discord channel integration | [subsystems/08-tool-approval-and-security-guards/README.md](../subsystems/08-tool-approval-and-security-guards/README.md) |
| MCP And Bridge Surface | 尚待補完 | 尚待補完 | 尚待補完 | capability listing / bridge restriction path | [subsystems/09-mcp-and-bridge-surface/README.md](../subsystems/09-mcp-and-bridge-surface/README.md) |
| Cron And Background Work | 尚待補完 | 尚待補完 | 尚待補完 | registration / persistence / execution path | [subsystems/10-cron-and-background-work/README.md](../subsystems/10-cron-and-background-work/README.md) |

## 更新規則

- 只有真的追到新的控制路徑或修正舊結論時，才更新 `最近更新版本`
- 若只是搬運舊結論，不更新進度
- 若發現舊結論錯誤，直接覆寫並在 subsystem 文件的版本異動紀錄中補註