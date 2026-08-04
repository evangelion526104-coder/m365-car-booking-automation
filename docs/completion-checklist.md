# 完成清單

更新日期：2026-08-04（v0.3.4）

## 已完成

- [x] 確認主要架構：Outlook 資源行事曆 + Power Automate + Teams Adaptive Card + SharePoint List。
- [x] 確認不使用 Power Automate Premium 連接器。
- [x] 確認 SharePoint 建立位置改為 `ALP_TW_AD`。
- [x] 建立 SharePoint List：`公務車借用管理`。
- [x] 建立車輛名稱欄位。
- [x] 建立資源信箱欄位。
- [x] 建立行事曆事件 ID 欄位。
- [x] 建立預約唯一鍵欄位。
- [x] 建立借用日期、借用起始時間、借用結束時間欄位。
- [x] 建立借用人姓名與 Email 欄位。
- [x] 建立使用事由欄位。
- [x] 建立共乘人數欄位。
- [x] 建立規範確認狀態欄位。
- [x] 建立 Teams 回覆時間欄位。
- [x] 建立通知發送時間欄位。
- [x] 建立最後同步時間欄位。
- [x] 建立領鑰狀態欄位。
- [x] 建立承辦人備註欄位。
- [x] 建立是否測試資料欄位。
- [x] 建立功能測試結果欄位。
- [x] 建立是否整天欄位。
- [x] 建立預計通知時間欄位。
- [x] 建立 Power Automate 功能測試流程。
- [x] 完成 SharePoint 標準連接器測試。
- [x] 完成 Office 365 Outlook 標準連接器基本測試。
- [x] 建立並執行資源信箱讀取測試流程。
- [x] 確認資源信箱 Email 不能直接作為 `取得事件的行事曆檢視 (V3)` Calendar ID。
- [x] 關閉資源信箱讀取測試流程，避免排程持續失敗。
- [x] 完成 Teams Adaptive Card 設計。
- [x] 完成 Adaptive Card JSON 範例。
- [x] 完成通知時間規則設計。
- [x] 完成 Master 專案紀錄更新。
- [x] 完成系統防呆設計文件。
- [x] 完成五項防呆機制設計：時區、唯一鍵、取消異動、舊卡失效、Flow 防重複。
- [x] 完成防呆測試案例設計。
- [x] 新增防呆欄位到正式 SharePoint List。
- [x] 完成 `iCalUId`、`唯一識別來源`、`事件同步狀態`、`卡片版本`、`卡片狀態`、`通知狀態`、`Flow 執行 ID`、`處理鎖定狀態`、`重複資料檢查結果`、`防呆檢查結果`、`已取消`、`已失效` 與相關時間欄位建立。
- [x] 完成 ATA-9627 V3 事件讀取測試實作版文件，包含 Power Automate 動作、Expression、成功判斷與下一步 MVP 草案。
- [x] 使用候選 GUID 完成一次 V3 實測，確認輸入已送達連接器並取得 `ErrorInvalidIdMalformed` 證據。

## v0.2.11～v0.3.4 期間新完成項目

- [x] 放棄 Calendar ID／V3 連接器路徑，改以「傳送 HTTP 要求」直連 Graph `calendar/events`，三台車（Altis、Camry、Cross）皆完成事件讀取驗證。
- [x] 停用 `公務車功能測試-ATA9627事件讀取` 等暫存測試流程。
- [x] 建立正式 Outlook 行事曆同步流程 `公務車行事曆同步至SharePoint`（v0.2.12 建立，v0.2.13～v0.2.16 陸續修正並正式化）。
- [x] 實作 Asia/Taipei 統一時區轉換（SG-T01，v0.2.14 修正 Update 路徑 +8 小時位移錯誤後驗證通過）。
- [x] 實作 Event ID / iCalUId 唯一識別與重複資料防止（SG-T08、SG-T09，SG-T09 於 v0.3.3 完成補強邏輯）。
- [x] 建立行事曆取消預約同步邏輯（FT-103／SG-T04，v0.2.15）。
- [x] 建立行事曆時間異動同步邏輯（FT-102）。
- [x] 建立 `預計通知時間` 自動計算邏輯（FT-104～FT-107）。
- [x] 建立 Teams Adaptive Card 正式發送流程 `公務車借用前Teams通知與回覆`（v0.3.0）。
- [x] 建立 Teams 回覆寫回 SharePoint 流程（v0.3.0）。
- [x] 建立舊 Teams Adaptive Card 失效機制（SG-T06，v0.3.0）。
- [x] 啟用 Flow Concurrency Control 與執行鎖（SG-T10，v0.2.16）。
- [x] 建立未完成填寫狀態維持邏輯（FT-110、FT-111，Adaptive Card 必填/必勾選機制）。
- [x] 建立承辦人已領鑰更新流程或操作規則（`docs/admin-manual-confirmation-procedure.md`，v0.3.2）。
- [x] 完成系統防呆測試 SG-T01 至 SG-T10（SG-T05 為部分通過，詳見 `test-cases/go-live-acceptance-record.md`）。
- [x] 完成端到端測試（P3 情境測試全數執行）。
- [x] 建立行政、總務操作手冊（`docs/operation-manual.md`，v0.3.4）。
- [x] 建立異常處理說明（`docs/incident-handling-guide.md`，v0.3.4）。
- [x] 建立日常維護說明（`docs/maintenance-guide.md`，v0.3.4）。
- [x] 建立上線前驗收紀錄（`test-cases/go-live-acceptance-record.md`，v0.3.4）。

## 尚未完成

- [ ] 清理 FT-113、SG-T09 本輪測試資料（詳見 `test-cases/go-live-acceptance-record.md` 第四節）。
- [ ] 完成正式上線驗收確認（測試資料清理後即可完成）。
