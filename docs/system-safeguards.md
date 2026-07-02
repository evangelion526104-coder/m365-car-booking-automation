# 系統防呆設計（System Safeguards）

更新日期：2026-07-02  
適用專案：M365 公務車借用自動通知與後台管理流程  
狀態：SharePoint 防呆欄位已完成；正式 Power Automate 流程與防呆測試仍須於上線前完成。後續架構設計、Teams Adaptive Card、測試案例與文件均需遵守。

## 一、總原則

本專案的防呆設計目標，是避免公務車預約資料因時區、重複觸發、行事曆取消、舊 Teams 卡片或資料比對錯誤，造成錯誤通知、重複通知、重複紀錄或已取消案件被重新啟用。

五項正式上線前必須完成的防呆機制如下：

| 編號 | 防呆項目 | 上線要求 |
|---|---|---|
| SG-01 | 統一時區為 Asia/Taipei | 所有顯示、通知與判斷時間都必須轉為 UTC+8 |
| SG-02 | 行事曆事件 ID 作為唯一識別碼 | 不得只用日期、主旨、姓名或 Email 比對 |
| SG-03 | 取消／異動行事曆同步 SharePoint | Outlook 新增、修改、取消都必須同步 |
| SG-04 | 舊 Teams Adaptive Card 失效機制 | 回覆前必須重新檢查最新狀態 |
| SG-05 | Flow 執行鎖與重複資料防止 | 避免重複建立、重複通知與 Race Condition |

## 二、SG-01 統一時區為 Asia/Taipei

### 設計要求

- 所有 Outlook、Power Automate、SharePoint 日期時間均需統一轉換為 `Asia/Taipei`。
- Power Automate 內建時區名稱建議使用 `Taipei Standard Time`。
- 不得直接使用未轉換的 UTC 原始時間作為通知或後台顯示時間。
- 全天事件不得以 UTC 午夜直接扣 1 小時計算通知，避免變成前一天 23:00。

### 可落地流程

1. Outlook 讀到的開始與結束時間，先保留原始 UTC 欄位。
2. 使用 Power Automate `convertTimeZone()` 轉成台北時間。
3. SharePoint 後台主要顯示欄位使用台北時間。
4. 若 `是否整天 = 是`，直接使用「借用日期 08:00 Asia/Taipei」作為 `預計通知時間`。
5. 若非整天，使用「台北起始時間 - 1 小時」，但不得早於當天 08:00。

### Power Automate 表達式參考

```text
convertTimeZone(<OutlookStartDateTime>, 'UTC', 'Taipei Standard Time')
```

通知時間判斷概念：

```text
若 是否整天 = true：
  預計通知時間 = 借用日期 08:00 Asia/Taipei
否則：
  預計通知時間 = max(借用起始時間台北時間 - 1小時, 借用日期 08:00 Asia/Taipei)
```

## 三、SG-02 行事曆事件 ID 作為唯一識別碼

### 設計要求

- 每筆公務車借用紀錄必須以 Outlook `Event ID` 或 `iCalUId` 作為唯一識別碼。
- 建議實務唯一鍵為：`資源信箱 + Event ID`。
- 若 Event ID 在特定情境改變，使用 `資源信箱 + iCalUId` 作為輔助比對。
- 後續更新、取消、重新通知、Adaptive Card 回覆、SharePoint 狀態更新，都必須以該 ID 比對。
- 不得只用日期、主旨、借用人姓名或 Email 作為唯一比對依據。

### 可落地流程

1. Power Automate 讀取 Outlook 事件時，取得 `Event ID`、`iCalUId`、`Last Modified Time`。
2. 組合 `預約唯一鍵 = 資源信箱 + '|' + Event ID`。
3. 查詢 SharePoint：`預約唯一鍵` 是否已存在。
4. 若不存在，新增 SharePoint 項目。
5. 若已存在，更新同一筆 SharePoint 項目。
6. 若查到多筆，標記為重複資料異常，停止通知並通知承辦人檢查。

## 四、SG-03 取消／異動行事曆同步 SharePoint

### 設計要求

- 需設計 Outlook 行事曆新增、修改、取消的同步邏輯。
- 當時間、車輛、主旨、借用人等資訊異動時，SharePoint 必須同步更新。
- 當 Outlook 預約取消時，SharePoint `領鑰狀態` 需更新為 `已取消`。
- 已取消資料必須停止或失效後續 Teams 通知與提醒流程。

### 可落地流程

1. 同步流程每次讀取未來一段期間的資源行事曆。
2. 對每個 Outlook 事件用 `預約唯一鍵` 查詢 SharePoint。
3. 新事件：新增 SharePoint 項目，狀態為 `待通知`。
4. 已存在事件：比較時間、車輛、主旨、借用人、Last Modified Time。
5. 若資料不同，更新 SharePoint，重算 `預計通知時間`，並將舊卡片版本標示為失效。
6. 對 SharePoint 中仍為有效狀態，但本次 Outlook 查不到的未來事件，更新為 `已取消` 或 `已失效`。
7. 已取消資料不再發送 Teams 通知，也不得被舊卡片回覆重新啟用。

## 五、SG-04 舊 Teams Adaptive Card 失效機制

### 設計要求

借用人送出 Adaptive Card 回覆前，系統需重新檢查 SharePoint 中該筆 Event ID 的最新狀態。

若狀態已為以下任一情況，系統不得更新舊資料：

- `已取消`
- `已完成確認`
- `已領鑰`
- `已失效`
- Event ID 不一致
- 卡片版本不是最新版本
- 事件內容已異動

需回覆提示：

```text
此借用已取消或已變更，請重新確認最新借用資訊。
```

### 可落地流程

1. Adaptive Card 發送時帶入 `sharePointItemId`、`eventId`、`iCalUId`、`cardVersion`、`eventLastModifiedTime`。
2. 使用者按下送出後，Power Automate 先 `Get item` 取得 SharePoint 最新資料。
3. 檢查 SharePoint 中的 `行事曆事件 ID` 是否等於卡片送回的 `eventId`。
4. 檢查 SharePoint 中的 `卡片版本` 是否等於卡片送回的 `cardVersion`。
5. 檢查 `領鑰狀態` 是否仍允許回覆。
6. 檢查 `事件最後修改時間` 是否與卡片送出時相同。
7. 全部通過才更新共乘人數、規範確認狀態與 Teams 回覆時間。
8. 任一條件不通過，停止更新並回覆失效提示。

## 六、SG-05 Flow 執行鎖與重複資料防止

### 設計要求

需避免 Outlook 同一事件短時間內多次觸發，造成重複建立或重複通知。

Power Automate 必須設計：

- 以 Event ID 查詢既有 SharePoint 紀錄。
- 已存在則更新，不存在才新增。
- 啟用 Concurrency Control，避免同一流程同時處理多筆造成 Race Condition。
- 避免重複發送 Teams 通知。
- 避免建立重複 SharePoint 紀錄。

### 可落地流程

1. 同步流程與通知流程均啟用 Concurrency Control。
2. 同步流程建議 Degree of Parallelism 設為 `1`。
3. 每次處理前，以 `預約唯一鍵` 查詢 SharePoint。
4. 若查無資料才新增。
5. 若已存在一筆，更新該筆。
6. 若查到多筆，將重複資料標記為異常，不發通知。
7. Teams 通知流程發送前檢查 `通知發送時間`、`通知狀態`、`最新通知批次 ID`。
8. 已通知且卡片仍有效時，不重複發送。
9. 行事曆異動後需產生新 `卡片版本`，舊卡片改為 `已失效`。

## 七、SharePoint List 已建立防呆欄位

以下欄位已於 2026-07-02 在正式 SharePoint 清單建立並完成畫面驗證，後續 Power Automate 流程需正確寫入與維護。

| 欄位名稱 | 類型 | 用途 |
|---|---|---|
| `iCalUId` | 單行文字 | Outlook 事件輔助唯一識別碼 |
| `唯一識別來源` | 選擇 | Event ID / iCalUId / 手動 |
| `事件最後修改時間` | 日期和時間 | Outlook 事件最後異動時間，使用台北時間顯示 |
| `原始開始時間 UTC` | 日期和時間 | 保留 Outlook 原始 UTC 開始時間供稽核 |
| `原始結束時間 UTC` | 日期和時間 | 保留 Outlook 原始 UTC 結束時間供稽核 |
| `顯示時區` | 單行文字 | 固定填入 `Asia/Taipei` |
| `事件同步狀態` | 選擇 | 有效 / 已異動 / 已取消 / 已失效 / 重複異常 |
| `取消時間` | 日期和時間 | Outlook 取消或同步判定取消的時間 |
| `卡片版本` | 數字 | 每次重新通知或事件異動時遞增 |
| `卡片狀態` | 選擇 | 未發送 / 有效 / 已回覆 / 已失效 |
| `最新通知批次 ID` | 單行文字 | 避免重複通知 |
| `通知狀態` | 選擇 | 未通知 / 已通知 / 不需通知 / 通知失敗 |
| `Flow 執行 ID` | 單行文字 | 記錄最近一次處理此筆資料的 Flow Run ID |
| `處理鎖定狀態` | 選擇 | 未鎖定 / 處理中 / 已完成 / 異常 |
| `處理鎖定時間` | 日期和時間 | 避免長時間鎖定無法處理 |
| `重複資料檢查結果` | 選擇 | 正常 / 疑似重複 / 已排除 |
| `防呆檢查結果` | 多行文字 | 記錄回覆前檢查或流程防呆判斷結果 |
| `已取消` | 是/否 | 快速判斷案件是否已取消 |
| `已失效` | 是/否 | 快速判斷舊資料或舊卡片是否已失效 |

`領鑰狀態` 建議新增或統一使用以下狀態：

- `待通知`
- `未完成填寫`
- `已完成確認`
- `已領鑰`
- `已取消`
- `已失效`

若現有狀態仍有 `取消`，正式上線前應統一改為 `已取消`，避免狀態名稱不一致。

## 八、測試案例表

| 測試編號 | 測試項目 | 測試方式 | 預期結果 |
|---|---|---|---|
| SG-T01 | 時區轉換測試 | 建立一般預約，檢查 Outlook UTC 與 SharePoint 顯示時間 | SharePoint 顯示為 Asia/Taipei，不直接顯示 UTC |
| SG-T02 | 全天事件測試 | 建立整天公務車預約 | `是否整天 = 是`，`預計通知時間 = 當天 08:00` |
| SG-T03 | 全天事件避免前一天通知 | 建立整天預約並檢查通知計算 | 不得產生前一天 23:00 或錯誤日期通知 |
| SG-T04 | 行事曆取消測試 | 取消 Outlook 預約 | SharePoint 更新為 `已取消`，Teams 通知停止 |
| SG-T05 | 行事曆異動測試 | 修改時間、主旨或借用人 | SharePoint 同步更新，重算通知時間，舊卡片失效 |
| SG-T06 | 舊卡片送出測試 | 使用者送出事件已取消或已異動的舊卡片 | 不更新資料，回覆「此借用已取消或已變更，請重新確認最新借用資訊。」 |
| SG-T07 | 重複觸發測試 | 同一 Outlook 事件短時間內被同步兩次 | 不建立重複 SharePoint 紀錄，不重複發送 Teams |
| SG-T08 | 重複資料異常測試 | 人為建立相同 Event ID 的兩筆資料 | 標記重複異常，不發通知，提醒承辦人檢查 |
| SG-T09 | 回覆前狀態檢查 | 已完成或已領鑰後再次送出卡片 | 不覆蓋既有資料 |
| SG-T10 | Flow Concurrency 測試 | 同時間大量事件同步 | 流程依序處理，不發生 Race Condition |

## 九、正式上線前驗收標準

五項防呆設計必須全部通過測試，才可正式上線：

- [ ] 所有日期時間已轉為 Asia/Taipei。
- [ ] Event ID / iCalUId 唯一識別邏輯已完成。
- [ ] 行事曆新增、修改、取消同步 SharePoint 已完成。
- [ ] 舊 Adaptive Card 回覆前檢查已完成。
- [ ] Flow Concurrency Control 與重複資料防止已完成。
- [ ] 測試案例 SG-T01 至 SG-T10 已通過。
