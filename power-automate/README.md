# Power Automate 流程設計與現況

## 設計原則

- 使用 Microsoft 365 / O365 E1 可用的標準連接器。
- 不使用 Power Automate Premium 連接器。
- 先完成功能測試，再啟用正式自動化。
- SharePoint List 作為流程資料主檔。
- Teams Adaptive Card 回覆必須寫回 SharePoint。
- 正式上線前必須完成五項系統防呆設計：統一時區、唯一識別碼、取消／異動同步、舊卡片失效、Flow 防重複。

## 目前已建立流程

| 項目 | 內容 |
|---|---|
| 流程名稱 | `公務車功能測試-SharePoint清單連線` |
| Flow ID | `272f59da-3a88-40b5-a3f7-4cf8cb56ca5b` |
| 環境 | `Default-0e690dd6-da00-4897-bf51-a91c687a02bd` |
| 目的 | 驗證 Power Automate 可連線 SharePoint List 與 Office 365 Outlook |
| 狀態 | 已建立，最新測試成功 |

## 已完成測試

| 測試項目 | 結果 | 備註 |
|---|---|---|
| SharePoint `取得多個項目` | 通過 | 可讀取 `ALP_TW_AD` 的 `公務車借用管理` 清單 |
| Office 365 Outlook 標準連接器驗證 | 通過 | `ad.general@alp.global` 可建立連線 |
| 資源行事曆 Calendar ID 取得 | 未通過 | Outlook 已顯示三台共用行事曆後，`取得行事曆 (V2)` 仍只回傳個人 `Calendar`；資源信箱 Email 不能直接當 Calendar ID |
| 流程執行紀錄 | 通過 | 2026-07-01 下午 05:07 顯示成功 |

## 資源信箱讀取實測

| 項目 | 內容 |
|---|---|
| 測試日期 | 2026-07-02（Asia/Taipei） |
| 流程名稱 | `公務車功能測試-資源信箱讀取` |
| Flow ID | `7aff726f-0f9d-4b7e-a128-baeb368ab1ce` |
| 測試動作 | `取得事件的行事曆檢視 (V3)` |
| 實測信箱 | `room_nhb4_car@alp.global` |
| 結果 | 失敗：`ID 格式不正確`（BadRequest） |
| 流程狀態 | 已關閉，保留執行紀錄供追蹤 |

本次結果代表 Outlook 連接器驗證成功，但資源信箱 Email 不能直接作為 Calendar ID。`ad.general@alp.global` 已具有三台車 Calendar Reviewer 權限，不需要 Exchange Administrator；2026-07-04 已確認 Outlook 網頁可顯示三台公務車共用行事曆，但 `取得行事曆 (V2)` 仍只回傳個人 Calendar。下一步需評估 Editor 權限或其他不使用 Premium 連接器的標準讀取方案，再依序重測 Altis、Camry、Cross。

## 尚未完成正式流程

正式流程建議拆成兩條，降低維護難度。

### 流程一：公務車行事曆同步至 SharePoint

目的：定期讀取三台公務車 Outlook 資源行事曆，將預約資料同步到 SharePoint List。

預計步驟：

1. 排程觸發，例如每 15 分鐘或每 30 分鐘執行。
2. 讀取三台資源信箱的行事曆預約。
3. 取得借用日期、起訖時間、借用人、主旨、車輛名稱、Event ID、iCalUId、事件最後修改時間。
4. 將 Outlook 原始時間轉換為 `Asia/Taipei`，並保留 UTC 原始時間供稽核。
5. 組合 `預約唯一鍵 = 資源信箱 + "|" + Event ID`。
6. 依 `預約唯一鍵` 查詢 SharePoint，已存在則更新，不存在才新增。
7. 判斷是否為整天預約，寫入 `是否整天`。
8. 計算 `預計通知時間`，全天事件固定為當天 08:00。
9. 預設 `領鑰狀態` 為 `待通知`，`事件同步狀態` 為 `有效`。
10. 若 Outlook 預約取消，更新 `領鑰狀態` 為 `已取消`，`事件同步狀態` 為 `已取消`。
11. 若 Outlook 預約異動，更新 SharePoint、重算通知時間，並將舊 `卡片版本` 標記為失效。

### 流程二：公務車借用前 Teams 通知與回覆

目的：到達預計通知時間後，發送 Teams Adaptive Card 給借用人，並將回覆寫回 SharePoint。

預計步驟：

1. 排程觸發，例如每 5 分鐘執行。
2. 查詢 SharePoint 中 `領鑰狀態 = 待通知` 或 `未完成填寫` 的項目。
3. 判斷目前台北時間是否已達 `預計通知時間`。
4. 發送前檢查 `通知狀態`、`通知發送時間`、`最新通知批次 ID`，避免重複發送。
5. 發送 Teams Adaptive Card 給 `借用人 Email`，並帶入 `sharePointItemId`、`eventId`、`iCalUId`、`cardVersion`。
6. 借用人填寫共乘人數並勾選規範確認。
7. 回覆寫回前，重新讀取 SharePoint 最新項目，檢查事件 ID、卡片版本與狀態。
8. 若資料仍有效，寫回 `共乘人數`、`規範確認狀態`、`Teams 回覆時間`、`通知發送時間`。
9. 若資料完整，更新 `領鑰狀態` 為 `已完成確認`，`卡片狀態` 為 `已回覆`。
10. 若資料不完整，維持或更新為 `未完成填寫`。
11. 若項目已取消、已失效或已異動，停止更新，並提示借用人重新確認最新借用資訊。

## 通知時間邏輯

### 規則

- 一般借用：借用起始時間前 1 小時通知。
- 最早通知時間：借用當天早上 08:00。
- 整天借用：借用當天早上 08:00。

### 判斷方式

| 情境 | 預計通知時間 |
|---|---|
| `是否整天 = 是` | 借用日期 08:00 |
| `借用起始時間 - 1 小時` 早於 08:00 | 借用日期 08:00 |
| 其他一般借用 | 借用起始時間 - 1 小時 |

### 寫入欄位

- `預計通知時間`：流程一計算後寫入。
- `通知發送時間`：流程二實際發送 Teams Adaptive Card 後寫入。

## 系統防呆流程改善

### 1. 統一時區

- Outlook 原始時間需先轉為 `Taipei Standard Time`。
- SharePoint 後台顯示與通知判斷一律使用 `Asia/Taipei`。
- 全天事件不得用 UTC 午夜扣 1 小時計算，必須直接指定當天 08:00。

### 2. Event ID / iCalUId 唯一比對

- 每次新增或更新前，先用 `預約唯一鍵` 查詢 SharePoint。
- `預約唯一鍵` 建議格式：`資源信箱 + "|" + Event ID`。
- 若 Event ID 不可用，才以 `資源信箱 + "|" + iCalUId` 作輔助比對。
- 不得只用日期、主旨、借用人姓名或 Email 比對。

### 3. 取消與異動同步

- Outlook 預約異動時，同步更新 SharePoint 的借用時間、主旨、借用人、車輛與通知時間。
- Outlook 預約取消時，SharePoint `領鑰狀態` 更新為 `已取消`。
- 取消或異動後，舊 Teams Adaptive Card 必須改為失效，不得再更新資料。

### 4. 舊 Adaptive Card 失效

回覆前必須重新檢查 SharePoint：

| 檢查項目 | 通過條件 |
|---|---|
| Event ID | 卡片送回的 Event ID 等於 SharePoint 最新 Event ID |
| 卡片版本 | 卡片送回的版本等於 SharePoint 最新版本 |
| 領鑰狀態 | 不是 `已取消`、`已完成確認`、`已領鑰`、`已失效` |
| 事件同步狀態 | 仍為 `有效` |

任一條件不通過，回覆：

```text
此借用已取消或已變更，請重新確認最新借用資訊。
```

### 5. Flow 執行鎖與重複防止

- 同步流程與通知流程啟用 Concurrency Control。
- 建議同步流程 Degree of Parallelism 設為 `1`。
- 若 SharePoint 查到同一 `預約唯一鍵` 多筆資料，標記 `重複異常` 並停止通知。
- Teams 發送前檢查 `通知狀態` 與 `最新通知批次 ID`，避免重複通知。

## 待確認事項

| 項目 | 狀態 | 說明 |
|---|---|---|
| 三台資源信箱讀取權限 | 部分具備 | `ad.general@alp.global` 已有 Calendar Reviewer 且 Outlook 看得到三台車；但 Power Automate V2 未列出，待評估 Editor 或替代方案 |
| 取消預約同步 | 待實作 | 預約取消後需更新 SharePoint 狀態 |
| 時間異動同步 | 待實作 | 預約時間異動後需重算 `預計通知時間` |
| Teams 回覆寫回 | 待實作 | 需完成 Adaptive Card 回覆與 SharePoint 更新 |
| 端到端測試 | 待執行 | 從 Outlook 預約到 Teams 回覆到領鑰判斷 |
| 系統防呆設計實作 | 待實作 | 時區、唯一鍵、取消異動、舊卡失效、Flow 防重複皆為上線前必要條件 |
## v0.2.7 測試流程 - ATA-9627 事件讀取

請新增一支獨立測試流程，不直接修改正式同步流程。

流程名稱：

`公務車功能測試-ATA9627事件讀取`

流程目的：

驗證 Office 365 Outlook 標準連接器是否能讀取 `room_nhb4_car@alp.global` 的行事曆事件。

建議動作：

| 順序 | 動作 | 說明 |
|---|---|---|
| 1 | 手動觸發流程 | 方便反覆測試 |
| 2 | 取得事件的行事曆檢視 (V3) | Calendar Id 填 `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |
| 3 | 篩選陣列 | 篩選主旨等於 `TEST` 的事件 |
| 4 | Compose - 測試事件數量 | 顯示找到幾筆 TEST |
| 5 | Compose - 事件摘要 | 顯示 subject、start、end、id、iCalUId |
| 6 | 條件判斷 | 找到 TEST 即判定本階段成功 |

V3 查詢期間：

- 開始時間：`utcNow()`
- 結束時間：`addDays(utcNow(),7)`

注意事項：

- 文件與 SharePoint 顯示時間一律使用 `Asia/Taipei`。
- 若 V3 動作要求 UTC 格式，查詢可使用 UTC，但輸出給承辦人確認時需轉換為台北時間。
- 不再使用 `room_nhb4_car@alp.global` 作為 V3 Calendar Id；請改用 ATA-9627 真正 Calendar ID `6049e1d1-b34c-4cca-b530-2c7c4b77abe9`。
- 若 GUID 仍失敗，需保留完整錯誤訊息，並記錄是 Calendar Id、連接器限制、快取或權限問題。

## v0.2.8 更新 - ATA-9627 Calendar ID 測試值

2026-07-15 已取得 ATA-9627 真正 Calendar ID：

`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`

下一步請執行 `公務車功能測試-ATA9627事件讀取`，確認 V3 動作是否可讀取未來 7 天事件。成功後才可把同樣方法套用到 Camry 與 Cross，並進入正式 SharePoint 同步流程。

## v0.2.9 更新 - ATA-9627 V3 測試實作版

已新增實作文件：

`power-automate/ata9627-v3-test-implementation.md`

本版本將 `公務車功能測試-ATA9627事件讀取` 拆成可直接建立的 Power Automate 動作：

| 順序 | 動作名稱 | 用途 |
|---|---|---|
| 1 | 手動觸發流程 | 手動按測試 |
| 2 | `Get_ATA9627_Events_V3` | 用 Calendar ID 讀取 ATA-9627 未來 7 天事件 |
| 3 | `Filter_TEST_Events` | 找出主旨為 `TEST` 的事件 |
| 4 | `Compose_All_Event_Count` | 計算全部事件數量 |
| 5 | `Compose_TEST_Event_Count` | 計算 TEST 事件數量 |
| 6 | `Compose_First_Event_Raw` | 顯示第一筆事件原始資料 |
| 7 | `Select_Event_Field_Check` | 檢查 subject、start、end、isAllDay、id、iCalUId |

本階段判斷規則：

- 若 V3 可取得事件，且可取得 Event ID 或 iCalUId，才可進入 `公務車行事曆同步至 SharePoint - ATA9627 MVP`。
- 若 V3 失敗，需先依錯誤訊息判斷是 Calendar ID、權限、連接器限制或查詢期間問題。
- 在 V3 測試成功前，不啟用正式 SharePoint 同步。
