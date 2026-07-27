# M365 公務車借用自動通知與後台管理流程 Master 專案紀錄

更新日期：2026-07-24
Master 狀態：本文件為目前唯一最新版本，後續功能分支應以本文件為基準。

## 一、目前專案總覽

| 項目 | 內容 |
|---|---|
| 專案名稱 | M365 公務車借用自動通知與後台管理流程 |
| 目前版本 | `v0.2.14` |
| 專案目標 | 讓員工透過 Outlook 預約公務車，系統自動同步至 SharePoint 後台，並在借用前透過 Teams Adaptive Card 通知借用人填寫共乘人數與確認公務車使用規範，供承辦人判斷是否發放鑰匙 |
| 主要架構 | Outlook 資源行事曆 + Power Automate（Office 365 Outlook「傳送 HTTP 要求」直連 Graph）+ Teams Adaptive Card + SharePoint List |
| 授權限制 | O365 E1 可行，不使用 Power Automate Premium 連接器 |
| 目前開發階段 | M6 正式行事曆同步流程已完成 P3 情境測試：時間異動、整天借用、重複觸發皆通過；取消預約發現新問題（事件刪除無法反映至 SharePoint），待補強 |
| 專案完成度 | 90% |
| 下一個預計完成階段 | 設計並實作事件取消偵測補強邏輯，通過後將 `是否測試資料` 切換為正式運作邏輯，並進入 Teams 通知與回覆流程開發 |

## 二、本次新增完成內容

### SharePoint

- 已在 `ALP_TW_AD` 建立 SharePoint List：`公務車借用管理`。
- 已建立並驗證主要欄位。
- 已新增 `是否整天` 欄位，用於支援 Outlook 整天預約。
- 已新增 `預計通知時間` 欄位，用於支援借用前通知規則。
- 已在正式清單新增並驗證系統防呆欄位，包含事件唯一識別、UTC 原始時間、事件同步狀態、卡片版本、通知狀態、Flow 執行鎖、重複資料檢查、取消與失效旗標。
- 已確認清單網址為 `https://alpglobal.sharepoint.com/sites/ALP_TW_AD/Lists/List6/AllItems.aspx`。

### Power Automate

- 已建立功能測試流程：`公務車功能測試-SharePoint清單連線`。
- 已確認 SharePoint 標準連接器可讀取 `公務車借用管理` 清單。
- 已確認 Office 365 Outlook 標準連接器基本連線可用。
- 已將正式流程拆分為兩條建議流程：行事曆同步、Teams 通知與回覆。
- 已建立並執行 `公務車功能測試-資源信箱讀取`（Flow ID：`7aff726f-0f9d-4b7e-a128-baeb368ab1ce`）。
- 已確認 Calendar ID 選單目前只有個人 `Calendar`；直接填入 Altis 資源信箱會回傳「ID 格式不正確」。
- 測試流程已關閉，保留執行紀錄供追蹤。

### Teams Adaptive Card

- 已完成卡片內容設計。
- 已納入固定版「公司公務車使用管理規範」文字。
- 已設計借用人必填欄位：共乘人數、規範確認。
- 已建立 Adaptive Card JSON 範例。

### Outlook

- 已確認三台公務車資源信箱資訊。
- 已確認 Outlook 標準連接器基本可用。
- 2026-07-02 已實測資源行事曆讀取；目前三台車未出現在 Power Automate Calendar ID 選單。
- 資源信箱 Email 不是 `取得事件的行事曆檢視 (V3)` 可接受的 Calendar ID。
- 已確認 `ad.general@alp.global` 具有三台資源信箱 Calendar Reviewer 權限，可在 Outlook 查看行事曆；但 V2 實測尚未列出三台共用行事曆。
- 本專案不修改 Resource Mailbox、Calendar Processing 或 Exchange 組態，因此不需要 Exchange Administrator。
- 已於 2026-07-02 15:34 成功執行 `取得行事曆 (V2)`，輸出只有個人 `Calendar`。
- 已於 2026-07-04 確認 Outlook 網頁可顯示三台公務車共用行事曆，但重新執行 `取得行事曆 (V2)` 仍只回傳一個個人 Calendar。
- 下一步需評估 Editor 權限或其他不使用 Premium 連接器的標準讀取方案。

### Excel

- 目前不使用 Excel 作為正式資料來源。
- 後台管理以 SharePoint List 為唯一主檔，避免多份資料造成維護成本。

### 通知流程

- 已新增通知時間規則：
  - 一般借用：借用起始時間前 1 小時通知。
  - 最早通知時間：當天早上 08:00。
  - 整天借用：當天早上 08:00。
- 已新增 `預計通知時間` 欄位，供 Power Automate 計算後寫回。

### 文件

- 已更新 `README.md`。
- 已更新 `CHANGELOG.md`。
- 已更新 SharePoint 欄位文件。
- 已更新 Power Automate 流程文件。
- 已更新 Teams Adaptive Card 文件。
- 已更新功能測試計畫。
- 已新增 Master 專案紀錄、里程碑、完成清單、待辦事項。

### 測試

- SharePoint List 欄位驗證完成。
- Power Automate SharePoint 連線測試完成。
- Power Automate Outlook 標準連接器基本測試完成。
- 通知時間支援欄位驗證完成。
- 資源行事曆 Calendar ID 可見性測試未通過，已確認為 M5 正式阻擋點。
- 資源信箱讀取測試流程已關閉，避免持續排程失敗。
- 已修正先前將 Calendar ID 錯誤推論為需要 Exchange Administrator 的判斷；既有 Reviewer 權限已足夠。
- 已確認 Codex 內建瀏覽器與 Chrome 均可作為 Power Automate 操作後端；後續應沿用目前已登入且已連線的工作階段，不得因舊版瀏覽器紀錄推定目前不可操作。實際失敗時須記錄當次工具授權或連線錯誤；詳見 `docs/calendar-access-test.md` 與 `docs/Project_Workflow.md`。

### 系統防呆設計

- 已將五項防呆機制列為正式上線前必須完成：
  - 統一時區為 `Asia/Taipei`。
  - 以 Event ID / iCalUId 作為唯一識別。
  - Outlook 取消與異動需同步 SharePoint。
  - 舊 Teams Adaptive Card 回覆前需重新檢查並可失效。
  - Flow 啟用執行鎖與重複資料防止。
- 已新增防呆設計文件：`docs/system-safeguards.md`。
- 已補強 SharePoint、Power Automate、Adaptive Card、測試案例文件。
- 已將防呆欄位實際建立於 SharePoint 正式清單，後續正式流程可直接寫入與更新。

## 三、目前各功能狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---:|---|
| 需求分析 | 已完成 | 100% | 核心需求、限制與例外情境已整理 |
| SharePoint List | 已完成 | 100% | 已在 `ALP_TW_AD` 建立並驗證 |
| Power Automate | 進行中 | 85% | 時區位移錯誤已修正並驗證；P3 情境測試（時間異動、整天借用、重複觸發）已通過，取消預約發現新問題待補強 |
| Adaptive Card | 設計完成 | 80% | JSON 範例完成，尚未正式串接發送 |
| Teams 通知 | 設計完成，待串接 | 30% | 通知時間規則已確認 |
| Outlook | 測試中 | 70% | ATA-9627 已驗證可讀取真實事件；Camry、Cross 待重複驗證 |
| 系統防呆設計 | 欄位已落地，流程待實作 | 82% | 五項防呆機制已文件化，SharePoint 欄位已建立 |
| 測試案例 | 進行中 | 65% | 已記錄 HTTP 直連 Graph 成功案例，防呆情境待實測 |
| 維護文件 | 已更新 | 95% | Master、治理、防呆、欄位與測試紀錄已更新 |
| 操作手冊 | 待建立 | 20% | 尚需整理行政人員日常操作手冊 |
| Git 版本 | 已建立 | 82% | 目前在 `feature/calendar-access-test` 分支，依 Project Governance 提交並同步 GitHub |

## 四、本次異動摘要

### 新增功能

- 新增 SharePoint 欄位 `是否整天`。
- 新增 SharePoint 欄位 `預計通知時間`。
- 新增通知時間規則：最早 08:00，整天借用 08:00。
- 新增 Master 專案紀錄文件。
- 新增里程碑、完成清單、待辦事項文件。
- 新增系統防呆設計文件與五項正式上線前必做防呆機制。
- 新增 SharePoint 正式清單防呆欄位：`iCalUId`、`唯一識別來源`、`事件最後修改時間`、`原始開始時間 UTC`、`原始結束時間 UTC`、`顯示時區`、`事件同步狀態`、`取消時間`、`卡片版本`、`卡片狀態`、`最新通知批次 ID`、`通知狀態`、`Flow 執行 ID`、`處理鎖定狀態`、`處理鎖定時間`、`重複資料檢查結果`、`防呆檢查結果`、`已取消`、`已失效`。

### 修改功能

- 專案 SharePoint 建立位置統一改為 `ALP_TW_AD`。
- 通知邏輯由單純「借用前 1 小時」改為「借用前 1 小時，但最早 08:00」。
- 整天借用通知時間改為固定當天 08:00。
- SharePoint 後台新增欄位以支援正式通知判斷。

### 修正 Bug

- 修正文檔亂碼問題。
- 修正早上過早借用時，通知時間可能落在上班前的流程設計問題。
- 修正整天借用沒有明確通知時間的規則缺口。

### 新增文件

- `docs/project-master-record.md`
- `docs/project-milestones.md`
- `docs/completion-checklist.md`
- `docs/todo.md`
- `release-notes/v0.2.0.md`
- `docs/Exchange_Resource_Mailbox_Permission.md`
- `release-notes/v0.2.4-resource-mailbox-access-check.md`

### 更新文件

- `README.md`
- `CHANGELOG.md`
- `sharepoint/list-schema.md`
- `power-automate/README.md`
- `adaptive-cards/README.md`
- `test-cases/function-test-plan.md`

## 五、下一階段工作

| 優先順序 | 工作項目 | 說明 |
|---:|---|---|
| 1 | [已完成] 解除 Resource Calendar 讀取阻擋 | 已改用「傳送 HTTP 要求」+ `calendar/events` 直連 Graph，ATA-9627 驗證成功，不需 Calendar ID |
| 2 | 對 Camry、Cross 重複驗證 | 沿用 ATA-9627 相同的 HTTP 要求路徑，逐台確認可讀取事件 |
| 3 | 建立正式行事曆同步流程 | 將 Outlook 預約同步到 SharePoint List，並寫入已建立的防呆欄位 |
| 4 | 建立取消與異動同步 | 行事曆取消、時間、車輛、主旨或借用人異動時同步更新 |
| 5 | 建立通知時間計算 | 依 `Asia/Taipei`、`是否整天`、`借用起始時間` 計算 `預計通知時間` |
| 6 | 建立 Teams Adaptive Card 發送流程 | 到達 `預計通知時間` 後發送給借用人，並避免重複通知 |
| 7 | 建立回覆前防呆檢查 | 檢查 Event ID、卡片版本、事件狀態，避免舊卡片覆蓋資料 |
| 8 | 建立 Flow 執行鎖 | 啟用 Concurrency Control，避免重複資料與 Race Condition |
| 9 | 執行端到端與防呆測試 | 從 Outlook 預約到 Teams 回覆到 SharePoint 後台確認 |
| 10 | 建立操作手冊 | 提供行政、總務人員日常使用與異常處理 |
| 11 | 上線前驗收 | 五項防呆機制與所有端到端測試通過後才可上線 |

## 六、Master 專案紀錄更新狀態

| 項目 | 狀態 |
|---|---|
| README | 已更新 |
| CHANGELOG | 已更新 |
| 專案里程碑 | 已更新 |
| 完成清單 | 已更新 |
| 待辦事項 | 已更新 |

## 七、目前判斷

本專案目前已完成後台資料結構、系統防呆欄位、基礎連線測試，並已解除資源行事曆存取閘門阻擋。SharePoint 與 Outlook 連接器驗證正常，ATA-9627 已可透過 Power Automate 正式讀取真實借用事件。

目前不需要以 Exchange Administrator 登入或執行流程。`ad.general@alp.global` 已具備三台資源信箱 Mailbox Full Access 權限；`取得行事曆 (V2)` 與 `取得事件的行事曆檢視 (V3)` 兩個動作經證實無法讀取 Resource Mailbox（屬連接器限制，非權限問題），已改用 Office 365 Outlook 標準連接器內建的「傳送 HTTP 要求」動作直連 Graph（`GET /v1.0/users/{mailbox}/calendar/events?$filter=...`），不需要 Calendar ID、不需要 Premium 授權。ATA-9627 已驗證成功，下一步為 Camry、Cross 重複驗證，並建立正式 SharePoint 同步流程。
## v0.2.7 Master 更新 - ATA-9627 V3 事件讀取測試

更新日期：2026-07-14

目前版本更新為 `v0.2.7`，本階段為 Resource Mailbox 事件讀取驗證階段。

### 本次新增完成內容

- 紀錄 ATA-9627 Resource Mailbox 的 Outlook Editor 權限已由 IT 設定完成。
- 紀錄 `ad.general@alp.global` 已可在 Outlook 端看到、建立、修改、刪除 ATA-9627 行事曆事件。
- 紀錄 Power Automate `取得行事曆 (V2)` 雖執行成功，但僅回傳 `ad.general@alp.global` 自身 Calendar。
- 新增下一階段 V3 事件讀取測試流程設計。
- 新增 `docs/ata9627-v3-calendar-event-test.md` 測試紀錄文件。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| Outlook Editor 權限驗證 | 已完成 | 100% | ATA-9627 已在 Outlook 端驗證 |
| 取得行事曆 (V2) 測試 | 已完成 | 100% | 僅回傳自身 Calendar |
| V3 Resource Calendar 事件讀取 | 待測 | 0% | 下一步測 `subject = TEST` |
| 三台車 Calendar ID 確認 | 進行中 | 33% | 先測 ATA-9627 |
| SharePoint 同步正式串接 | 待續 | 0% | 等 V3 讀取成功後進行 |

### 下一階段工作

1. 在 Power Automate 建立 `公務車功能測試-ATA9627事件讀取`。
2. 使用 `取得事件的行事曆檢視 (V3)`，Calendar Id 改填當時的候選 GUID `6049e1d1-b34c-4cca-b530-2c7c4b77abe9`。
3. 查詢 `utcNow()` 到 `addDays(utcNow(),7)`。
4. 確認是否可讀到主旨 `TEST`。
5. 回填 Calendar Id 實際設定方式、錯誤訊息或成功輸出。
6. 成功後再套用 Camry 與 Cross。

## v0.2.8 Master 更新 - ATA-9627 Calendar ID 已取得

更新日期：2026-07-15

本階段曾取得並視為 ATA-9627 Calendar ID 的候選 GUID：

`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`

### 本次新增完成內容

- 將 ATA-9627 Calendar ID 記錄至 README、Power Automate 設計、測試紀錄與 Master 紀錄。
- 將 V3 事件讀取測試值由資源信箱 Email 改為候選 GUID；其有效性已於 v0.2.10 被實測推翻。
- 明確保留 V3 讀取狀態為待執行，不提前標記為通過。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| ATA-9627 候選 GUID 紀錄 | 已完成 | 100% | v0.2.10 已證實不適用 V3 |
| ATA-9627 V3 事件讀取 | 待執行 | 0% | 需實際執行 Flow |
| Camry / Cross Calendar ID 確認 | 待開始 | 0% | ATA-9627 成功後再處理 |
| SharePoint 同步正式串接 | 待續 | 0% | 等三台車讀取成功後進行 |

### 下一階段工作

1. 在 Power Automate 開啟 `公務車功能測試-ATA9627事件讀取`。
2. 將 `取得事件的行事曆檢視 (V3)` 的 Calendar Id 設為 `6049e1d1-b34c-4cca-b530-2c7c4b77abe9`。
3. 查詢期間使用 `utcNow()` 至 `addDays(utcNow(),7)`。
4. 執行 Flow，確認是否可取得主旨、起始時間、結束時間、是否全天、建立者／借用人、Event ID 與 iCalUId。
5. 若成功，回填測試結果並進入 Camry、Cross Calendar ID 取得與測試。

## v0.2.9 Master 更新 - ATA-9627 V3 測試實作版

更新日期：2026-07-15

本階段已將 ATA-9627 V3 事件讀取測試整理成 Power Automate 可落地執行版本，文件位置：

`power-automate/ata9627-v3-test-implementation.md`

### 本次新增完成內容

- 建立 `公務車功能測試-ATA9627事件讀取` 的實作步驟。
- 指定 V3 動作 Calendar Id 使用 `6049e1d1-b34c-4cca-b530-2c7c4b77abe9`。
- 新增事件數量、TEST 篩選、第一筆原始事件、欄位摘要等測試輸出設計。
- 明確定義 V3 成功前不得進入正式 SharePoint 同步流程。
- 建立成功後下一步 `公務車行事曆同步至 SharePoint - ATA9627 MVP` 草案。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| ATA-9627 候選 GUID 紀錄 | 已完成 | 100% | v0.2.10 已證實不適用 V3 |
| ATA-9627 V3 測試實作版 | 已完成 | 100% | 已建立可照 Power Automate 畫面操作的步驟 |
| ATA-9627 V3 實際執行 | 已執行／未通過 | 100% | `ErrorInvalidIdMalformed` |
| SharePoint 同步 MVP | 待開始 | 0% | 等 V3 事件讀取成功後才能建立 |

### 下一階段工作

1. 在 Power Automate 依 `power-automate/ata9627-v3-test-implementation.md` 建立或調整測試流程。
2. 執行 `公務車功能測試-ATA9627事件讀取`。
3. 回填全部事件數量、TEST 事件數量、第一筆事件原始資料。
4. 確認是否取得 subject、start、end、isAllDay、id、iCalUId。
5. 若成功，建立 `公務車行事曆同步至 SharePoint - ATA9627 MVP`。

## v0.2.10 Master 更新 - ATA-9627 候選 GUID V3 實測未通過

更新日期：2026-07-18

### 實測證據

| 項目 | 值 |
|---|---|
| Flow | `公務車功能測試-ATA9627事件讀取` |
| Flow ID | `7d66adc8-eccd-4cc0-9ab1-a031a6676df9` |
| Run ID | `08584172227294871773330429620CU04` |
| Calendar Id 輸入 | `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |
| 查詢期間 | `2026-07-14T00:00:00Z` 至 `2026-07-21T23:59:59Z` |
| 結果 | `400 BadRequest` |
| 錯誤 | `ErrorInvalidIdMalformed` / `The Id is invalid.` |
| clientRequestId | `6edfd333-e4b0-4d2d-a3e7-46ced3ee4270` |
| serviceRequestId | `8db0eba7-f781-4dd8-aacc-d7e4d81d3c70` |

### Master 判定

- `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` 為候選物件 GUID，不是已驗證可供 Office 365 Outlook V3 使用的 Calendar ID。
- CAL-V3-06 未通過；CAL-V3-07、CAL-V3-08 因事件讀取未成立而阻擋。
- 本次不是 Access Denied，且 Calendar Id 已由執行輸入證實確實送達連接器。
- `公務車功能測試-ATA9627事件讀取` 仍顯示「開啟」且為每分鐘排程，需優先人工停用。
- M5 維持進行中／受阻；不得建立正式 SharePoint 同步 MVP。

## v0.2.11 Master 更新 - Resource Calendar 讀取阻擋解除

更新日期：2026-07-23

### 本次新增完成內容

- IT 已將 `ad.general@alp.global` 對三台公務車 Resource Mailbox 的權限提升為 Mailbox Full Access。
- 重測 `取得行事曆 (V2)` 與 `取得事件的行事曆檢視 (V3)`：確認兩者皆為連接器動作限制，僅能存取連線帳號自己的行事曆，與權限層級無關。
- 確認 `公務車功能測試-ATA9627事件讀取` 已由「開啟／每分鐘排程」變更為「關閉」。
- 驗證 `取得電子郵件 (V3)` 的 `原始信箱地址` 參數可指定 Resource Mailbox 並成功讀信，證實 Full Access 對郵件類動作有效。
- 找到正式解法：Office 365 Outlook 標準連接器（非 Premium）的「傳送 HTTP 要求」動作可直連 Graph，白名單物件包含 `calendar`、`events`、`calendars`。
- 以 `GET /v1.0/users/room_nhb4_car@alp.global/calendar/events?$filter=...` 實測成功（`statusCode 200`），取得 ATA-9627 真實借用紀錄，欄位完整（主旨、起訖時間、是否全天、借用人、iCalUId、最後修改時間、地點、重複規則）。
- 新增測試紀錄：`docs/ata9627-v3-calendar-event-test.md` 第十四節；新增 `release-notes/v0.2.11.md`。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| ATA-9627 Resource Calendar 讀取 | 已完成 | 100% | HTTP 要求 + `calendar/events` + `$filter` 驗證成功 |
| Camry / Cross Resource Calendar 讀取 | 待驗證 | 0% | 沿用 ATA-9627 相同路徑，待逐台實測 |
| 正式行事曆同步流程 | 待開始 | 0% | 讀取路徑已確認，可開始建置 |
| SharePoint 同步 MVP | 待開始 | 0% | 等三台車讀取驗證完成後同步進行 |

### Master 判定

- M5 Resource Calendar 事件讀取正式解除阻擋，判定為**通過**。
- 不需要 Exchange Administrator、Power Automate Premium 授權或額外 Azure AD App。
- 可以正式進入 `公務車行事曆同步至 SharePoint` 流程建置階段。

### 下一階段工作

1. 對 Camry（`room_nhb4_car_camry@alp.global`）、Cross（`room_nhb4_car_cross@alp.global`）重複相同的 HTTP 要求驗證。
2. 建立正式流程 `公務車行事曆同步至 SharePoint`，三台車共用同一組讀取、解析、寫入邏輯。
3. 依 `預約唯一鍵`（資源信箱 + 行事曆事件 ID）判斷新增或更新 SharePoint 項目。
4. 計算 `是否整天`、`預計通知時間`，並將 UTC 時間轉換為 `Asia/Taipei`。
5. 完成後建立下一個階段版本記錄，並依 `docs/version-control-workflow.md` 執行 commit、push、tag 與 GitHub Release。

## v0.2.12 Master 更新 - 正式行事曆同步流程建立並啟用排程

更新日期：2026-07-23

### 本次新增完成內容

- 完成三台公務車共用的正式流程 `公務車行事曆同步至SharePoint`（Flow ID `b6d1ec5c-0fc5-46c1-85b0-d8d68c72c0ce`）：Recurrence（15 分鐘）→ 車輛清單 → 套用至各項（三台車）→ 傳送 HTTP 要求 → 剖析 JSON → 套用至各項 1（逐筆事件）→ 計算預約唯一鍵／預計通知時間 → SharePoint 取得多個項目 → 條件 → 更新項目／建立項目。
- 依 `sharepoint/list-schema.md` 於「更新項目」「建立項目」兩分支完整寫入約 40 個欄位，含系統防呆欄位預設值。
- 排除設計問題：確認 Power Automate 動作／迴圈顯示名稱含空格時，從外部以名稱參照會被轉換為底線（例如 `套用至各項 1` 需寫成 `items('套用至各項_1')`），修正「更新項目」約 13 個受影響欄位。
- 流程儲存驗證零錯誤，流程檢查程式錯誤 0、警告 0。
- 已將流程由「關閉」切換為「開啟」，正式進入每 15 分鐘排程運作。
- 新增 `release-notes/v0.2.12.md`，更新 `docs/todo.md`、`power-automate/README.md`、`CHANGELOG.md`。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| ATA-9627 / Camry / Cross Resource Calendar 讀取 | 已完成 | 100% | 三台車皆採用相同 HTTP 直連 Graph 路徑 |
| 正式行事曆同步流程建立 | 已完成 | 100% | 已儲存零錯誤並開啟排程 |
| 正式行事曆同步流程端到端測試 | 待開始 | 0% | 尚未以實際 Outlook 預約驗證同步結果 |
| Teams 通知與回覆流程 | 待開始 | 0% | 等同步流程端到端測試通過後進行 |

### Master 判定

- M6「建立正式 `公務車行事曆同步至 SharePoint` 流程」正式完成，判定為**通過**。
- 流程已上線排程（每 15 分鐘執行一次），惟尚未完成端到端測試，`是否測試資料` 暫維持 `是` 作為安全預設值。
- 尚未建立 Concurrency Control，暫存測試流程 `公務車功能測試-郵件動作參數檢查(可刪除)` 尚未刪除，皆列為下一階段工作。

### 下一階段工作

1. 執行端到端測試：實際於 Outlook 建立／修改／取消預約，確認流程可在 15 分鐘內正確同步至 SharePoint。
2. 依 `docs/todo.md` P3 測試各項情境（整天借用、早上過早借用、取消、時間異動、重複觸發等）。
3. 端到端測試通過後，將 `是否測試資料` 切換為正式運作邏輯，並刪除暫存測試流程。
4. 建立 Concurrency Control，降低重複同步風險。
5. 進入 P2 階段，建立「公務車借用前 Teams 通知與回覆」流程。

## v0.2.13 Master 更新 - 執行期錯誤修正並完成首次成功執行驗證

更新日期：2026-07-24

### 本次新增完成內容

- v0.2.12 儲存驗證雖為零錯誤，但開啟排程後所有執行紀錄皆顯示失敗；本版本逐一排查並修正四種造成執行期失敗的錯誤：
  1. 兩處 Apply to each 迴圈的 `foreach` 參數誤加大括號（`@{items(...)}`），修正為 `@items(...)`。
  2. 「取得多個項目」篩選查詢誤用中文欄位顯示名稱，改以 SharePoint REST API 驗證後的正確內部名稱 `OData__x9810__x7d04__x552f__x4e00__x93`。
  3. 「條件」判斷式 `greater` 比較誤加大括號，修正為 `@length(...)`。
  4. 「建立項目」「更新項目」的「是否整天」欄位誤寫入中文字串，改為原生布林參照 `@items('套用至各項_1')?['isAllDay']`。
- 修正後手動觸發測試，運行紀錄首次出現「測試成功」；逐層展開執行追蹤確認三台車、內層事件迴圈、SharePoint 取得/建立/更新項目皆成功執行。
- 以 SharePoint REST API 直接查詢清單，確認三筆借用紀錄正確寫入，「是否整天」欄位為真正布林值（非文字字串）；後續排程再次執行後項目數未增加，確認「預約唯一鍵」防重複比對邏輯正常運作。
- 新增 `release-notes/v0.2.13.md`，更新 `docs/todo.md`、`power-automate/README.md`、`CHANGELOG.md`。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| 正式行事曆同步流程執行期錯誤修正 | 已完成 | 100% | 四種錯誤皆已修正並驗證 |
| 首次成功執行驗證 | 已完成 | 100% | 運行紀錄「測試成功」，SharePoint 資料驗證通過 |
| P3 各項情境測試 | 待開始 | 0% | 取消、時間異動、整天借用、重複觸發等尚待測試 |
| Teams 通知與回覆流程 | 待開始 | 0% | 等 P3 測試通過後進行 |

### Master 判定

- 正式流程 `公務車行事曆同步至SharePoint` 完成首次真正端到端成功執行，並以 SharePoint 實際資料驗證寫入正確性，判定為**通過**。
- v0.2.12「流程儲存驗證零錯誤」之敘述，就實際執行結果而言並不完整；本版本已補充說明並修正執行期錯誤，後續版本紀錄應留意「儲存零錯誤」不等於「執行成功」。
- 尚未涵蓋取消、時間異動等情境，`是否測試資料` 暫維持 `是`，Concurrency Control 與暫存測試流程刪除仍列為下一階段工作。

### 下一階段工作

1. 執行 P3 各項情境測試（取消、時間異動、整天借用、重複觸發等）。
2. 確認「我的流程」清單中是否有本流程的重複項目需清理。
3. 通過後將 `是否測試資料` 切換為正式運作邏輯，並刪除暫存測試流程。
4. 建立 Concurrency Control，降低重複同步風險。
5. 進入 P2 階段，建立「公務車借用前 Teams 通知與回覆」流程。

## v0.2.14 Master 更新 - 修正「更新項目」時間欄位 +8 小時位移嚴重錯誤

更新日期：2026-07-24

### 本次新增完成內容

- 執行 P3 情境測試時，透過測試紀錄連續三次 SharePoint REST API 比對，發現「更新項目」（Update，PATCH/MERGE）動作寫入借用起始時間、借用結束時間、預計通知時間、事件最後修改時間時，比「建立項目」（Create，POST）多了 8 小時（Taipei 時區 UTC 位移量），即使兩者運算式與來源資料完全相同。
- 以剖析 JSON 原始輸出比對前後兩次執行，確認 Graph 來源資料未變，排除運算式撰寫錯誤或來源資料變動；鎖定成因為 SharePoint 連接器 Create／Update 對「不含時區後綴的模糊時間字串」解讀邏輯不一致。
- 修正「更新項目」動作：借用起始/結束時間、最後同步時間、事件最後修改時間，移除不必要的 `convertTimeZone` 轉換，改為直接輸出 UTC 原始時間並明確加上 `Z` 時區後綴；預計通知時間因參照與「建立項目」共用的 Compose 輸出，改為在引用處以 `addHours(...,-8)` 反推回 UTC 再補 `Z` 後綴，避免影響「建立項目」既有正確行為。
- 儲存流程確認零錯誤，手動觸發測試（測試紀錄已存在，故走「更新項目」分支），以 SharePoint REST API 驗證借用起始/結束時間、預計通知時間皆恢復與「建立項目」當下一致的正確值。
- 新增 `release-notes/v0.2.14.md`，更新 `docs/todo.md`、`CHANGELOG.md`、`power-automate/README.md`。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| 「更新項目」時間欄位 +8 小時位移錯誤修正 | 已完成 | 100% | 已鎖定成因並以測試紀錄驗證通過 |
| 正式紀錄（Id=1、Id=2）時間欄位回復抽查 | 待執行 | 0% | 需等下次排程自動更新後抽查 |
| P3 其餘情境測試 | 待開始 | 0% | 取消、時間異動、整天借用、重複觸發等尚待測試 |
| Teams 通知與回覆流程 | 待開始 | 0% | 等 P3 測試通過後進行 |

### Master 判定

- 「更新項目」動作時間欄位 +8 小時位移錯誤，判定為**已修正並驗證通過**（以測試紀錄即時驗證）。
- 本版本修正並補充 v0.2.13「判定：通過」結論之不足——該結論僅驗證 Create 路徑，未涵蓋排程自動觸發的 Update 路徑；此為本專案文件慣例中「儲存零錯誤不等於執行正確」原則的再一次體現，這次更進一步顯示「單次執行正確不等於所有路徑（Create／Update）皆正確」。
- 正式（非測試）紀錄 Id=1、Id=2 過去因走過 Update 路徑可能已被寫入錯誤時間，待其下次排程自動更新後應會被本次修正覆寫回正確值，建議另行抽查確認。

### 下一階段工作

1. 抽查正式紀錄 Id=1、Id=2 於下次排程執行後，時間欄位是否已回復正確。
2. 繼續執行 P3 其餘情境測試（取消、時間異動、整天借用、重複觸發等）。
3. 確認「我的流程」清單中是否有本流程的重複項目需清理。
4. 通過後將 `是否測試資料` 切換為正式運作邏輯，並刪除暫存測試流程、建立 Concurrency Control。
5. 進入 P2 階段，建立「公務車借用前 Teams 通知與回覆」流程。

## v0.2.14 追加更新 - 正式紀錄抽查與 P3 情境測試結果

更新日期：2026-07-27

### 本次新增完成內容

- 以 SharePoint REST API 抽查正式紀錄 Id=1、Id=2，確認於下次排程自動觸發後，兩筆紀錄的借用起始/結束時間、預計通知時間皆已回復正確（與修正前相差精準 8 小時），且 `Modified` 時間皆晚於 v0.2.14 修正部署時間，證實修正對既有正式資料同樣有效。
- 執行 P3 情境測試：
  - **時間異動（FT-102／SG-T05 部分）**：通過。將測試事件時間由 03:00-04:00 改為 10:00-11:00，SharePoint 正確更新起訖時間與預計通知時間。
  - **整天借用（SG-T02／SG-T03）**：通過。以 Id=1 抽查證據確認起訖時間與預計通知時間皆為當天 08:00，符合整天借用邏輯，且不會產生前一天的錯誤通知時間。
  - **重複觸發（SG-T07）**：通過。以歷史執行記錄佐證，同一批事件經排程每 15 分鐘重複觸發數十次，SharePoint 對應紀錄皆維持單筆，Event ID/iCalUId 比對機制有效防止重複建立。
  - **取消預約（FT-103／SG-T04）**：**未通過，發現新問題**。刪除 Outlook 事件後，流程的「取得多個項目」不再回傳該事件，目前無任何機制偵測事件消失並將 SharePoint 紀錄標記為已取消；SharePoint 對應紀錄會維持在刪除前的最後狀態，`已取消` 仍為否，後續通知不會停止。
- 診斷發現一項操作性事實（非流程錯誤）：流程的行事曆讀取為未來/當下時間範圍導向，過去日期的事件不會被讀取到；測試時若事件日期已成為過去，即使流程執行成功，也不會反映任何異動，需將測試事件移至未來日期方能正確驗證。
- 已更新 `test-cases/function-test-plan.md`（FT-101～FT-103、SG-T01～SG-T07 狀態列）、`docs/todo.md`（v0.2.14 待辦事項第 6～8 項）。

### 目前狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---|---|
| 正式紀錄（Id=1、Id=2）時間欄位回復抽查 | 已完成 | 100% | 兩筆紀錄時間欄位皆已確認回復正確 |
| P3 情境測試 - 時間異動 | 已完成 | 100% | 通過 |
| P3 情境測試 - 整天借用 | 已完成 | 100% | 通過（以 Id=1 抽查證據佐證） |
| P3 情境測試 - 重複觸發 | 已完成 | 100% | 通過（以歷史執行記錄佐證） |
| P3 情境測試 - 取消預約 | 已完成（結果：未通過） | 100% | 發現新問題：事件刪除無法反映至 SharePoint，待補強偵測邏輯 |
| 事件取消偵測補強邏輯 | 待開始 | 0% | 新增待辦，需比對 SharePoint 既有紀錄與行事曆讀取結果，標記消失的 Event ID 為已取消 |
| Teams 通知與回覆流程 | 待開始 | 0% | 等取消偵測補強完成、`是否測試資料` 切換後進行 |

### Master 判定

- 正式紀錄 Id=1、Id=2 時間欄位回復，判定為**已驗證通過**。
- P3 情境測試中，時間異動、整天借用、重複觸發三項，判定為**通過**。
- 取消預約情境，判定為**未通過**：這是本專案迄今發現的第一個非時區類、屬於「架構缺口」的問題——流程設計上從未包含「偵測資料消失」的比對步驟，需在下一開發階段新增此邏輯，而非單純修正運算式即可解決。
- 測試紀錄 Id=4（其 Outlook 來源事件已刪除、SharePoint 紀錄未被標記取消）予以保留，作為此問題的可重現證據，暫不清理。

### 下一階段工作

1. 設計並實作事件取消偵測補強邏輯（比對 SharePoint 既有紀錄與本次行事曆讀取結果，標記消失的 Event ID 為已取消，停止後續通知）。
2. 待車輛、主旨、借用人異動與舊卡片失效機制（SG-T05 其餘部分、SG-T06）於 Teams Adaptive Card 流程開發後一併測試。
3. 確認「我的流程」清單中是否有本流程的重複項目需清理。
4. 全部通過後將 `是否測試資料` 切換為正式運作邏輯，並刪除暫存測試流程、建立 Concurrency Control。
5. 進入 P2 階段，建立「公務車借用前 Teams 通知與回覆」流程。
