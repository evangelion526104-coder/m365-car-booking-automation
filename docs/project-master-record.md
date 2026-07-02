# M365 公務車借用自動通知與後台管理流程 Master 專案紀錄

更新日期：2026-07-02  
Master 狀態：本文件為目前唯一最新版本，後續功能分支應以本文件為基準。

## 一、目前專案總覽

| 項目 | 內容 |
|---|---|
| 專案名稱 | M365 公務車借用自動通知與後台管理流程 |
| 目前版本 | `v0.2.5` |
| 專案目標 | 讓員工透過 Outlook 預約公務車，系統自動同步至 SharePoint 後台，並在借用前透過 Teams Adaptive Card 通知借用人填寫共乘人數與確認公務車使用規範，供承辦人判斷是否發放鑰匙 |
| 主要架構 | Outlook 資源行事曆 + Power Automate + Teams Adaptive Card + SharePoint List |
| 授權限制 | O365 E1 可行，不使用 Power Automate Premium 連接器 |
| 目前開發階段 | M5 已確認 Calendar Reviewer 權限足夠，待取得三台車 Calendar ID 與逐台事件讀取實測 |
| 專案完成度 | 69% |
| 下一個預計完成階段 | 完成三台資源行事曆 Calendar ID 可見性與逐台讀取驗證，再建立正式 Outlook 同步流程 |

## 二、本次新增完成內容

### SharePoint

- 已在 `ALP_TW_AD` 建立 SharePoint List：`公務車借用管理`。
- 已建立並驗證主要欄位。
- 已新增 `是否整天` 欄位，用於支援 Outlook 整天預約。
- 已新增 `預計通知時間` 欄位，用於支援借用前通知規則。
- 已在正式清單新增並驗證系統防呆欄位，包含事件唯一識別、UTC 原始時間、事件同步狀態、卡片版本、通知狀態、Flow 執行鎖、重複資料檢查、取消與失效旗標。
- 已確認清單網址為 `https://alpglobal.sharepoint.com/sites/ALP_TW_AD/Lists/List6/AllItems.aspx`。

### Power Automate

- 已建立功能測試流程：`公務車功能測試-SharePoint清單連線`。
- 已確認 SharePoint 標準連接器可讀取 `公務車借用管理` 清單。
- 已確認 Office 365 Outlook 標準連接器基本連線可用。
- 已將正式流程拆分為兩條建議流程：行事曆同步、Teams 通知與回覆。
- 已建立並執行 `公務車功能測試-資源信箱讀取`（Flow ID：`7aff726f-0f9d-4b7e-a128-baeb368ab1ce`）。
- 已確認 Calendar ID 選單目前只有個人 `Calendar`；直接填入 Altis 資源信箱會回傳「ID 格式不正確」。
- 測試流程已關閉，保留執行紀錄供追蹤。

### Teams Adaptive Card

- 已完成卡片內容設計。
- 已納入固定版「公司公務車使用管理規範」文字。
- 已設計借用人必填欄位：共乘人數、規範確認。
- 已建立 Adaptive Card JSON 範例。

### Outlook

- 已確認三台公務車資源信箱資訊。
- 已確認 Outlook 標準連接器基本可用。
- 2026-07-02 已實測資源行事曆讀取；目前三台車未出現在 Power Automate Calendar ID 選單。
- 資源信箱 Email 不是 `取得事件的行事曆檢視 (V3)` 可接受的 Calendar ID。
- 已確認 `ad.general@alp.global` 具有三台資源信箱 Calendar Reviewer 權限，可正常讀取行事曆。
- 本專案不修改 Resource Mailbox、Calendar Processing 或 Exchange 組態，因此不需要 Exchange Administrator。
- 下一步使用 `取得行事曆 (V2)` 取得真正 Calendar ID，再逐台重測。

### Excel

- 目前不使用 Excel 作為正式資料來源。
- 後台管理以 SharePoint List 為唯一主檔，避免多份資料造成維護成本。

### 通知流程

- 已新增通知時間規則：
  - 一般借用：借用起始時間前 1 小時通知。
  - 最早通知時間：當天早上 08:00。
  - 整天借用：當天早上 08:00。
- 已新增 `預計通知時間` 欄位，供 Power Automate 計算後寫回。

### 文件

- 已更新 `README.md`。
- 已更新 `CHANGELOG.md`。
- 已更新 SharePoint 欄位文件。
- 已更新 Power Automate 流程文件。
- 已更新 Teams Adaptive Card 文件。
- 已更新功能測試計畫。
- 已新增 Master 專案紀錄、里程碑、完成清單、待辦事項。

### 測試

- SharePoint List 欄位驗證完成。
- Power Automate SharePoint 連線測試完成。
- Power Automate Outlook 標準連接器基本測試完成。
- 通知時間支援欄位驗證完成。
- 資源行事曆 Calendar ID 可見性測試未通過，已確認為 M5 正式阻擋點。
- 資源信箱讀取測試流程已關閉，避免持續排程失敗。
- 已修正先前將 Calendar ID 錯誤推論為需要 Exchange Administrator 的判斷；既有 Reviewer 權限已足夠。
- 本輪 Power Automate 網頁編輯器自動控制持續逾時，尚未取得 Calendar ID；詳見 `docs/calendar-access-test.md`。

### 系統防呆設計

- 已將五項防呆機制列為正式上線前必須完成：
  - 統一時區為 `Asia/Taipei`。
  - 以 Event ID / iCalUId 作為唯一識別。
  - Outlook 取消與異動需同步 SharePoint。
  - 舊 Teams Adaptive Card 回覆前需重新檢查並可失效。
  - Flow 啟用執行鎖與重複資料防止。
- 已新增防呆設計文件：`docs/system-safeguards.md`。
- 已補強 SharePoint、Power Automate、Adaptive Card、測試案例文件。
- 已將防呆欄位實際建立於 SharePoint 正式清單，後續正式流程可直接寫入與更新。

## 三、目前各功能狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---:|---|
| 需求分析 | 已完成 | 100% | 核心需求、限制與例外情境已整理 |
| SharePoint List | 已完成 | 100% | 已在 `ALP_TW_AD` 建立並驗證 |
| Power Automate | 進行中 | 38% | 已完成資源行事曆存取實測與阻擋診斷，正式自動化尚未完成 |
| Adaptive Card | 設計完成 | 80% | JSON 範例完成，尚未正式串接發送 |
| Teams 通知 | 設計完成，待串接 | 30% | 通知時間規則已確認 |
| Outlook | 測試中 | 42% | 標準連線與 Reviewer 權限可用；待取得真正 Calendar ID 並逐台讀取 |
| 系統防呆設計 | 欄位已落地，流程待實作 | 82% | 五項防呆機制已文件化，SharePoint 欄位已建立 |
| 測試案例 | 進行中 | 60% | 已記錄 Calendar ID 可見性與無效 ID 實測，防呆情境待實測 |
| 維護文件 | 已更新 | 94% | Master、治理、防呆、欄位與 Exchange 權限文件已更新 |
| 操作手冊 | 待建立 | 20% | 尚需整理行政人員日常操作手冊 |
| Git 版本 | 已建立 | 80% | 目前在 `develop` 分支，依 Project Governance 自動提交與同步 |

## 四、本次異動摘要

### 新增功能

- 新增 SharePoint 欄位 `是否整天`。
- 新增 SharePoint 欄位 `預計通知時間`。
- 新增通知時間規則：最早 08:00，整天借用 08:00。
- 新增 Master 專案紀錄文件。
- 新增里程碑、完成清單、待辦事項文件。
- 新增系統防呆設計文件與五項正式上線前必做防呆機制。
- 新增 SharePoint 正式清單防呆欄位：`iCalUId`、`唯一識別來源`、`事件最後修改時間`、`原始開始時間 UTC`、`原始結束時間 UTC`、`顯示時區`、`事件同步狀態`、`取消時間`、`卡片版本`、`卡片狀態`、`最新通知批次 ID`、`通知狀態`、`Flow 執行 ID`、`處理鎖定狀態`、`處理鎖定時間`、`重複資料檢查結果`、`防呆檢查結果`、`已取消`、`已失效`。

### 修改功能

- 專案 SharePoint 建立位置統一改為 `ALP_TW_AD`。
- 通知邏輯由單純「借用前 1 小時」改為「借用前 1 小時，但最早 08:00」。
- 整天借用通知時間改為固定當天 08:00。
- SharePoint 後台新增欄位以支援正式通知判斷。

### 修正 Bug

- 修正文檔亂碼問題。
- 修正早上過早借用時，通知時間可能落在上班前的流程設計問題。
- 修正整天借用沒有明確通知時間的規則缺口。

### 新增文件

- `docs/project-master-record.md`
- `docs/project-milestones.md`
- `docs/completion-checklist.md`
- `docs/todo.md`
- `release-notes/v0.2.0.md`
- `docs/Exchange_Resource_Mailbox_Permission.md`
- `release-notes/v0.2.4-resource-mailbox-access-check.md`

### 更新文件

- `README.md`
- `CHANGELOG.md`
- `sharepoint/list-schema.md`
- `power-automate/README.md`
- `adaptive-cards/README.md`
- `test-cases/function-test-plan.md`

## 五、下一階段工作

| 優先順序 | 工作項目 | 說明 |
|---:|---|---|
| 1 | 取得三台 Calendar ID | 透過 `取得行事曆 (V2)` 列出可用行事曆並記錄三台資源行事曆真正 ID |
| 2 | 建立正式行事曆同步流程 | 將 Outlook 預約同步到 SharePoint List，並寫入已建立的防呆欄位 |
| 3 | 建立取消與異動同步 | 行事曆取消、時間、車輛、主旨或借用人異動時同步更新 |
| 4 | 建立通知時間計算 | 依 `Asia/Taipei`、`是否整天`、`借用起始時間` 計算 `預計通知時間` |
| 5 | 建立 Teams Adaptive Card 發送流程 | 到達 `預計通知時間` 後發送給借用人，並避免重複通知 |
| 6 | 建立回覆前防呆檢查 | 檢查 Event ID、卡片版本、事件狀態，避免舊卡片覆蓋資料 |
| 7 | 建立 Flow 執行鎖 | 啟用 Concurrency Control，避免重複資料與 Race Condition |
| 8 | 執行端到端與防呆測試 | 從 Outlook 預約到 Teams 回覆到 SharePoint 後台確認 |
| 9 | 建立操作手冊 | 提供行政、總務人員日常使用與異常處理 |
| 10 | 上線前驗收 | 五項防呆機制與所有端到端測試通過後才可上線 |

## 六、Master 專案紀錄更新狀態

| 項目 | 狀態 |
|---|---|
| README | 已更新 |
| CHANGELOG | 已更新 |
| 專案里程碑 | 已更新 |
| 完成清單 | 已更新 |
| 待辦事項 | 已更新 |

## 七、目前判斷

本專案目前已完成後台資料結構、系統防呆欄位、基礎連線測試與資源行事曆存取閘門實測。SharePoint 與 Outlook 連接器驗證正常，但尚未具備進入正式同步流程的資源行事曆存取條件。

目前不需要 Exchange Administrator。`ad.general@alp.global` 已具備三台資源行事曆 Calendar Reviewer 權限；P0 工作改為透過 Office 365 Outlook 標準連線的 `取得行事曆 (V2)` 取得真正 Calendar ID，再用 `取得事件的行事曆檢視 (V3)` 逐台驗證未來 7 天事件欄位。本輪因 Power Automate 網頁編輯器自動控制持續逾時，尚未完成實際 ID 與事件讀取。
