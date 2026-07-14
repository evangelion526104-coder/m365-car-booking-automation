# ATA-9627 Resource Calendar V3 事件讀取測試

版本：v0.2.7  
建立日期：2026-07-14  
測試目標：確認 Power Automate 是否可以透過 Office 365 Outlook 標準連接器，直接讀取 ATA-9627 公務車 Resource Mailbox 的行事曆事件。

## 一、測試背景

本專案流程執行帳號為：

`ad.general@alp.global`

本次先以單一公務車測試：

| 項目 | 內容 |
|---|---|
| 公務車名稱 | 公務車Altis ATA-9627 B4-16-永聯內湖辦公室 |
| Resource Mailbox | room_nhb4_car@alp.global |
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
| 2 | 取得事件的行事曆檢視 (V3) | Calendar Id 先填 `room_nhb4_car@alp.global` |
| 3 | 篩選陣列 | 篩選 Subject 等於 `TEST` 的事件 |
| 4 | Compose - 事件數量 | 顯示抓到幾筆事件 |
| 5 | Compose - 第一筆事件摘要 | 顯示主旨、開始時間、結束時間、Event ID、iCalUId |
| 6 | 條件判斷 | 若事件數量大於 0，判定讀取成功 |

## 五、V3 動作建議設定

動作名稱：

`取得事件的行事曆檢視 (V3)`

Calendar Id 測試順序：

1. 第一優先：直接填入 `room_nhb4_car@alp.global`
2. 若失敗：記錄完整錯誤訊息，確認是否為 Calendar Id 格式限制。
3. 若錯誤指出 Calendar Id 不存在或無權存取：請 IT 協助確認 Office 365 Outlook 連接器是否支援以 Resource Mailbox SMTP 地址作為 Calendar Id。
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
| 取得事件的行事曆檢視 (V3) 是否可讀取 TEST | 待測 | 下一階段測試 |
| Calendar Id 最終格式 | 待確認 | 第一輪先使用 `room_nhb4_car@alp.global` |
| 是否可取得 Event ID / iCalUId | 待測 | 成功讀到事件後確認 |

## 十、測試完成後需回填

完成 Power Automate 測試後，請回填：

1. 實際使用的 Power Automate 動作名稱。
2. Calendar Id 最終填入內容。
3. 開始時間與結束時間設定。
4. 是否成功讀到 `TEST`。
5. 若失敗，完整錯誤訊息。
6. 問題屬於 Calendar Id、連接器限制、快取、權限或其他原因。
7. 下一步建議。

