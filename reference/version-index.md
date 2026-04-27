# Version Index

這份索引用來管理版本證據層，而不是取代 feature 主題文件。

| 版本 | upstream revision | 角色 | 狀態 | 入口 |
|------|------|------|------|------|
| v2026.1.5 | 尚待補完 | 早期分析快照 | 既有 | [v2026.1.5/README.md](../v2026.1.5/README.md) |
| v2026.1.5-1 | 尚待補完 | 補強版本 | 既有 | [v2026.1.5-1/README.md](../v2026.1.5-1/README.md) |
| v2026.1.5-2 | 尚待補完 | 補強版本 | 既有 | [v2026.1.5-2/README.md](../v2026.1.5-2/README.md) |
| v2026.4.23 | 尚待補完 | 近期版本快照 | 既有 | [v2026.4.23/README.md](../v2026.4.23/README.md) |

## 用法

- 查一個功能最近怎麼演進：先看 feature 文件，再看這裡對應版本
- 查某版是否引入新控制點：直接進版本目錄看 `changelog-notes.md` 與 `core-modules.md`
- 查版本之間的主要差異與 revision：看 [version-delta-log.md](./version-delta-log.md)

## 後續規則

- 新版本仍沿用 `v<version>/` 根目錄格式
- 每個版本都應記錄對應的 upstream revision、commit、tag、PR 或 release revision；若目前沒有，明寫 `尚待補完`
- 每次新增版本後，要同步更新此索引
- 若穩定學習文件因新版證據需要修正，也要回寫相關 feature 文件