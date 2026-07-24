# Power Automate 流程設計與現況

## 設計原則

- 使用 Microsoft 365 / O365 E1 可用的標準連接器。
- 不使用 Power Automate Premium 連接器。
- 先完成功能測試，再啟用正式自動化。
- SharePoint List 作為流程資料主檔。
- Teams Adaptive Card 回覆必須寫回 SharePoint。
- 正式上線前必須完成五項系統防呆設計：統一時區、唯一識別碼、取消／異動同步、舊卡片失效、Flow 防重複。

## 目前已建立流程

### 正式流程：公務車行事曆同步至SharePoint（v0.2.13 已修正執行期錯誤並完成首次成功執行）

| 項目 | 內容 |
|---|---|
| 流程名稱 | `公務車行事曆同步至SharePoint` |
| Flow ID | `b6d1ec5c-0fc5-46c1-85b0-d8d68c72c0ce` |
| 環境 | `Default-0e690dd6-da00-4897-bf51-a91c687a02bd` |
| 觸發方式 | Recurrence，每 15 分鐘 |
| 狀態 | 已開啟（排程運作中），已確認可成功執行 |
| 結構 | Recurrence → 車輛清單（Compose）→ 套用至各項（三台車）→ 傳送 HTTP 要求 → 剖析 JSON → 套用至各項 1（逐筆事件）→ 編輯（預約唯一鍵）→ 編輯 1（預計通知時間）→ 取得多個項目（SharePoint）→ 條件 → 更新項目／建立項目 |
| 儲存驗證 | 零錯誤，流程檢查程式錯誤 0、警告 0 |
| 執行期驗證 | v0.2.12 開啟排程後所有執行紀錄皆失敗；v0.2.13 修正四種執行期錯誤（迴圈 foreach 大括號、SharePoint 篩選查詢內部名稱、條件判斷式大括號、「是否整天」欄位型別）後，運行紀錄首次出現「測試成功」，並以 SharePoint REST API 驗證資料正確寫入且無重複 |
| 尚待驗證 | P3 各項情境測試（取消、時間異動、整天借用、重複觸發等） |

詳見 `release-notes/v0.2.13.md`（前次見 `release-notes/v0.2.12.md`）。

### 測試流程：公務車功能測試-SharePoint清單連線

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

## 正式流程設計（流程一已完成，流程二尚未完成）

正式流程拆成兩條，降低維護難度。

### 流程一：公務車行事曆同步至 SharePoint（v0.2.12 已完成）

目的：定期讀取三台公務車 Outlook 資源行事曆，將預約資料同步到 SharePoint List。

預計步驟：

1. 排程觸發，例如每 15 分鐘或每 30 分鐘執行。
2. 讀取三台資源信箱的行事曆預約：使用 Office 365 Outlook 連接器的「傳送 HTTP 要求」動作，呼叫 `GET /v1.0/users/{資源信箱}/calendar/events?$filter=start/dateTime ge '...' and start/dateTime le '...'`（v0.2.11 已驗證，詳見下方說明）。
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
| 三台資源信箱讀取權限 | 已完成 | 三台車皆已驗證「傳送 HTTP 要求」直連 Graph 讀取成功 |
| 正式行事曆同步流程建立 | 已完成 | v0.2.12 已建立並開啟排程，每 15 分鐘執行 |
| 正式行事曆同步流程執行期錯誤修正 | 已完成 | v0.2.13 修正四種執行期錯誤並完成首次成功執行與 SharePoint 資料驗證 |
| 正式行事曆同步流程端到端測試（P3 情境） | 待執行 | 已驗證新增與一般更新情境；取消、時間異動、整天借用、重複觸發等 P3 情境尚待測試 |
| 取消預約同步 | 待驗證 | 欄位已寫入預設值，實際取消情境尚未測試 |
| 時間異動同步 | 待驗證 | 欄位已寫入，實際異動情境尚未測試 |
| Teams 回覆寫回 | 待實作 | 需完成 Adaptive Card 回覆與 SharePoint 更新 |
| 端到端測試 | 待執行 | 從 Outlook 預約到 Teams 回覆到領鑰判斷 |
| 系統防呆設計實作 | 欄位已寫入，異動情境待驗證 | 時區、唯一鍵已於流程中實作；取消異動、舊卡失效、Flow 防重複情境仍待測試 |
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
- 不再使用 `room_nhb4_car@alp.global` 作為 V3 Calendar Id；`6049e1d1-b34c-4cca-b530-2c7c4b77abe9` 亦已於 2026-07-18 證實不是 V3 可接受的 Calendar ID。
- 若 GUID 仍失敗，需保留完整錯誤訊息，並記錄是 Calendar Id、連接器限制、快取或權限問題。

## v0.2.8 更新 - ATA-9627 Calendar ID 測試值

2026-07-15 曾取得 ATA-9627 候選物件 GUID：

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

## v0.2.10 更新 - ATA-9627 候選 GUID V3 實測未通過

2026-07-18 執行紀錄 `08584172227294871773330429620CU04` 已確認候選 GUID 實際送入 `取得事件的行事曆檢視 (V3)`，但回傳 `400 BadRequest / ErrorInvalidIdMalformed / The Id is invalid.`。

後續規則：

- 不得再把該 GUID 當成已驗證 Calendar ID。
- 先停用目前每分鐘執行且仍顯示「開啟」的測試流程。
- 取得 Office 365 Outlook 連接器可接受的 Calendar ID 後，改用受控手動測試。
- 未取得事件及 Event ID / iCalUId 前，不得建立正式 SharePoint 同步。

## v0.2.11 更新 - Resource Calendar 讀取阻擋解除（HTTP 直連 Graph）

2026-07-23：IT 將 `ad.general@alp.global` 對三台車 Resource Mailbox 的權限提升為 Mailbox Full Access 後，重測確認 `取得行事曆 (V2)` 與 `取得事件的行事曆檢視 (V3)` 兩個動作皆為連接器限制，無法讀取 Resource Mailbox，不論授予何種權限皆無效。

改用 Office 365 Outlook 標準連接器（非 Premium）內建的「傳送 HTTP 要求」動作直連 Graph，實測成功：

| 項目 | 內容 |
|---|---|
| 動作名稱 | 傳送 HTTP 要求（Office 365 Outlook 連接器） |
| 方法 | GET |
| URI | `/v1.0/users/{資源信箱}/calendar/events?$filter=start/dateTime ge '{起始}' and start/dateTime le '{結束}'` |
| 白名單物件 | `messages`、`mailFolders`、`events`、`calendar`、`calendars`、`outlook`、`inferenceClassification`（`calendarView` 不可用） |
| ATA-9627 實測結果 | `statusCode 200`，取得真實借用事件（借用人、主旨、起訖時間、是否全天、iCalUId 等） |

這個方法不需要 Calendar ID、不需要 Premium 授權、也不需要另建 Azure AD App。三台車共用同一個動作，只需替換 `{資源信箱}` 與時間區間參數。

`公務車行事曆同步至 SharePoint` 正式流程建議直接採用此方法作為讀取步驟；詳細測試過程見 `docs/ata9627-v3-calendar-event-test.md` 第十四節與 `release-notes/v0.2.11.md`。

下一步：對 Camry（`room_nhb4_car_camry@alp.global`）、Cross（`room_nhb4_car_cross@alp.global`）重複驗證後，建立正式同步流程。

## v0.2.12 更新 - 建立正式公務車行事曆同步至SharePoint流程並啟用排程

2026-07-23：Camry、Cross 重複驗證通過後，建立正式流程 `公務車行事曆同步至SharePoint`（Flow ID `b6d1ec5c-0fc5-46c1-85b0-d8d68c72c0ce`），結構如下：

```text
Recurrence（15 分鐘）
  → 車輛清單（Compose，三台車資源信箱與名稱）
  → 套用至各項（外層，逐台車）
      → 傳送 HTTP 要求（calendar/events + $filter）
      → 剖析 JSON
      → 套用至各項 1（內層，逐筆事件）
          → 編輯（Compose，預約唯一鍵）
          → 編輯 1（Compose，預計通知時間）
          → 取得多個項目（SharePoint，依預約唯一鍵篩選）
          → 條件（篩選筆數 > 0？）
              → 真：更新項目
              → 假：建立項目
```

「更新項目」「建立項目」兩分支皆依 `sharepoint/list-schema.md` 完整寫入約 40 個欄位，含系統防呆欄位預設值；`是否測試資料` 暫維持 `是` 作為上線前安全預設值。

### 排除的設計問題：Apply to each 命名空格轉底線

儲存流程時曾出現：

```text
InvalidTemplate: The template validation failed: 'The repetition action(s) '套用至各項 1' referenced by 'inputs' in action '更新項目' are not defined in the template.'
```

原因：Power Automate 動作／迴圈的顯示名稱若含空格，從**該動作自身作用範圍以外**以名稱參照時（`items(...)`、`outputs(...)`），空格會被轉換為底線。巢狀迴圈 `套用至各項 1` 須寫成 `items('套用至各項_1')`，而非 `items('套用至各項 1')`。此為 Power Automate 設計工具的命名慣例限制，後續建立巢狀迴圈時應直接採用底線形式，避免重複踩坑。

### 驗證與啟用

- 流程儲存驗證零錯誤（「您的流程已準備就緒」）。
- 流程檢查程式（Flow Checker）開啟前僅顯示 1 項警告「此流程已關閉」；點選「開啟此流程」後，狀態變更為「開啟」，錯誤 0、警告 0。
- 流程現為每 15 分鐘自動執行一次，正式進入排程運作階段。

詳細版本紀錄見 `release-notes/v0.2.12.md`。

下一步：執行端到端測試（Outlook 預約 → SharePoint 同步）、P3 情境測試、建立 Concurrency Control，通過後進入「公務車借用前 Teams 通知與回覆」流程開發，並刪除暫存測試流程 `公務車功能測試-郵件動作參數檢查(可刪除)`。

## v0.2.13 更新 - 執行期錯誤修正並完成首次成功執行驗證

2026-07-24：v0.2.12 開啟排程後，運行紀錄（Run History）中所有執行皆顯示失敗，代表「儲存零錯誤」不等於「執行成功」。逐一排查後找出並修正四種執行期錯誤：

1. **兩處 Apply to each 迴圈的 `foreach` 參數誤加大括號**：`"@{items(...)}"` 會被當成字串插值處理、強制轉為字串型別，導致 `InvalidTemplate` 型別不符；修正為不加大括號的原生運算式 `"@items(...)"`。
2. **「取得多個項目」篩選查詢誤用中文欄位顯示名稱**：SharePoint 對非英數欄位顯示名稱會以 Unicode 逐字元編碼產生 `_xNNNN_` 內部名稱（上限 32 字元，超過會截斷），且 OData 查詢時非英數起首的內部名稱前面需再加 `OData_` 前綴。以 REST API `/_api/web/lists(guid'...')/fields?$filter=Title eq '預約唯一鍵'&$select=InternalName,StaticName,EntityPropertyName` 查得正確篩選欄位為 `OData__x9810__x7d04__x552f__x4e00__x93`。
3. **「條件」判斷式 `greater` 比較誤加大括號**：同第 1 項的大括號型別轉換問題，修正為 `"@length(...)"`；此動作屬結構性動作，程式碼檢視為唯讀，須改由「參數」頁籤 UI 編輯。
4. **「是否整天」欄位型別不符**：`if(equals(...),'是','否')` 回傳中文字串，但欄位為布林型別，觸發 `OpenApiOperationParameterTypeConversionFailed`；修正為直接參照原生布林值 `@items('套用至各項_1')?['isAllDay']`。

修正後手動觸發測試，運行紀錄首次出現「測試成功」，並逐層展開執行追蹤確認：外層「套用至各項」（三台車）→「傳送 HTTP 要求」→「剖析 JSON」→ 內層「套用至各項 1」→「編輯」「編輯 1」→「取得多個項目」→「條件」→「建立項目」（本次事件為新資料，條件為假，成功執行；「更新項目」分支正確顯示為略過）皆為成功狀態。另以 SharePoint REST API 直接查詢清單，確認三筆借用紀錄正確寫入且「是否整天」為真正布林值；後續排程再次自動執行後項目數未增加，確認「預約唯一鍵」防重複比對邏輯正常運作。

詳細版本紀錄見 `release-notes/v0.2.13.md`。

下一步：執行 P3 各項情境測試（取消、時間異動、整天借用、重複觸發等），通過後將 `是否測試資料` 切換為正式運作邏輯、建立 Concurrency Control、刪除暫存測試流程，並確認「我的流程」清單中是否有本流程的重複項目需清理。
