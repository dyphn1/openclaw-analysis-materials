---
version: v2026.1.5-2
date: 2026-04-24
---

# OpenClaw v2026.1.5-2 變更重點分析

## 新功能

本版沒有看到可由 source diff 驗證的新功能；它是 `v2026.1.5-1` 的補強 patch。

## Fixes

| 項目 | 實作位置 | 說明 | 證據來源類型 | 信心 |
|------|----------|------|--------------|------|
| `QRErrorCorrectLevel` 改成顯式 `.js` import | `src/web/qr-image.ts` | 完成 QR vendor import 的 Node ESM 相容性修補 | 原始碼 diff | 高 |
| `declare module` 同步改成 `.js` | `src/types/qrcode-terminal.d.ts` | 保持型別系統與 runtime import path 一致 | 原始碼 diff | 高 |
| 測試新增 `.js` import 斷言 | `src/web/qr-image.test.ts` | 防止後續回歸到沒有副檔名的 vendor import | 測試 | 高 |

## Breaking Changes

沒有看到明確 breaking change。若外部程式碼直接依賴舊的 declaration path，理論上可能需要同步調整，但這份版本 diff 沒有證據顯示官方把它視為 public breaking surface。

## 安全性修正

沒有看到被明確標記為 security fix 的變更。

## 與前版比較

相對於 `v2026.1.5-1`，這版做的事情非常單純：把上一版尚未處理完的 QR vendor import 收尾。

- `v2026.1.5-1`：`QRCode` 改成 `index.js`
- `v2026.1.5-2`：`QRErrorCorrectLevel` 也改成 `.js`，並同步測試與 d.ts

因此，v2026.1.5-2 的演進目的可以描述成「把 Node 25 / ESM 相容性修補做完整」，而不是新增新的 QR 功能。