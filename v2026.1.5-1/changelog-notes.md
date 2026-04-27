---
version: v2026.1.5-1
date: 2026-04-24
---

# OpenClaw v2026.1.5-1 變更重點分析

## 新功能

本版沒有看到可由 source diff 驗證的新功能；它是相容性修補版。

## Fixes

| 項目 | 實作位置 | 說明 | 證據來源類型 | 信心 |
|------|----------|------|--------------|------|
| npm 發佈包補上 `dist/sessions/**` | `package.json` | 修補 npx / npm 安裝後已編譯 code 需要 `sessions/send-policy` 卻找不到 dist subtree 的問題 | 原始碼 diff | 高 |
| Node 25 QR vendor import 改成 `QRCode/index.js` | `src/web/qr-image.ts`、`src/types/qrcode-terminal.d.ts` | 避免 `qrcode-terminal/vendor/QRCode` 目錄匯入在較嚴格 ESM runtime 下失敗 | 原始碼 diff + 測試 | 高 |
| QR helper 測試鎖定顯式 vendor import | `src/web/qr-image.test.ts` | 測試明確要求 source 不可用 dynamic require，且必須包含 `index.js` import | 測試 | 高 |
| Onboarding: resolve CLI entrypoint when running via npx | `CHANGELOG.md` | CHANGELOG 有記錄，但 `v2026.1.5..v2026.1.5-1` 的 source diff 沒看到對應修補 | changelog only | 低 |

## Breaking Changes

沒有看到明確的 breaking change。

## 安全性修正

沒有看到被明確標記為 security fix 的變更。

## 與前版比較

相對於 `v2026.1.5`，`v2026.1.5-1` 的已驗證變化非常集中：

- package 發佈面新增 `dist/sessions/**`
- QR vendor import 從目錄路徑收斂為明確 `index.js`

也就是說，這版的演進目的不是增加新能力，而是讓既有 agent / gateway send policy 與 QR helper 在發佈後、Node 25 下仍能穩定運作。