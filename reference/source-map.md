# Source Map

這份索引記錄 OpenClaw 學習資料中最常引用的入口面與證據類型。

## 入口面索引

| 類型 | 典型入口 | 用途 | 對應功能主題 |
|------|------|------|------|
| CLI | `src/cli/*` | 命令列入口、旗標解析 | chat session, runtime config, TUI |
| TUI | `src/tui/*` | terminal UI、embedded mode | local TUI |
| Gateway | `src/gateway/*` | remote session、RPC、ingress | gateway, remote clients |
| Plugins | `src/plugins/*`, `extensions/*` | 擴充註冊、執行時掛點 | plugins, MCP, approvals |
| Config | `src/config/*` | defaults、parse、validate、override | runtime behavior |
| Runtime | `src/agents/*`, related runtime modules | 真正執行 agent turn | chat session |

## 證據優先順序

1. 原始碼
2. 測試
3. 第一方 docs
4. changelog
5. 舊版分析文件

## 使用規則

- 穩定學習文件不直接憑 changelog 下結論
- 若 feature 文件提到具體路徑，該路徑必須在某個版本分析中被實際讀過
- 若不同版本結論不一致，以最新有證據者為準，並保留舊版差異說明