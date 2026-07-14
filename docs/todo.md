# 待辦事項

更新日期：2026-07-04

## P0 必須先完成

1. 不再要求以 Exchange Administrator 登入流程；使用 `ad.general@alp.global` 與 Office 365 Outlook 標準連線。
2. 已完成 Outlook 明確顯示 Altis、Camry、Cross 三台共用行事曆後重測；`取得行事曆 (V2)` 仍只回傳個人 Calendar。
3. 評估將三台行事曆資料夾權限調整為 Editor，或改採其他不使用 Premium 連接器的標準讀取方案；流程日常執行仍不需要 Exchange Administrator。
4. 權限或方案調整後，再依 owner Email／顯示名稱找出 Altis、Camry、Cross 的真正 Calendar ID；不得直接把資源信箱 Email 當 Calendar ID。
5. 將三個 Calendar ID 分別套用到 `取得事件的行事曆檢視 (V3)`，讀取未來 7 天事件。
6. 驗證主旨、起始時間、結束時間、是否全天、建立者／借用人、Event ID 與 iCalUId；完成後再次關閉測試流程。
7. 確認資源信箱取消與時間異動是否能被 Power Automate 偵測。
8. 確認所有流程日期時間均轉為 `Asia/Taipei`。
9. 確認 Event ID / iCalUId 唯一識別邏輯可用。
10. 確認舊 Teams Adaptive Card 回覆前防呆檢查可用。
11. 確認 Flow Concurrency Control 與重複資料防止規則可用。

## P1 正式流程開發

1. 建立 `公務車行事曆同步至 SharePoint` 流程。
2. 設定三台資源信箱同步邏輯。
3. 以 `預約唯一鍵` 避免重複建立項目。
4. 建立 `是否整天` 判斷。
5. 建立 `預計通知時間` 計算。
6. 建立取消預約更新為 `取消`。
7. 建立時間異動後重新計算通知時間。
8. 在正式流程中寫入並維護已建立的防呆欄位，例如 iCalUId、事件同步狀態、卡片版本、通知狀態、處理鎖定狀態。
9. 啟用 Concurrency Control，避免重複同步。

## P2 Teams 通知與回覆

1. 建立 `公務車借用前 Teams 通知與回覆` 流程。
2. 以 `預計通知時間` 判斷是否發送。
3. 發送 Adaptive Card 給 `借用人 Email`。
4. 寫入 `通知發送時間`。
5. 接收借用人回覆。
6. 寫入 `共乘人數`、`規範確認狀態`、`Teams 回覆時間`。
7. 完整填寫時更新為 `已完成確認`。
8. 未完整填寫時維持或更新為 `未完成填寫`。
9. 回覆寫回前檢查 Event ID、卡片版本、事件同步狀態。
10. 事件已取消、已失效或已異動時，提示借用人重新確認最新資訊。

## P3 測試與驗收

1. 測試一般借用。
2. 測試早上 08:30 借用，通知時間應為 08:00。
3. 測試 08:00 前借用，通知時間應為 08:00。
4. 測試整天借用，通知時間應為 08:00。
5. 測試 Outlook 預約取消。
6. 測試 Outlook 預約時間異動。
7. 測試未填共乘人數。
8. 測試未勾選規範確認。
9. 測試承辦人後台判斷是否可領鑰。
10. 測試時區轉換與 UTC 原始時間不直接顯示。
11. 測試全天事件不會產生前一天 23:00 通知。
12. 測試舊 Teams Adaptive Card 送出不得覆蓋新資料。
13. 測試重複觸發不會重複建立或重複通知。

## P4 文件與上線

1. 建立行政、總務操作手冊。
2. 建立異常處理說明。
3. 建立日常維護說明。
4. 完成上線前驗收紀錄。
5. 視需要建立 Git tag 與正式 release note。
## v0.2.7 待辦事項 - ATA-9627 V3 事件讀取

1. 建立 Power Automate 測試流程 `公務車功能測試-ATA9627事件讀取`。
2. 在 `取得事件的行事曆檢視 (V3)` 的 Calendar Id 第一輪填入 `room_nhb4_car@alp.global`。
3. 查詢期間設定為 `utcNow()` 至 `addDays(utcNow(),7)`。
4. 執行測試並確認是否可讀到 `subject = TEST`。
5. 若成功，記錄 Event ID 與 iCalUId。
6. 若失敗，保留完整錯誤訊息並分類原因。
7. 測試完成後更新 `docs/ata9627-v3-calendar-event-test.md`、README、CHANGELOG 與 Master 紀錄。
