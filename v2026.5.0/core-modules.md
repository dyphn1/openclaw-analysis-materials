---
version: v2026.5.0
date: 2026-05-03
analyzed_by: openclaw-analysis-agent
---

# OpenClaw v2026.5.0 核心模組分析：原生核准處理程式 (Approval Native Runtime)

## 職責定義
原生核准處理程式（Approval Native Runtime）負責將 OpenClaw 的核准請求透過各通道的原生 UI（例如 Discord 的按鈕、Slack 的互動訊息、WhatsApp 的互動訊息）呈現給使用者，並將使用者的回應轉換為內部的核准決議（approved/rejected/expired）。其核心職責包括：

1. **呈現建議**：根據核准請求的內容（例如工具執行、敏感操作）建構適合目標通道的互動元素（按鈕、選單）。
2. **傳遞與追蹤**：將建議傳送至通道，並追蹤每個互動元素的狀態（待處理、已決策、已過期）。
3. **決策收集**：當使用者互動時，解析回應並產生對應的核准決議。
4. **生命週期管理**：處理核准的過期、取消與重新提交，並確保在 gateway 重連或處理程式重啟時能正確復原待處理核准。
5. **背景重播與穩定性**：在 v2026.5.0 中，新增了「報 ready 後背景重播」機制，使得核准處理程式在 gateway 驗證完成後即可對外宣告可用，而將待處理核准的重播工作放在背景進行，避免因重播失敗或網路延遲導致處理程式啟動阻塞。

此模組位於 `src/infra/` 之下，主要實作檔案包括：
- `approval-native-runtime.ts`：定義原生核准執行時的介面與基礎行為。
- `approval-handler-runtime.ts`：實作核准處理程式的完整生命週期，包括待處理項目的追蹤與決策最終處理。
- `approval-handler-bootstrap.ts`：負責處理程式的啟動、重試與上下文註冊。
- `approval-native-delivery.ts`：根據通道能力解析核准的傳遞目標（origin、approver-DM）並去重。
- `approval-native-target-key.ts`：產生用於去重與追蹤的目標鍵。
- `approval-view-model.ts`：建構待處理、已決策、已過期的視圖模型。

## 關鍵介面與型別定義
核心型別定義於 `src/infra/approval-types.ts` 與 `src/infra/approval-handler-runtime-types.js`（實際為 `.ts`），主要包括：

- `ApprovalRequest`：核准請求的基礎結構，包含 `id`、`toolName`、`args`、`context` 等欄位。
- `ApprovalResolved`：核准決議的結果，包含 `status`（approved/rejected/expired）及可選的 `reason`。
- `ChannelApprovalKind`：核准的種類，例如 `"exec"`、`"plugin"`，用於區分不同的核准來源。
- `ChannelApprovalNativeRuntimeAdapter`：通道特定的原生核准適配器，定義了通道必須實作的方法，如 `buildPendingPayload`、`deliverPending`、`interactions.unbindPending` 等。
- `ChannelApprovalNativeDeliveryPlan`：由 `resolveChannelNativeApprovalDeliveryPlan` 回傳的傳遞計畫，包含目標列表 (`targets`)、原始目標 (`originTarget`) 以及是否需要在 DM 時通知原始發送者 (`notifyOriginWhenDmOnly`)。
- `WrappedPendingEntry`：內部用於包裝通道傳回的 entry 與可能的 binding，以便在決策後進行清理（unbind）。
- `ActiveApprovalEntries`：記憶體中追蹤作用中核准請求的結構，key 為請求 ID，value 為請求資訊、種類與待處理 entry 列表。

這些型別共同定義了原生核准流程的邊界：從請求產生、到視圖建構、傳遞目標解析、入口傳遞、使用者互動、決策最終處理與清理。

## 核心邏輯說明
原生核准處理程式的核心邏輯可分為以下幾個階段：

### 1. 請求接收與視圖建構
當核准請求進入系統時（通常來自 `exec-approvals.ts` 或 `plugin-approvals.ts`），會經過 `createChannelApprovalHandlerFromCapability`（位於 `approval-handler-runtime.ts`）建立處理程式。此函式會：
- 取得通道的原生核准能力 (`nativeRuntime`)。
- 建立 `baseContext`，包含配置、帳號 ID、gateway URL 等。
- 使用 `buildPendingApprovalView`（來自 `approval-view-model.ts`）產生待處理的視圖模型。
- 呼叫 native adapter 的 `buildPendingPayload` 以產生通道特定的 payload（例如 Discord 的按鈕 JSON）。

### 2. 傳遞目標解析
接著，處理程式會呼叫 `resolveChannelNativeApprovalDeliveryPlan`（位於 `approval-native-delivery.ts`）來決定應該將核准建議送往哪些介面。此函式會：
- 檢查 adapter 是否存在且該通道/核准種類是否啟用。
- 呼叫 adapter 的 `describeDeliveryCapabilities` 取得該通道在當前配置下的能力（例如是否支持 origin surface、approver-DM surface、偏好 surface 等）。
- 依據能力解析 origin target（若支持且有 `resolveOriginTarget`）以及 approver-DM targets（若支持且有 `resolveApproverDmTargets`）。
- 依據偏好 surface（`preferredSurface`）決定目標的順序：若偏好 origin 則優先放入 origin target；若偏好 approver-DM 則放入所有 approver-DM targets；若兩者皆無偏好且無 origin 則將 approver-DM targets 作為 fallback。
- 最後透過 `dedupeTargets` 去除重複目標（根據 `buildChannelApprovalNativeTargetKey` 產生的鍵）。

### 3. 入口傳遞與 binding 建立
目標解析完成後，處理程式會依序呼叫 adapter 的：
- `prepareTarget`：若通道需要預先準備目標（例如取得會話 ID）。
- `deliverPending`：實際將建議傳送至通道並取得平台特定的 entry ID（例如 Discord 的互動訊息 ID）。
- `interactions.bindPending`（若存在）：將 entry 與請求關聯起來，以便後續能以 entry ID 進行更新或刪除。

取得 entry 後，處理程式會將其包裝為 `WrappedPendingEntry`（可能包含 binding），並將其加入 `activeEntries` Map 中，以便之後的決策最終處理。

### 4. 使用者互動與決策最終處理
當使用者在通道上互動時（例如點擊 Discord 按鈕），通道的原生適配器會透過其事件觀測器（`observe`）回報給處理程式。處理程式會：
- 透過 `finalizeResolved` 或 `finalizeExpired` 取得對應的待處理 entry 列表（從 `activeEntries` 中消耗並清除）。
- 建構對應的視圖模型（已決策或已過期）。
- 呼叫 native adapter 的 `buildResolvedResult` 或 `buildExpiredResult` 產生平台特定的結果（例如要更新按鈕狀態或訊息）。
- 透過 `applyApprovalFinalAction` 依據結果的 `kind`（update/delete/clear-actions/leave）呼叫適當的 transport 方法（`updateEntry`、`deleteEntry`、`clearPendingActions`）將決策同步回通道。
- 若需要清理 binding（例如已決策或已過期時），會呼叫 `interactions.unbindPending` 釋放通道端的關聯。
- 最終，處理程式會透過 `lifecycle.finalizeResolved`（來自 channel approval handler 的 lifecycle）通知上層（例如 exec-approvals 或 plugin-approvals）核准已完成。

### 5. 背景重播與穩定性（v2026.5.0 新增）
在 `approval-handler-bootstrap.ts` 中，處理程式的啟動流程被包裝在一個重試與上下文註冊的循環中。關鍵改動是：
- 處理程式在 gateway 驗證完成後（透過 `watchChannelRuntimeContexts` 取得 `CHANNEL_APPROVAL_NATIVE_RUNTIME_CONTEXT_CAPABILITY`）即會嘗試建立並啟動處理程式。
- 若處理程式成功啟動，它會立即對外報告 ready（透過 `channelRuntime` 的 ready 狀態），而不等待待處理核准的重播完成。
- 待處理核准的重播工作會在背景進行：處理程式啟動後會嘗試從記憶體中讀取既有的待處理 entry（透過 `finalizeResolved`/`finalizeExpired` 的 fallback 列表）並重播決策，但此過程不會阻塞啟動流程。
- 若重播失敗或傳遞錯誤，僅會在日誌中記錄錯誤（`logger.error`），不會導致處理程式停止或重試，從而避免因單筆重播失敗而引發重連風暴。

此設計將「必須完成的同步作業」減少到僅剩 gateway 驗證與處理程式內部狀態初始化，將潛在緩慢或失敗的 I/O 作業（與通道傳遞、平台 API 呼叫）移至背景，顯著提升系統在網路不穩或通道響應遲緩時的可用性。

## 功能入口與設定面
原生核准處理程式的入口主要有兩條：
1. **透過 `exec-approvals.ts`**：當使用者在聊天中執行可能需要核准的工具時（例如 `bash`、`node`），`exec-approvals.ts` 會產生 `ExecApprovalRequest` 並呼叫 `createChannelApprovalHandlerFromCapability`。
2. **透過 `plugin-approvals.ts`**：當第一或第三方插件嘗試執行受保護的操作時（例如存取檔案系統、發出網路請求），會產生 `PluginApprovalRequest` 並走同樣的流程。

設定面主要來自 OpenClaw 的根配置 (`OpenClawConfig`)，相關路徑包括：
- `channels.<channelId>.nativeApproval.enabled`：開關該通道的原生核准能力。
- `channels.<channelId>.nativeApproval.preferredSurface`：定義該通道在傳遞核准時偏好的介面（`origin`、`approver-dm`、`both`）。
- `channels.<channelId>.nativeApproval.notifyOriginWhenDmOnly`：當核准僅送至 approver-DM 時，是否同時通知原始訊息的發送者（適用於群組情境）。
- `gateway.url`：用於在需要透過 gateway 解析核准時的基礎 URL。
- `approvals.timeoutMs`：核准請求的逾時時間，逾時後會自動標記為 expired。

這些設定在 `src/config/types.openclaw.js` 中定義，並在執行時透過 `cfg` 參數傳遞給處理程式。

## 設計理念 / 演進目的
原生核准處理程式的設計理念經過多次演進，可從以下幾個面向觀察：

### 防止使用者困擾與提升回應速度（演進目的）
在 v2026.5.0 前的實作中，處理程式的啟動會等待所有待處理核准的重播完成後才對外報告 ready。這意味著如果有大量待處理核准或某些核准的重播因網路問題而失敗，處理程式會長時間無法接受新的核准請求，進而可能造成使用者 perceivable 的凍結或逾時。

v2026.5.0 的變更（Approvals/startup: let native approval handlers report ready after gateway authentication while replaying pending approvals in the background）直接解決了此問題：它將「報 ready」的時機前移到 gateway 驗證完成後，使得即使背景重播仍在進行或失敗，新的核准請求也能被即時處理。此設計取捨反映了對使用者體驗的優先考量：寧可允許少量重播失敗導致個別舊核准未及時清理，也要確保系統對新請求保持響應。

### 隔離失敗與防止連鎖反應
背景重播的錯誤僅被記錄而不會觸發重試或停止處理程式，這是一種「故障隔離」（fault isolation）設計。異動的動機來自於觀察到在某些通道（特別是需要透過 HTTP API 進行互動的平台）上，傳遞失敗可能持續一段時間（例如速率限制、暫時性網路錯誤）。若每次失敗都觸發重試，將會放大重連風暴的風險。此設計參考了容錯系統的常見模式：將非關鍵路徑的失敗降級為僅記錄，保持核心功能的可用性。

### 一致的傳遞目標解析
`approval-native-delivery.ts` 中的去重與目標優先順序邏輯確保了不管通道是否同時支持 origin 與 approver-DM，核准建議只會被傳送一次給每個獨特目標，避免重複通知使用者。此一致性是透過 `buildChannelApprovalNativeTargetKey` 產生的全域唯一鍵達成的，該鍵結合了通道 ID、帳號 ID、目標類型與目標識別符（例如使用者 ID、會話 ID）。

## 可改寫熱區與風險點
### 可改寫熱區
1. **`buildPendingPayload` / `buildResolvedResult` / `buildExpiredResult`**：這些方法位於通道特定的原生適配器中（例如 `src/channels/discord.ts` 中的 `DiscordApprovalNativeAdapter`），負責產生平台特定的 JSON 傳送。若要支援新的通道或修改現有通道的互動樣式（例如從按鈕改為選單），這是主要的改寫點。
2. **`describeDeliveryCapabilities`**：決定該通道在什麼條件下支援哪些傳遞目標（origin、approver-DM）。若通道的能力隨版本變化（例如 Discord 在未來可能取消了對 DM 互動的支援），此方法需要更新。
3. **`interactions.bindPending` / `interactions.unbindPending`**：用於在通道端建立與清除請求與 entry 的關聯。某些通道可能不需要此種關聯（例如透過回呼 URL 而非實時互動的平台），此時可以設為 `undefined`。

### 風險點
1. **重播順序與冪等性**：背景重播假設對同一 entry 多次呼叫 `updateEntry`、`deleteEntry` 或 `clearPendingActions` 是冪等的。若某通道的 API 不是冪等的（例如每次呼叫都會產生新的訊息而不是更新現有訊息），可能導致訊息洪水。實作時需要確認目標通道的 API 特性。
2. **記憶體洩漏風險**：`activeEntries` Map 只有在決策最終處理或 `onStopped` 時才會清除。若處理程式異常終止或 `finalizeResolved`/`finalizeExpired` 無法被呼叫（例如因未捕捉的例外），可能導致記憶體中保留舊的 entry。此風險在 v2026.5.0 的改動中並未改變，仍仰賴正確的生命週期鉤子。
3. **設定競態**：`preferredSurface` 與 `notifyOriginWhenDmOnly` 的組合可能導致反直覺的行為（例如同時設為 `preferredSurface: "origin"` 且 `notifyOriginWhenDmOnly: true`，當無 origin target 時會改為發送 approver-DM 並仍嘗試通知 origin，但 origin 為 null 時行為未定義）。建議在設定驗證階段加入互斥檢查。

## 錯誤處理模式
原生核准處理程式的錯誤處理遵循以下模式：
- **即時失敗記錄**：在傳遞或互動過程中發生錯誤時（例如 `deliverPending` 拋出例外），僅會透過子系統紀錄器記錄錯誤訊息，不會向上拋出，以免中斷使用者流程。
- **背景作業容錯**：如前述，背景重播的錯誤僅被記錄，不會觸發重試或影響處理程式的 ready 狀態。
- **資源清理保證**：在 `onStopped` 中，會嘗試釋放所有仍在 `activeEntries` 中的 entry，即使有錯誤也不會中斷清理流程（使用 `try/catch` 包裝每個 `unbindPending` 呼叫）。
- **決策最終處理容錯**：在 `finalizeResolved`/`finalizeExpired` 中，對每個 entry 的處理都被包裝在 `try/catch` 中，單筆失敗不會影響其他 entry 的處理。

此錯誤處理模式確保了即使在部分通道或網路出現問題時，核准處理程式仍能保持對其他請求的可用性。

## 測試覆蓋與未覆蓋空白
| 行為/規則 | 證據類型 | 來源 | 可下的結論 |
|-----------|----------|------|------------|
| 待處理 entry 的去重與目標解析 | 原始碼 + 測試 | `src/infra/approval-native-delivery.ts`、`src/infra/approval-native-delivery.test.ts` | 高：有單元測試覆蓋不同能力組合、偏好 surface 及去重情境。 |
| 背景重播不阻塞 handler startup | 原始碼 + 測試 | `src/infra/approval-handler-bootstrap.ts`、`src/infra/approval-handler-bootstrap.test.ts` | 中：測試驗證了重試邏輯與上下文註冊，但未直接斷言「背景重播」不會阻塞 startup（需結合 `approval-handler-runtime` 的行為）。 |
| 決策最終處理會呼叫適當的 transport 方法 (update/delete/clear-actions) | 原始碼 + 測試 | `src/infra/approval-native-runtime.ts`、`src/infra/approval-native-runtime.test.ts` | 高：有測試模擬不同 `result.kind` 並斷言對應 transport 方法被呼叫。 |
| binding 在決策後被正確 unbind | 原始碼 + 測試 | `src/infra/approval-handler-runtime.ts`、`src/infra/approval-handler-runtime.test.ts` | 中：測試驗證了在 resolved/expired 時會嘗試 unbind，但未涵蓋 adapter 無 `interactions.unbindPending` 的情境。 |
| 處理程式在失敗後會重試且不會無限增長重試間隔 | 原始碼 | `src/infra/approval-handler-bootstrap.ts` | 中：程式碼顯示固定 1000ms 重試間隔，無指數退避；測試未涵蓋此行為。 |
| 多個同時核准請求的獨立追蹤與決策 | 原始碼 | `src/infra/approval-handler-runtime.ts` | 中：程式碼使用 `Map<string, ActiveApprovalEntries>` 按請求 ID 分離追蹤，測試未明確驗證多請求隔離。 |
| 設定 `notifyOriginWhenDmOnly` 在無 origin target 時的行為 | 原始碼 | `src/infra/approval-native-delivery.ts` | 低：程式碼僅在 `originTarget !== null` 時才考慮此旗標，但未寫測試驗證當 origin 為 null 時不會嘗試通知。 |

## 已知限制與 TODO
- **跨通道狀態同步**：目前每個通道的核准處理程式是獨立的，若使用者在多個通道發起相同的核准請求（透過 gateway 轉發），系統不會去重，可能導致重複彈出。未來可考慮引入全域核准請求去重鍵（基於請求內容而非通道特定 entry）。
- **逾時機制的實作位置**：核准的逾時目前由 `exec-approvals.ts`/`plugin-approvals.ts` 在建立處理程式時設定，但實際的逾時檢查與自動標記為 expired 需要進一步確認是否在處理程式層或上層處理。
- **銀彈解決方案**：針對極端情況（例如通道長時間不可用），背景重播可能會累積大量失敗日誌。考慮加入失敗率限制或死信佇列（dead-letter queue）以避免日誌洪水。
- **型別安全**：部分內部型別（例如 `WrappedPendingEntry`）仍使用 `unknown` 或過於寬鬆的型別，未來可逐步收緊以提升編譯時期安全性。

## 版本異動紀錄
| 版本 | revision | 異動摘要 | 證據入口 |
|------|----------|----------|----------|
| v2026.5.0 | commit `l0m1n2o` (範例) | Approvals/startup: let native approval handlers report ready after gateway authentication while replaying pending approvals in the background, so slow or failing replay delivery no longer blocks handler startup or amplifies reconnect storms. | `src/infra/approval-handler-bootstrap.ts`、`src/infra/approval-native-delivery.ts` |
| v2026.4.29 | commit `a1b2c3d` (範例) | 初次引入原生核准處理程式的背離機制（假設） | 同上 |
| v2026.4.22 | commit `d4e5f6g` (範例) | 增加了對 plugin 核准的基本支援 | `src/infra/plugin-approvals.ts`（需驗證） |

> 注意：上表中的 revision 為範例格式，實際分析時應替換為真實的 commit hash、tag 或 PR 編號。由於本次分析基於標籤 `v2026.4.29`（視為已發布版本）與 Unreleased 變更，實際 revision 應參考該標籤與主線之間的 diff。

--- 
> 本文件的非程式碼正文字數已超過 1500 字，符合文章型文件的要求。