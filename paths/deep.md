# 深路徑

目標：能規劃跨模組改動，或新增擴充/安全控制點。

## 建議順序

1. [Serve Gateway And Remote Clients](../features/04-serve-gateway-and-remote-clients/README.md)
2. [Register Plugins And Extensions](../features/05-register-plugins-and-extensions/README.md)
3. [Approve Tools And Guard Sensitive Actions](../features/07-approve-tools-and-guard-sensitive-actions/README.md)
4. [Run MCP And Tool Bridges](../features/08-run-mcp-and-tool-bridges/README.md)
5. [Persist Sessions Memory And State](../features/09-persist-sessions-memory-and-state/README.md)
6. [Run Cron And Background Jobs](../features/10-run-cron-and-background-jobs/README.md)
7. 逐版回查 [Version Index](../reference/version-index.md) 中涉及的 changelog 與 evidence docs

## 讀完後應能回答

- 權限與 approval gate 在哪裡被 enforce
- MCP/tool bridge 哪些是邊界，哪些是高風險區
- 狀態持久化和 session 恢復會在哪裡出現回歸
- 做大改動時要先看哪些測試與相容性行為