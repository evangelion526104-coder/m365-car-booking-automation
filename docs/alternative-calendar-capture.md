# 替代方案：邀請流程信箱模式

更新日期：2026-07-04

## 一、採用原因

目前 `ad.general@alp.global` 已可在 Outlook 網頁看到三台公務車共用行事曆，但 Power Automate 的 `取得行事曆 (V2)` 仍只回傳流程信箱自己的個人 `Calendar`，未列出三台 Resource Mailbox。

為避免等待 Exchange 權限調整，且維持 O365 E1、標準連接器、不使用 Premium 的限制，本專案先採用「邀請流程信箱模式」作為替代方案。

## 二、替代方案核心概念

員工仍照原本方式在 Outlook 預約公務車 Resource Mailbox，但預約時必須同步邀請：

```text
ad.general@alp.global
```

Power Automate 不再直接讀三台 Resource Mailbox 行事曆，而是讀取 `ad.general@alp.global` 自己的行事曆事件。

只要事件參與者中包含以下任一公務車資源信箱，即判定為公務車借用事件：

| 車輛 | Resource Mailbox |
|---|---|
| 公務車 Altis ATA-9627 B4-16 | `room_nhb4_car@alp.global` |
| 公務車 Camry BKX-2370 B4-17 | `room_nhb4_car_camry@alp.global` |
| 公務車 Cross BKY-0762 B4-44 | `room_nhb4_car_cross@alp.global` |

## 三、使用者預約規則

員工預約公務車時，Outlook 邀請對象需包含：

1. 對應公務車 Resource Mailbox。
2. 流程信箱 `ad.general@alp.global`。

範例：

```text
收件者 / 地點：
room_nhb4_car@alp.global
ad.general@alp.global
```

若未邀請 `ad.general@alp.global`，Power Automate 將無法自動收到該筆事件，也就無法發送 Teams Adaptive Card 或更新 SharePoint 後台。

## 四、Power Automate 設計

### 流程一：流程信箱行事曆同步至 SharePoint

目的：讀取 `ad.general@alp.global` 自己行事曆中的公務車借用事件，寫入或更新 SharePoint。

建議作法：

1. 使用 Office 365 Outlook 標準連接器。
2. 讀取 `ad.general@alp.global` 的個人 `Calendar`。
3. 查詢未來一段時間的事件，例如未來 14 天或 30 天。
4. 檢查事件參與者或地點是否包含三台公務車 Resource Mailbox。
5. 若不包含公務車資源信箱，略過。
6. 若包含公務車資源信箱，依對應 Email 判斷車輛名稱。
7. 寫入 SharePoint `公務車借用管理`。
8. 以 `iCalUId` 或 `ad.general@alp.global` 行事曆事件 ID 作為唯一識別。
9. 已存在則更新，不存在才新增。
10. 依 `Asia/Taipei` 計算借用日期、起訖時間、是否整天與預計通知時間。

### 流程二：Teams 通知與回覆

沿用既有設計：

1. 依 SharePoint `預計通知時間` 判斷是否發送。
2. 發送 Teams Adaptive Card 給借用人。
3. 借用人填寫共乘人數並勾選規範確認。
4. 回覆前檢查 Event ID / iCalUId、卡片版本與事件狀態。
5. 寫回 SharePoint。

## 五、欄位對應

| SharePoint 欄位 | 來源 |
|---|---|
| 車輛名稱 | 由事件參與者中的 Resource Mailbox Email 判斷 |
| 資源信箱 | 事件參與者中的公務車 Resource Mailbox |
| 行事曆事件 ID | `ad.general@alp.global` 行事曆事件 ID |
| iCalUId | Outlook 事件 iCalUId |
| 借用日期 | 事件開始時間轉 Asia/Taipei |
| 借用起始時間 | 事件開始時間轉 Asia/Taipei |
| 借用結束時間 | 事件結束時間轉 Asia/Taipei |
| 借用人姓名 | 事件 Organizer / From |
| 借用人 Email | 事件 Organizer Email |
| 使用事由 | 事件主旨 |
| 是否整天 | Outlook 事件 IsAllDay |
| 預計通知時間 | Power Automate 依規則計算 |

## 六、取消與異動邏輯

因 `ad.general@alp.global` 是會議參與者，當使用者修改或取消 Outlook 會議時，流程信箱的行事曆事件通常也會收到更新。

同步流程需定期重讀未來事件並更新 SharePoint：

1. 事件仍存在：更新時間、主旨、借用人、車輛與通知時間。
2. 事件找不到但 SharePoint 仍為有效：標記為 `已取消` 或 `已失效`。
3. 事件內容異動：更新 SharePoint，並將舊卡片版本標記失效。

## 七、風險與防呆

| 風險 | 防呆方式 |
|---|---|
| 使用者忘記邀請流程信箱 | 建立預約 SOP，承辦人抽查 Outlook / SharePoint 是否一致 |
| 使用者只邀請流程信箱但未邀請車輛 | 流程檢查是否包含三台 Resource Mailbox，不包含則不建立公務車紀錄 |
| 同一事件重複同步 | 以 `iCalUId` 或事件 ID 建立唯一鍵 |
| 事件取消後仍通知 | 同步流程找不到事件時更新為 `已取消`，通知流程排除取消項目 |
| 舊 Adaptive Card 被送出 | 回覆前重新讀取 SharePoint 最新狀態與卡片版本 |

## 八、測試案例

| 測試編號 | 測試項目 | 預期結果 |
|---|---|---|
| ALT-T01 | 預約 Altis 並邀請流程信箱 | SharePoint 新增 Altis 借用紀錄 |
| ALT-T02 | 預約 Camry 並邀請流程信箱 | SharePoint 新增 Camry 借用紀錄 |
| ALT-T03 | 預約 Cross 並邀請流程信箱 | SharePoint 新增 Cross 借用紀錄 |
| ALT-T04 | 未邀請流程信箱 | SharePoint 不會自動新增，列為操作缺漏 |
| ALT-T05 | 只邀請流程信箱但未邀請車輛 | 流程略過，不建立公務車紀錄 |
| ALT-T06 | 修改預約時間 | SharePoint 更新時間並重算預計通知時間 |
| ALT-T07 | 取消預約 | SharePoint 更新為已取消，停止通知 |
| ALT-T08 | 整天借用 | 預計通知時間為借用當天 08:00 |

## 九、結論

本替代方案可在不使用 Premium 連接器、不直接讀取 Resource Mailbox Calendar ID 的前提下，繼續完成自動通知與後台管理流程。

此方案需新增使用者操作規則：預約公務車時必須同步邀請 `ad.general@alp.global`。
