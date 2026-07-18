# ATA-9627 V3 事件讀取測試實作版

版本：v0.2.9  
更新日期：2026-07-15  
適用流程：`公務車功能測試-ATA9627事件讀取`  
測試車輛：公務車Altis ATA-9627 B4-16-永聯內湖辦公室  
Resource Mailbox：`room_nhb4_car@alp.global`  
Calendar ID：`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`

## 一、測試目的

本測試原用來確認 Power Automate 的 Office 365 Outlook 標準連接器，是否可以用 ATA-9627 的候選 Calendar ID 讀取未來 7 天行事曆事件。

若本測試成功，才可進入下一階段：

1. 取得 Camry 與 Cross 的 Calendar ID。
2. 建立正式 `公務車行事曆同步至 SharePoint` 流程。
3. 將 Outlook 預約資料寫入 `公務車借用管理` SharePoint List。

若本測試失敗，暫不串接正式 SharePoint 同步，避免建立錯誤或重複資料。

## 二、測試前確認

| 項目 | 必須確認 |
|---|---|
| Power Automate 連線帳號 | `ad.general@alp.global` |
| 連接器 | Office 365 Outlook 標準連接器 |
| 測試事件 | ATA-9627 行事曆中建議保留一筆未來 7 天內、主旨為 `TEST` 的事件 |
| Premium 連接器 | 不使用 |
| Exchange Administrator | 不需要 |

## 三、建議流程動作

請建立或開啟流程：

`公務車功能測試-ATA9627事件讀取`

### 動作 1：手動觸發流程

使用：

`Manually trigger a flow` / `手動觸發流程`

用途：方便行政或 IT 人員重複按「測試」。

### 動作 2：取得事件的行事曆檢視 (V3)

建議將動作重新命名為：

`Get_ATA9627_Events_V3`

設定：

| 欄位 | 值 |
|---|---|
| Calendar Id | `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |
| Start time | `utcNow()` |
| End time | `addDays(utcNow(),7)` |

注意：

- Calendar Id 請填 GUID，不要填 `room_nhb4_car@alp.global`。
- 查詢可以使用 UTC，但文件、SharePoint 後台與通知判斷必須轉換為 `Asia/Taipei`。

### 動作 3：篩選 TEST 事件

新增：

`Filter array` / `篩選陣列`

建議命名：

`Filter_TEST_Events`

From 請填：

```text
body('Get_ATA9627_Events_V3')?['value']
```

進階模式請填：

```text
@equals(item()?['subject'], 'TEST')
```

用途：確認是否讀到測試事件。

### 動作 4：全部事件數量

新增：

`Compose`

建議命名：

`Compose_All_Event_Count`

Expression：

```text
length(body('Get_ATA9627_Events_V3')?['value'])
```

### 動作 5：TEST 事件數量

新增：

`Compose`

建議命名：

`Compose_TEST_Event_Count`

Expression：

```text
length(body('Filter_TEST_Events'))
```

### 動作 6：第一筆事件原始資料

新增：

`Compose`

建議命名：

`Compose_First_Event_Raw`

Expression：

```text
if(greater(length(body('Get_ATA9627_Events_V3')?['value']),0),first(body('Get_ATA9627_Events_V3')?['value']),json('{}'))
```

用途：直接檢查 Power Automate 是否有取得 `subject`、`start`、`end`、`isAllDay`、`id`、`iCalUId` 等欄位。

### 動作 7：欄位摘要

新增：

`Select` / `選取`

建議命名：

`Select_Event_Field_Check`

From：

```text
body('Get_ATA9627_Events_V3')?['value']
```

欄位對照：

| Key | Value Expression |
|---|---|
| subject | `item()?['subject']` |
| start | `item()?['start']` |
| end | `item()?['end']` |
| isAllDay | `item()?['isAllDay']` |
| organizer | `item()?['organizer']` |
| createdBy | `item()?['createdBy']` |
| id | `item()?['id']` |
| iCalUId | `item()?['iCalUId']` |

## 四、成功判斷

| 測試結果 | 判斷 | 下一步 |
|---|---|---|
| `Compose_TEST_Event_Count` 大於 0 | 成功讀到 TEST | 可進入 Camry / Cross Calendar ID 測試 |
| `Compose_All_Event_Count` 大於 0，但 TEST 為 0 | 可讀取行事曆，但未找到 TEST | 確認 TEST 是否在未來 7 天內，或改用實際事件驗證欄位 |
| 全部事件數量為 0 | 尚不能證明成功或失敗 | 建立未來 7 天內 TEST 事件後重跑 |
| V3 回傳 Access Denied | 權限或連接器存取問題 | 請 IT 檢查 Calendar Folder Permission 與連線帳號 |
| V3 回傳 ID 格式不正確 | Calendar ID 不適用或填錯 | 確認是否填入 `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |

## 五、必須截取或貼回的測試輸出

測試後請記錄：

1. Flow 執行結果是成功或失敗。
2. `Compose_All_Event_Count` 的值。
3. `Compose_TEST_Event_Count` 的值。
4. `Compose_First_Event_Raw` 是否包含：
   - `subject`
   - `start`
   - `end`
   - `isAllDay`
   - `id`
   - `iCalUId`
5. 若失敗，完整錯誤訊息。

## 六、是否可以進入下一步流程

只有在以下條件都成立時，才可進入正式 SharePoint 同步流程：

- V3 動作執行成功。
- 可取得 ATA-9627 至少一筆事件，或可確認 `TEST` 事件。
- 可取得 `id` 或 `iCalUId`。
- 可取得 `start` 與 `end`。
- 時間欄位後續可轉換為 `Asia/Taipei`。

符合以上條件後，下一步建立 ATA-9627 MVP 同步流程：

`公務車行事曆同步至 SharePoint - ATA9627 MVP`

MVP 同步流程只先處理 ATA-9627，不直接擴大到三台車。等 ATA-9627 從 Outlook 到 SharePoint 的新增、更新、取消與防重複都測過，再套用到 Camry 與 Cross。

## 七、ATA-9627 MVP 同步流程草案

成功讀取 V3 後，下一條流程建議如下：

| 順序 | 動作 | 說明 |
|---|---|---|
| 1 | 排程觸發 | 每 15 或 30 分鐘執行 |
| 2 | Get_ATA9627_Events_V3 | 讀取 ATA-9627 未來固定期間事件 |
| 3 | Apply to each | 逐筆處理 Outlook 事件 |
| 4 | Compose - 預約唯一鍵 | `room_nhb4_car@alp.global` + `|` + Event ID |
| 5 | SharePoint 取得項目 | 依預約唯一鍵查詢是否已存在 |
| 6 | 條件判斷 | 已存在則更新，不存在才新增 |
| 7 | 時區轉換 | 將時間轉為 `Asia/Taipei` |
| 8 | 通知時間計算 | 一般借用前 1 小時，但最早 08:00；整天借用 08:00 |
| 9 | 防重複欄位寫入 | 寫入 Event ID、iCalUId、Flow 執行 ID、處理鎖定狀態 |

本 MVP 流程仍需遵守五項系統防呆設計，不得省略。

## 八、2026-07-18 實測回填

實際 Flow 目前為已排程流程，V3 動作仍使用畫面顯示名稱 `取得事件的行事曆檢視 (V3)`。執行紀錄 `08584172227294871773330429620CU04` 的輸入已確認：

```text
calendarId = 6049e1d1-b34c-4cca-b530-2c7c4b77abe9
startDateTimeUtc = 2026-07-14T00:00:00Z
endDateTimeUtc = 2026-07-21T23:59:59Z
```

結果：

```text
HTTP 400 BadRequest
ErrorInvalidIdMalformed
The Id is invalid.
```

因此文件中的候選 GUID 假設已被實測推翻。下一輪不得重複使用此值；必須先取得 V3 連接器可接受的 Calendar ID。測試流程仍顯示「開啟」且每分鐘執行，需人工停用後才進行下一輪受控測試。
