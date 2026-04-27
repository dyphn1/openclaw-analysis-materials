---
version: v2026.1.5-1
date: 2026-04-24
analyzed_by: openclaw-analysis-agent
---

# OpenClaw v2026.1.5-1 分析報告

## 版本概覽

v2026.1.5-1 不是一個重新整理整體架構的大版本，而是一個很窄的相容性修補版。從 `v2026.1.5..v2026.1.5-1` 的實際 diff 來看，真正被改動的核心只有兩條功能切片：

1. npm 發布包把 `dist/sessions/**` 納入 `package.json` 的 `files`，修補 `clawdbot agent` 與 gateway agent 路徑在 npx 安裝環境下對 `sessions/send-policy` 的 runtime 解析問題。
2. `src/web/qr-image.ts` 將 `qrcode-terminal/vendor/QRCode` 改為明確匯入 `qrcode-terminal/vendor/QRCode/index.js`，修補 Node 25 / ESM 對 vendor 目錄匯入的相容性問題。

CHANGELOG 在這個版本上另外提到「Onboarding: resolve CLI entrypoint when running via npx」，但這個說法沒有出現在 `v2026.1.5..v2026.1.5-1` 的 source diff 中，因此本版本分析將它標成 changelog-only，而不是已驗證實作事實。

## 主要功能清單

| 功能 | 狀態 | 入口模組 |
|------|------|----------|
| sessions 打包修補 | 已深追 | `package.json`、`src/commands/agent.ts`、`src/gateway/server-methods/agent.ts`、`src/sessions/send-policy.ts` |
| Web / macOS QR 相容性修補 | 已深追 | `src/web/login-qr.ts`、`src/web/qr-image.ts`、`src/macos/relay.ts` |
| Onboarding CLI entrypoint 說明 | 僅 changelog | `CHANGELOG.md` |

## 快速安全性摘要

- 沒有看到明確標成 security fix 的變更。
- 這版主要是 runtime compatibility hardening：降低 npx 安裝與 Node 25 啟動時的模組解析失敗風險。

## 完整性狀態

- 已深追：`dist/sessions` 打包缺件問題、QR vendor import 控制路徑、對應測試與 caller。
- 尚待補完：整體系統架構、完整 extension 樹、changelog 中 onboarding 敘述的實作證據。
- 本版文件刻意採 patch-slice 分析，不假裝重做整個 v2026.1.5 系統總覽。

## 文件索引

- [架構分析](./architecture.md)
- [核心模組](./core-modules.md)
- [插件/通道](./extensions.md)
- [版本變更重點](./changelog-notes.md)