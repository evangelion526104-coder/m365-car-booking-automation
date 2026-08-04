# 專案里程碑

更新日期：2026-08-04（v0.3.4）

| 里程碑 | 目標 | 狀態 | 完成日期或預計日期 |
|---|---|---|---|
| M0 專案啟動 | 建立專案架構與初版文件 | 已完成 | 2026-06-25 |
| M1 需求確認 | 確認 Outlook、Power Automate、Teams、SharePoint 架構 | 已完成 | 2026-07-01 |
| M2 SharePoint 後台建立 | 在 `ALP_TW_AD` 建立 `公務車借用管理` 清單與欄位 | 已完成 | 2026-07-01 |
| M3 功能測試流程建立 | 建立 Power Automate 測試流程並驗證 SharePoint、Outlook 標準連線 | 已完成 | 2026-07-01 |
| M4 通知規則補強 | 加入最早 08:00 與整天借用 08:00 通知規則 | 已完成 | 2026-07-02 |
| M4.5 系統防呆設計 | 納入時區、唯一鍵、取消異動、舊卡失效、Flow 防重複設計 | 已完成 | 2026-07-02 |
| M4.6 SharePoint 防呆欄位建置 | 在正式 SharePoint 清單新增並驗證防呆欄位 | 已完成 | 2026-07-02 |
| M5 資源信箱讀取確認 | 取得三台車事件讀取路徑 | 已完成（改採替代方案） | 2026-07-21，改以「傳送 HTTP 要求」直連 Graph `calendar/events`，取代原先受阻的 Calendar ID／V3 連接器路徑 |
| M6 正式同步流程 | 建立 Outlook 行事曆同步至 SharePoint 流程，並納入防呆欄位與取消異動邏輯 | 已完成 | 2026-07-27（v0.2.16 完成清理、正式化切換與 Concurrency Control） |
| M7 Teams 通知與回覆 | 建立 Adaptive Card 發送、回覆寫回與舊卡失效機制 | 已完成 | 2026-07-29（v0.3.0） |
| M8 端到端與防呆測試 | 完成 Outlook 預約、Teams 回覆、SharePoint 後台與 SG-T01 至 SG-T10 測試 | 已完成 | 2026-08-04（v0.3.3，SG-T09 為最後完成項目） |
| M9 上線驗收 | 行政、總務人員確認可日常使用 | 進行中 | P4 四份上線文件已於 v0.3.4 完成；待清理 FT-113、SG-T09 測試資料後即可完成正式上線驗收 |

## 目前所在里程碑

目前位於 M9，為上線驗收最後階段。M0～M8 皆已完成，P4 上線文件（操作手冊、異常處理說明、日常維護說明、上線前驗收紀錄）已於 v0.3.4 建立完成，詳見 `test-cases/go-live-acceptance-record.md`。剩餘工作為清理本輪測試資料（FT-113、SG-T09），完成後即符合正式上線條件。
## v0.2.7 - ATA-9627 V3 事件讀取測試流程

狀態：進行中

目標：

- 確認 Outlook Editor 權限已在 ATA-9627 生效。
- 確認 `取得行事曆 (V2)` 不能作為 Resource Mailbox 讀取的唯一判斷依據。
- 建立 `取得事件的行事曆檢視 (V3)` 直接讀取 ATA-9627 事件的測試流程。

完成條件：

- Power Automate 可讀到 ATA-9627 測試事件 `TEST`。
- 可取得 subject、start、end、id、iCalUId。
- 測試結果回填至 `docs/ata9627-v3-calendar-event-test.md`。

## v0.2.8 - ATA-9627 Calendar ID 已取得

狀態：進行中

已完成：

- ATA-9627 Calendar ID 已取得：`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`。
- 已將 V3 測試值由資源信箱 Email 改為 Calendar ID。

下一個完成條件：

- 使用上述 Calendar ID 執行 Power Automate V3 事件讀取。
- 可取得 subject、start、end、isAllDay、organizer / creator、id、iCalUId。

## v0.2.9 - ATA-9627 V3 測試實作版

狀態：進行中

已完成：

- 建立 `power-automate/ata9627-v3-test-implementation.md`。
- 明確列出 Power Automate 中 V3 測試流程的動作、命名與 Expression。
- 明確建立 V3 成功前不得進入正式 SharePoint 同步的閘門。

下一個完成條件：

- 使用 `6049e1d1-b34c-4cca-b530-2c7c4b77abe9` 實際執行 V3 測試。
- 可取得 ATA-9627 事件資料與 Event ID / iCalUId。
- 測試成功後進入 `公務車行事曆同步至 SharePoint - ATA9627 MVP`。

## v0.2.10 - ATA-9627 候選 GUID V3 實測未通過

狀態：進行中／受阻

已完成：

- 執行紀錄 `08584172227294871773330429620CU04` 已確認候選 GUID 確實送入 V3。
- 保存 `400 BadRequest / ErrorInvalidIdMalformed / The Id is invalid.` 證據。
- 排除 Access Denied 與查詢期間未涵蓋事件為本次主要原因。

下一個完成條件：

- 取得 Office 365 Outlook V3 可接受的 ATA-9627 Calendar ID。
- 成功取得至少一筆事件及 Event ID / iCalUId。
- 在達成前不得開始 M6 SharePoint 正式同步流程。
