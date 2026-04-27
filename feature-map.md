# 功能地圖

這份地圖用來把 OpenClaw 拆成可學、可改、可驗證的功能主題。

| 功能主題 | 核心問題 | 穩定文件 | 對應子系統 | 常見入口面 | 主要版本證據來源 |
|------|------|------|------|------|------|
| Run A Chat Session | 一次聊天互動如何從入口走到 agent 執行 | [features/01-run-a-chat-session/README.md](./features/01-run-a-chat-session/README.md) | Entry, Execution, Session | CLI, chat UI, gateway ingress | `v*/core-modules.md` |
| Configure Runtime Behavior | config、env、runtime override 怎麼裁決 | [features/02-configure-runtime-behavior/README.md](./features/02-configure-runtime-behavior/README.md) | Config And Runtime Policy | config load, CLI flags, defaults | `v*/architecture.md` |
| Use Local TUI And Terminal Chat | local TUI 與 terminal 模式如何接本地 runtime | [features/03-use-local-tui-and-terminal-chat/README.md](./features/03-use-local-tui-and-terminal-chat/README.md) | Entrypoints, Execution | `tui`, `terminal`, embedded mode | `v2026.4.23/core-modules.md` |
| Serve Gateway And Remote Clients | remote gateway 怎麼接 session 與 agent 執行 | [features/04-serve-gateway-and-remote-clients/README.md](./features/04-serve-gateway-and-remote-clients/README.md) | Gateway And Remote Transport | gateway server, RPC, session patching | `v*/architecture.md` |
| Register Plugins And Extensions | 擴充如何被發現、註冊、掛到 runtime | [features/05-register-plugins-and-extensions/README.md](./features/05-register-plugins-and-extensions/README.md) | Plugin And Extension Runtime | plugin registry, extension factory | `v*/extensions.md` |
| Route Models And Providers | model/provider 路由怎麼決定、套用在哪一層 | [features/06-route-models-and-providers/README.md](./features/06-route-models-and-providers/README.md) | Provider And Model Routing | provider config, model listing, execution routing | `v*/core-modules.md` |
| Approve Tools And Guard Sensitive Actions | approval gate 與安全檢查在哪裡生效 | [features/07-approve-tools-and-guard-sensitive-actions/README.md](./features/07-approve-tools-and-guard-sensitive-actions/README.md) | Tool Approval And Security Guards | tool execution, chat approval, privileged commands | `v*/changelog-notes.md` |
| Run MCP And Tool Bridges | MCP server 與 tool bridge 怎麼暴露能力 | [features/08-run-mcp-and-tool-bridges/README.md](./features/08-run-mcp-and-tool-bridges/README.md) | MCP And Bridge Surface | MCP tools, internal bridge, owners-only tools | `v*/extensions.md` |
| Persist Sessions Memory And State | session、memory、state 怎麼存取與恢復 | [features/09-persist-sessions-memory-and-state/README.md](./features/09-persist-sessions-memory-and-state/README.md) | Session Memory And Persistence | session utils, memory tools, persistence | `v*/core-modules.md` |
| Run Cron And Background Jobs | 背景工作與排程怎麼註冊與觸發 | [features/10-run-cron-and-background-jobs/README.md](./features/10-run-cron-and-background-jobs/README.md) | Cron And Background Work | cron tools, job runtime, async flows | `v*/changelog-notes.md` |

## 擴充規則

- 新主題必須是功能或使用情境，不是抽象名詞
- 只在它對工程修改路徑有清楚入口與控制點時，才新增為獨立主題
- 若只是某主題的子流程，優先擴寫既有 feature 文件，而不是新增新資料夾