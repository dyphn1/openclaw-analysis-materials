# Version Delta Log

這份帳本集中記錄每個版本的主要差異，讓讀者可以快速回答：哪一版改了哪些功能、波及哪些子系統、對應 revision 是什麼。

## 版本差異總表

| 版本 | upstream revision | 主要差異 | 受影響功能 | 受影響子系統 | 證據入口 | 狀態 |
|------|------|------|------|------|------|------|
| v2026.1.5 | 尚待補完 | 尚待補完 | 尚待補完 | 尚待補完 | [v2026.1.5/README.md](../v2026.1.5/README.md) | 待補 |
| v2026.1.5-1 | 尚待補完 | 尚待補完 | 尚待補完 | 尚待補完 | [v2026.1.5-1/README.md](../v2026.1.5-1/README.md) | 待補 |
| v2026.1.5-2 | 尚待補完 | 尚待補完 | 尚待補完 | 尚待補完 | [v2026.1.5-2/README.md](../v2026.1.5-2/README.md) | 待補 |
| v2026.4.23 | 尚待補完 | plugin resolution fix, local TUI related evidence, security hardening | Local TUI, Plugins, MCP, Approvals | Entrypoints, Plugin Runtime, MCP, Security | [v2026.4.23/README.md](../v2026.4.23/README.md) | 部分 |

## revision 記錄規則

- 優先填 commit hash、tag 或 release revision
- 若 changelog 只提供 PR 或 issue 編號，可先記 PR / issue
- 若目前查不到，明寫 `尚待補完`

## 更新規則

- 每次新增或補強版本目錄後，都要回寫這份帳本
- 若同一版又補出新的受影響功能或子系統，直接補欄位，不新增重複列