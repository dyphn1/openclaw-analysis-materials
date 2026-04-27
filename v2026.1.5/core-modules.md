---
version: v2026.1.5
date: 2026-04-22
#

# OpenClaw v2026.1.5 核心模組深度分析

## 核心目錄結構

在 `src/` 目錄下，核心模組包括：

### 1. 基礎設施 (infra)
- `src/infra/`：包含系統事件、節點配對、殼環境等基礎設施。
- 關鍵檔案：`system-events.ts`, `node-pairing.ts`, `shell-env.ts`, `heartbeat-runner.ts` 等。

### 2. 代理人系統 (agents)
- `src/agents/`：負責工具、模型選擇、技能安裝、工作空間管理等。
- 重要子模組：
  - `tools/`：工具實作。
  - `skills.ts`、`skills-install.ts`：技能系統。
  - `model-selection.ts`、`model-catalog.ts`：AI 模型選擇與備援。
  - `workspace.ts`：工作空間管理。
  - `pi-embedded*`：針對 Raspberry Pi 的嵌入式支援。

### 3. 命令列介面 (cli)
- `src/cli/`：CLI 入口點與指令解析。
- `src/commands/`：具體的 CLI 指令實作。

### 4. 設定與配置 (config)
- `src/config/`：設定檔載入與驗證。

### 5. 通訊通道整合
- `src/discord/`、`src/telegram/`、`src/slack/`、`src/web/` 等：各通訊平台的適配器。

### 6. 其他核心模組
- `src/sessions/`：會話管理。
- `src/cron/`：定時任務排程。
- `src/provider-web.ts`：網頁提供者（用於某些功能）。
- `src/index.ts`：主入口。

## 模組間依賴

- 核心框架依賴 `infra` 提供的低層次功能。
- `agents` 模組提供工具給 `cli` 和 `commands` 使用。
- `config` 被各模組用於讀取設定。
- 各通道模組 (`discord`, `telegram` 等) 實作通用介面以接收和發送訊息。

## 重要設計決策

1. **模組化**：每個功能區域都是獨立的目錄，便於維護和擴展。
2. **工具導向**：透過 `agents/tools` 統一工具介紹，方便擴展新功能。
3. **平台抽象**：通訊平台透過適配器模式實作，核心邏輯與平台解耦。
4. **設定集中**：所有設定透過 `config` 模組載入，提供單一來源的真實。

## 版本特有變更 (v2026.1.5)

根據 `AGENTS.md` 和原始碼檢查，此版本主要更新了：
- `chore: update appcast for 2026.1.5`（見提交訊息）
- 可能包括依賴更新與小錯誤修正。
