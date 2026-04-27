---
version: v2026.1.5-1
date: 2026-04-24
---

# OpenClaw v2026.1.5-1 插件與通道分析

## 插件架構概覽

就 `v2026.1.5..v2026.1.5-1` 的實際 diff 而言，這個版本沒有直接修改 `extensions/` 目錄中的 channel、provider 或 plugin contract。這代表本版的真實重點不在 extension 行為，而在兩個核心 runtime 邊界：package surface 與 QR vendor import。

因此，這份文件不假裝重做整棵 extension 樹，而是明確回答兩件事：

1. 本版有沒有 verified extension 變更？
2. 本版 patch 會不會間接影響某些通道或 bundle 路徑？

答案是：沒有看到 verified `extensions/` source change，但 QR helper 的相容性修補會間接影響使用 QR 登入或 bundle smoke 的路徑。

## 主要 extension / provider / channel 列表

| 類別 | 這版是否有直接變更 | 證據 |
|------|--------------------|------|
| `extensions/` 目錄下的 channel / provider | 否 | `git diff --name-only v2026.1.5..v2026.1.5-1` 無 `extensions/` 檔案 |
| web login QR 流程 | 間接受影響 | `src/web/login-qr.ts` -> `src/web/qr-image.ts` |
| macOS bundled relay smoke | 間接受影響 | `src/macos/relay.ts` |

## plugin SDK / contract 證據

- 本次版本差異中沒有讀到 plugin SDK 或 extension contract 被修改。
- 因此不能把 plugin architecture 的泛論寫成這版變更事實。
- 若要分析當時的 extension contract，需額外針對 `extensions/` 與相關 SDK 入口做獨立版本追蹤。

## 本版變更摘要

| 項目 | 類型 | 影響 |
|------|------|------|
| `qrcode-terminal` vendor import 改成顯式檔案路徑 | runtime compatibility | 影響 web login QR 產生與 macOS smoke QR |
| `dist/sessions/**` 納入 npm package | packaging | 不屬於 extension 變更，但會影響 CLI / gateway runtime 是否能正確執行 |

## 已知限制

- 本版沒有驗證 `extensions/` 子樹的全貌，只能確定 patch diff 沒有直接碰它。
- 若使用者需要「哪個 channel/provider 當時怎麼運作」，這份 patch 分析不夠，必須對該版本的 extension 檔案本身另開功能切片分析。
- CHANGELOG 中若有看似與 channel 或 onboarding 相關的敘述，沒有 source diff 佐證時只能標示為待確認。