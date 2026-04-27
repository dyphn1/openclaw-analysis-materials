# Subsystem Overview

功能主題回答的是「工程師要完成什麼事」，子系統回答的是「這些事背後共用哪些內部邊界與責任分工」。

## 子系統列表

- [Entrypoints And CLI](./01-entrypoints-and-cli/README.md)
- [Config And Runtime Policy](./02-config-and-runtime-policy/README.md)
- [Agent Execution Pipeline](./03-agent-execution-pipeline/README.md)
- [Gateway And Remote Transport](./04-gateway-and-remote-transport/README.md)
- [Session Memory And Persistence](./05-session-memory-and-persistence/README.md)
- [Provider And Model Routing](./06-provider-and-model-routing/README.md)
- [Plugin And Extension Runtime](./07-plugin-and-extension-runtime/README.md)
- [Tool Approval And Security Guards](./08-tool-approval-and-security-guards/README.md)
- [MCP And Bridge Surface](./09-mcp-and-bridge-surface/README.md)
- [Cron And Background Work](./10-cron-and-background-work/README.md)

## 使用規則

- 先從功能文件理解行為
- 再進 subsystem 文件理解共享邊界
- 若要改跨功能能力，優先補 subsystem 文件再動個別 feature