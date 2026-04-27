---
version: v2026.1.5-2
date: 2026-04-24
---

# OpenClaw v2026.1.5-2 插件與通道分析

## 插件架構概覽

`v2026.1.5-2` 的實際 diff 沒有碰 `extensions/` 目錄，也沒有看到 provider / channel / plugin contract 的直接修改。這意味著本版不應該被寫成「extension 重構版」；它本質上是 QR helper vendor import 的 follow-up 修補版。

## 主要 extension / provider / channel 列表

| 類別 | 這版是否有直接變更 | 證據 |
|------|--------------------|------|
| `extensions/` 目錄下的通道或 provider | 否 | `git diff --name-only v2026.1.5-1..v2026.1.5-2` 無 `extensions/` 檔案 |
| web login QR 路徑 | 間接受影響 | `src/web/login-qr.ts` -> `src/web/qr-image.ts` |
| macOS bundled relay smoke | 間接受影響 | `src/macos/relay.ts` |

## plugin SDK / contract 證據

- 這版沒有讀到 plugin SDK 或 extension contract 的實際變更。
- 因此若要回答「當時某個 provider/channel 怎麼註冊」，仍需額外針對該版本的 `extensions/` 樹做獨立追蹤。

## 本版變更摘要

| 項目 | 類型 | 影響 |
|------|------|------|
| `QRErrorCorrectLevel.js` 顯式 import | runtime compatibility | 間接影響 web login QR 與 macOS smoke |
| `qrcode-terminal` 型別宣告同步 | TypeScript compatibility | 避免型別與 runtime path 分裂 |

## 已知限制

- 這份文件只能證明本版沒有 direct extension change，不能代表整個 extension 架構已被重新驗證。
- 如果需要新標準的 provider/channel 分析，應另開該版本的 extension feature slice，而不是在 patch 版泛論整棵 extension tree。