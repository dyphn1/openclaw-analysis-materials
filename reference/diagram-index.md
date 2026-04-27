# Diagram Index

這份索引用來追蹤各文件的 Mermaid 圖覆蓋狀態，確保 repo 能像 system-design-primer 一樣先讓讀者用圖理解，再回到內文。

| 文件 | Mermaid 類型 | 狀態 | 備註 |
|------|------|------|------|
| README.md | 無 | 待補 | 入口導覽可考慮加 repo mindmap |
| feature-map.md | 無 | 待補 | 可補功能與子系統對照圖 |
| features/01-run-a-chat-session/README.md | 無 | 待補 | 至少 flowchart + sequenceDiagram |
| features/02-configure-runtime-behavior/README.md | 無 | 待補 | 至少 flowchart |
| features/03-use-local-tui-and-terminal-chat/README.md | 無 | 待補 | 可引用 v2026.4.23 的控制路徑 |
| subsystems/01-entrypoints-and-cli/README.md | 無 | 待補 | 至少 flowchart + mindmap |
| subsystems/03-agent-execution-pipeline/README.md | 無 | 待補 | 至少 flowchart + sequenceDiagram |
| v2026.4.23/architecture.md | Mermaid present | 已有 | 已有 dataflow-like diagrams |
| v2026.4.23/core-modules.md | Mermaid present | 已有 | 已有 call chain diagram |

## 缺圖清單

- 所有新建 feature 文件
- 所有新建 subsystem 文件
- README / feature-map 的導覽圖

## 更新規則

- 新增 Mermaid 圖後，補上圖類型與狀態
- 若圖只畫到部分已驗證邊界，備註需清楚寫出停點