# CHANGELOG

本檔案記錄本專案的主要異動。最新 Master 狀態以 `docs/project-master-record.md` 為準。

## v0.2.3 - SharePoint 防呆欄位建置完成

* 已在 SharePoint `公務車借用管理` 清單實際新增並驗證系統防呆欄位
* 新增 Event ID / iCalUId、事件同步、卡片狀態、通知狀態、Flow 執行鎖與重複資料檢查相關欄位
* 新增 `已取消`、`已失效` 防呆旗標欄位
* 更新 Master 專案紀錄、SharePoint 欄位設計、完成清單、待辦事項、里程碑與 release note
* 目前仍待完成正式 Power Automate 同步、Teams 通知回覆與 SG-T01 至 SG-T10 實測

## System Safeguards

* 新增系統防呆設計正式文件
* 新增 Asia/Taipei 統一時區規則
* 新增 Event ID / iCalUId 唯一識別規則
* 新增 Outlook 取消與異動同步 SharePoint 規則
* 新增舊 Teams Adaptive Card 回覆前失效檢查
* 新增 Flow Concurrency Control 與重複資料防止規則
* 新增防呆欄位設計與測試案例
* 明確標示五項防呆機制為正式上線前必須完成

## Project Governance

* 新增 New Work Startup Rule
* 新增 Completion-driven Versioning
* 新增自動 Workflow
* 新增 Master 同步規則
* 新增 Branch 同步檢查
* 新增 GitHub 自動同步規則
* 新增固定回覆格式

## v0.1 - 專案版本控管規則建立

* 建立專案開發與版本控管規則
* 新增功能完成後標準流程
* 新增 Branch / Commit / Tag 規則
* 新增舊版本保留規則
* 新增 Codex 功能完成檢查清單

## [v0.2.0-master] - 2026-07-02

### 新增

- 建立 SharePoint List：`公務車借用管理`。
- 將正式建立位置調整為 `ALP_TW_AD` 站台。
- 新增完整 SharePoint 欄位設計，包含車輛、資源信箱、借用資訊、Teams 回覆、領鑰狀態、測試欄位。
- 新增 `是否整天` 欄位，用於判斷 Outlook 整天預約。
- 新增 `預計通知時間` 欄位，用於記錄 Power Automate 計算後的 Teams 預定通知時間。
- 建立 Power Automate 功能測試流程：`公務車功能測試-SharePoint清單連線`。
- 確認 SharePoint 標準連接器可讀取 `ALP_TW_AD` 清單。
- 確認 Office 365 Outlook 標準連接器可執行基本連線測試。
- 新增 Teams 通知時間規則：借用前 1 小時通知，但不得早於借用當天 08:00；整天借用一律 08:00 通知。
- 新增 Master 專案紀錄：`docs/project-master-record.md`。
- 新增專案里程碑：`docs/project-milestones.md`。
- 新增完成清單：`docs/completion-checklist.md`。
- 新增待辦事項：`docs/todo.md`。
- 新增 v0.2.0 release note：`release-notes/v0.2.0.md`。

### 修改

- 重寫 `README.md`，改為唯一最新專案總覽。
- 重寫 `sharepoint/list-schema.md`，更新為實際已建立欄位。
- 重寫 `power-automate/README.md`，補上目前已完成測試與待完成正式流程。
- 重寫 `adaptive-cards/README.md`，補上 Teams Adaptive Card 內容與 JSON 範例。
- 重寫 `test-cases/function-test-plan.md`，補上目前測試狀態與後續測試案例。

### 修正

- 修正原專案文件部分文字亂碼問題。
- 修正專案建立位置紀錄，從 `ALP_TW_AD-AD` 改為 `ALP_TW_AD`。
- 明確標示正式自動化尚未啟用，避免誤判為已完成上線。

### 尚未完成

- 尚未完成三台資源行事曆的正式讀取驗證。
- 尚未完成正式 Outlook 行事曆同步流程。
- 尚未完成 Teams Adaptive Card 正式發送與回覆寫回流程。
- 尚未完成端到端測試。

## [v0.1.2] - 2026-06-25

### 新增

- 建立初版版本控管與 GitHub Release 流程文件。
- 建立階段版本策略。
- 建立 Codex 功能完成檢查清單。

## [v0.1.1] - 2026-06-25

### 新增

- 補充版本命名與階段交付規則。
- 補充 README 內的開發階段說明。

## [v0.1] - 2026-06-25

### 新增

- 建立初始專案資料夾。
- 建立初版 SharePoint、Power Automate、Adaptive Card、測試文件骨架。
