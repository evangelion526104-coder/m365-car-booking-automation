# ATA-9627 Resource Calendar V3 事件讀取測試

版本：v0.2.8
建立日期：2026-07-14
最近更新：2026-07-15
測試目標：確認 Power Automate 是否可以透過 Office 365 Outlook 標準連接器，直接讀取 ATA-9627 公務車 Resource Mailbox 的行事曆事件。

## 一、測試背景

本專案流程執行帳號為：

`ad.general@alp.global`

本次先以單一公務車測試：

| 項目 | 內容 |
|---|---|
| 公務車名稱 | 公務車Altis ATA-9627 B4-16-永聯內湖辦公室 |
| Resource Mailbox | room_nhb4_car@alp.global |
| Calendar ID | `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |
| 測試事件主旨 | TEST |
| 目標連接器 | Office 365 Outlook |
| 是否使用 Premium 連接器 | 否 |

## 二、目前已確認事項

IT 已將 `ad.general@alp.global` 對 `room_nhb4_car@alp.global` 的 Calendar Folder Permission 調整為 Editor。

Outlook 端已實際驗證：

1. `ad.general@alp.global` 可以在 Outlook 中看到 ATA-9627 行事曆。
2. 可以透過「新增會議室或位置」預約 ATA-9627。
3. 可以新增行事曆事件。
4. 可以修改事件。
5. 可以刪除事件。

因此可確認 Outlook 端 Editor 權限已生效。

## 三、已知 Power Automate 測試結果

既有測試流程：

`公務車功能測試-資源信箱讀取`

流程內容：

1. Recurrence
2. 取得行事曆 (V2)

Office 365 Outlook 連線帳號：

`ad.general@alp.global`

執行結果：

| 項目 | 結果 |
|---|---|
| Flow 執行 | 成功 |
| statusCode | 200 |
| value.0.name | Calendar |
| value.0.owner.name | AD General 總機專用 |
| value.0.owner.address | ad.general@alp.global |
| 是否列出 room_nhb4_car@alp.global | 否 |
| 是否列出 ATA-9627 行事曆名稱 | 否 |

目前判斷：

`取得行事曆 (V2)` 只列出登入帳號本身的 Calendar，不能作為 Resource Mailbox 是否可讀取的最終判斷依據。

下一步需改用 `取得事件的行事曆檢視 (V3)` 直接測試是否能讀取 Resource Mailbox 事件。

## 四、建議建立的獨立測試流程

流程名稱：

`公務車功能測試-ATA9627事件讀取`

流程用途：

直接驗證 Power Automate 是否可以讀取 `room_nhb4_car@alp.global` 的行事曆事件，並確認是否能抓到主旨為 `TEST` 的測試事件。

### 流程動作

| 順序 | Power Automate 動作 | 設定重點 |
|---|---|---|
| 1 | 手動觸發流程 | 方便行政或 IT 人員重複測試 |
| 2 | 取得事件的行事曆檢視 (V3) | Calendar Id 填 `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |
| 3 | 篩選陣列 | 篩選 Subject 等於 `TEST` 的事件 |
| 4 | Compose - 事件數量 | 顯示抓到幾筆事件 |
| 5 | Compose - 第一筆事件摘要 | 顯示主旨、開始時間、結束時間、Event ID、iCalUId |
| 6 | 條件判斷 | 若事件數量大於 0，判定讀取成功 |

## 五、V3 動作建議設定

動作名稱：

`取得事件的行事曆檢視 (V3)`

Calendar Id 測試順序：

1. 歷史測試值：`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`；2026-07-18 已證實不適用 V3，不得再使用。
2. 不再使用 `room_nhb4_car@alp.global` 作為 V3 Calendar Id；先前已確認信箱地址格式不適用。
3. 若 GUID 仍失敗：記錄完整錯誤訊息，確認是否為權限、連接器限制、快取或 Calendar ID 對應問題。
4. 若標準連接器無法直接讀取：需評估替代方案，但仍以不使用 Premium 連接器為優先。

開始時間：

```text
utcNow()
```

結束時間：

```text
addDays(utcNow(),7)
```

時區原則：

所有文件、SharePoint 顯示與後續通知時間，一律以 `Asia/Taipei` 為準。  
若 V3 動作要求 UTC 格式，Power Automate 內部可用 UTC 查詢，但輸出給人員確認時必須轉換為台北時間。

## 六、建議輸出欄位

請在 Compose 或測試結果中確認是否可取得：

| 欄位 | 用途 |
|---|---|
| subject | 使用事由或測試主旨 |
| start | 借用起始時間 |
| end | 借用結束時間 |
| isAllDay | 是否全天事件 |
| organizer / creator | 建立者或借用人資訊 |
| id | Outlook Event ID |
| iCalUId | 跨流程比對用唯一識別碼 |

## 七、成功判斷

若流程輸出中可以看到：

```text
subject = TEST
```

或能看到 ATA-9627 行事曆中的其他事件資料，即判定 Resource Mailbox 事件讀取成功。

成功後，下一步可將此 Calendar Id 設定方式套用到三台車，並進入 SharePoint 同步流程測試。

## 八、失敗判斷與原因分類

若讀取失敗，請依錯誤訊息分類：

| 類型 | 判斷方式 | 後續處理 |
|---|---|---|
| Calendar Id 問題 | 找不到指定 Calendar Id | 測試其他 Calendar Id 格式，並請 IT 確認連接器支援方式 |
| 連接器限制 | Outlook 可看見，但 Power Automate V3 仍不能讀 | 評估替代方案，例如 Outlook 轉寄/同步、SharePoint 手動登錄輔助或 Graph 權限方案 |
| 快取問題 | 權限剛調整後仍讀不到 | 等待一段時間、重新整理連線、重新登入連接器後再測 |
| 權限問題 | 錯誤明確顯示 Access Denied | 請 IT 重新確認 Calendar Folder Permission 與共享狀態 |
| 測試期間問題 | 查詢區間未涵蓋 TEST | 調整起訖時間，確認 TEST 事件位於未來 7 天內 |

## 九、測試紀錄表

| 測試項目 | 結果 | 備註 |
|---|---|---|
| Outlook 是否可看到 ATA-9627 | 通過 | Editor 權限已在 Outlook 端生效 |
| Outlook 是否可新增 TEST 事件 | 通過 | 已建立測試事件 |
| 取得行事曆 (V2) 是否列出 ATA-9627 | 未通過 | 僅列出 ad.general 自身 Calendar |
| 取得事件的行事曆檢視 (V3) 是否可讀取 TEST | 未通過 | `ErrorInvalidIdMalformed` |
| Calendar Id 最終格式 | 候選 GUID 不適用 V3 | `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |
| 是否可取得 Event ID / iCalUId | 阻擋 | V3 未取得事件 |

## 十、測試完成後需回填

完成 Power Automate 測試後，請回填：

1. 實際使用的 Power Automate 動作名稱。
2. Calendar Id 最終填入內容。
3. 開始時間與結束時間設定。
4. 是否成功讀到 `TEST`。
5. 若失敗，完整錯誤訊息。
6. 問題屬於 Calendar Id、連接器限制、快取、權限或其他原因。
7. 下一步建議。

## 十一、v0.2.8 補充紀錄 - ATA-9627 Calendar ID 已取得

更新日期：2026-07-15

當時取得並視為 ATA-9627 Calendar ID 的候選 GUID：

`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`

請在 Power Automate 測試流程 `公務車功能測試-ATA9627事件讀取` 的 `取得事件的行事曆檢視 (V3)` 動作中，將 Calendar Id 改填上述 GUID。

本階段尚未完成 V3 實際執行驗證，因此不得將 CAL-V3-06、CAL-V3-07 或 M5 標記為通過。需等 Flow 執行輸出確認可取得事件資料後，再更新測試結果。

## 十二、v0.2.9 補充紀錄 - V3 測試實作版已建立

更新日期：2026-07-15

已新增 Power Automate 實作版文件：

`power-automate/ata9627-v3-test-implementation.md`

本次補強內容：

1. 明確指定測試流程名稱：`公務車功能測試-ATA9627事件讀取`。
2. 明確指定 V3 動作命名：`Get_ATA9627_Events_V3`。
3. 明確指定 Calendar Id：`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`。
4. 明確指定查詢期間：`utcNow()` 到 `addDays(utcNow(),7)`。
5. 新增 `Filter_TEST_Events`、`Compose_All_Event_Count`、`Compose_TEST_Event_Count`、`Compose_First_Event_Raw`、`Select_Event_Field_Check` 等測試動作。
6. 定義成功、部分成功、失敗的判斷方式。
7. 明確規定 V3 成功前不得串接正式 SharePoint 同步流程。

目前狀態：

| 項目 | 狀態 |
|---|---|
| ATA-9627 Calendar ID | 候選 GUID 已實測不適用 |
| Power Automate V3 測試實作版 | 已完成 |
| V3 實際執行 | 已執行，未通過 |
| Event ID / iCalUId 實際取得 | 阻擋 |
| 是否可進入 SharePoint 同步 | 不可 |

## 十三、2026-07-18 V3 候選 GUID 實測結果

### 實際執行

| 項目 | 值 |
|---|---|
| Flow | `公務車功能測試-ATA9627事件讀取` |
| Flow ID | `7d66adc8-eccd-4cc0-9ab1-a031a6676df9` |
| Run ID | `08584172227294871773330429620CU04` |
| 開始／結束時間 | 2026-07-18 22:15:56／22:15:57（Asia/Taipei） |
| Calendar Id | `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` |
| startDateTimeUtc | `2026-07-14T00:00:00Z` |
| endDateTimeUtc | `2026-07-21T23:59:59Z` |
| HTTP | `400 BadRequest` |
| 錯誤碼 | `ErrorInvalidIdMalformed` |
| 原始訊息 | `The Id is invalid.` |
| 中文訊息 | `ID 格式不正確。` |
| clientRequestId | `6edfd333-e4b0-4d2d-a3e7-46ced3ee4270` |
| serviceRequestId | `8db0eba7-f781-4dd8-aacc-d7e4d81d3c70` |
| Action tracking ID | `5f715fa9-1108-48fd-82c7-0b862ca71a51` |

### 判定

1. 執行輸入已證明候選 GUID 確實送達 Office 365 Outlook V3 連接器。
2. 此 GUID 不是連接器可接受的 Calendar ID；先前「真正 Calendar ID」的假設已被實測推翻。
3. 本次錯誤不是 Access Denied，也不是查詢期間未涵蓋 `TEST`。
4. CAL-V3-06 未通過；CAL-V3-07 與 CAL-V3-08 因未取得事件而阻擋。
5. 尚不可進入 SharePoint 同步 MVP。
6. 測試 Flow 目前為每分鐘排程且仍顯示「開啟」，需優先人工停用後再進行下一輪受控測試。

### 下一步

- 從 Office 365 Outlook 連接器可列舉的行事曆、重新建立的標準連線或 IT 可提供的 Exchange/Outlook Folder ID 取得可接受值。
- 若標準連接器仍無法讀取 Resource Calendar，評估 O365 E1 可行且不使用 Premium 的替代方案。
- 僅在成功取得事件與 `id`／`iCalUId` 後，才建立 `公務車行事曆同步至 SharePoint - ATA9627 MVP`。

## 十四、v0.2.11 補充紀錄 - Full Access 委派後重測與 HTTP 直連 Graph 成功

更新日期：2026-07-23

### Full Access 委派後重測既有動作

IT 已將 `ad.general@alp.global` 對 `room_nhb4_car@alp.global` 的權限由 Calendar Folder Editor 提升為 Mailbox Full Access。重新執行 `公務車功能測試-ATA9627事件讀取`：

| 動作 | 結果 |
|---|---|
| 取得行事曆 (V2) | 仍只回傳 `ad.general@alp.global` 自身 `Calendar`，未列出 Resource Mailbox |
| 取得事件的行事曆檢視 (V3) | 改用格式正確的 Exchange Item ID 測試，錯誤由 `ErrorInvalidIdMalformed` 變為 `404 ErrorItemNotFound`（`The specified object was not found in the store.`） |
| V3 完整參數檢查 | 確認僅有 3 個必填欄位＋5 個進階參數（篩選查詢、排序依據、最高計數、略過計數、搜尋），無任何「指定其他信箱」欄位 |

判定：`取得行事曆 (V2)` 與 `取得事件的行事曆檢視 (V3)` 兩者在架構上僅能存取連線帳號自己的行事曆，Full Access 委派無法解除此限制，屬於連接器動作本身設計限制，不是權限問題。

附帶完成：確認測試 Flow 狀態已由「開啟／每分鐘排程」變更為「關閉」，解除 v0.2.10 遺留待辦。

### 驗證 Full Access 對郵件類動作有效

建立暫存測試流程 `公務車功能測試-郵件動作參數檢查(可刪除)`：

- `取得電子郵件 (V3)` 動作進階參數中的 `原始信箱地址`（mailboxAddress）填入 `room_nhb4_car@alp.global`，`statusCode 200`，成功讀到該資源信箱 Inbox 真實郵件。
- 證實 Full Access 委派對「郵件類」動作有效，限制僅發生在「行事曆類」動作。

### 正式解法：傳送 HTTP 要求直連 Graph

Office 365 Outlook 標準連接器（非 Premium）內建「傳送 HTTP 要求」動作，可直接呼叫 Graph 端點：

1. 第一次嘗試 `GET /v1.0/users/room_nhb4_car@alp.global/calendarView?startDateTime=...&endDateTime=...` 失敗，回傳 `BadRequest`：

   ```text
   URI path is not a valid Graph endpoint, path is neither absolute nor relative or resource/object is not supported for this connector.
   Invalid resource,Allowed values: me,users.
   Invalid Object,Allowed values: messages,mailFolders,events,calendar,calendars,outlook,inferenceClassification.
   ```

2. 改用白名單內的 `calendar/events` 物件，並以 `$filter` 篩選時間區間：

   ```text
   GET /v1.0/users/room_nhb4_car@alp.global/calendar/events?$filter=start/dateTime ge '2026-07-14T00:00:00' and start/dateTime le '2026-07-30T23:59:59'
   ```

3. 執行結果：`statusCode 200`，成功取得 ATA-9627 真實借用紀錄，包含 `subject`、`start.dateTime`、`end.dateTime`、`isAllDay`、`organizer.emailAddress.name`／`address`、`id`、`iCalUId`、`lastModifiedDateTime`、`location.displayName`、`recurrence` 等完整欄位。實際取得的一筆真實事件範例：借用人 `daniel.yen@alp.global`，主旨「瑞芳參訪」，2026-07-24 借用。

### 判定

- M5 Resource Calendar 事件讀取正式解除阻擋，判定為**通過**。
- 不需要 Premium 授權、不需要另建 Azure AD App 或 Logic App。
- 可以正式進入 `公務車行事曆同步至 SharePoint` 流程建置階段。

### 已知限制

1. Graph 端點僅限白名單物件，`calendarView` 不可用，一律改用 `calendar/events` + `$filter`。
2. 重複事件（recurring series）目前僅回傳 `seriesMaster`，尚未展開個別發生實例；MVP 階段先涵蓋單次與系列主體事件，個別實例展開留待後續評估。
3. 目前僅以 ATA-9627 驗證，Camry、Cross 尚未逐一重測。
4. 回傳時間為 UTC，寫入 SharePoint 時仍須依既有防呆規則轉換為 `Asia/Taipei`。

### 下一步

1. 對 Camry（`room_nhb4_car_camry@alp.global`）、Cross（`room_nhb4_car_cross@alp.global`）重複相同的 HTTP 要求驗證。
2. 建立正式流程 `公務車行事曆同步至 SharePoint`，三台車共用同一組 HTTP 要求＋解析＋寫入邏輯。
3. 依 `預約唯一鍵`（資源信箱 + 行事曆事件 ID）判斷新增或更新 SharePoint 項目，並計算 `是否整天`、`預計通知時間`。
