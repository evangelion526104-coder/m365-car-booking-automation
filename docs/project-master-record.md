# M365 公務車借用自動通知與後台管理流程 Master 專案紀錄

更新日期：2026-07-02  
Master 狀態：本文件為目前唯一最新版本，後續功能分支應以本文件為基準。

## 一、目前專案總覽

| 項目 | 內容 |
|---|---|
| 專案名稱 | M365 公務車借用自動通知與後台管理流程 |
| 專案目標 | 讓員工透過 Outlook 預約公務車，系統自動同步至 SharePoint 後台，並在借用前透過 Teams Adaptive Card 通知借用人填寫共乘人數與確認公務車使用規範，供承辦人判斷是否發放鑰匙 |
| 主要架構 | Outlook 資源行事曆 + Power Automate + Teams Adaptive Card + SharePoint List |
| 授權限制 | O365 E1 可行，不使用 Power Automate Premium 連接器 |
| 目前開發階段 | 後台 SharePoint List 已完成，Power Automate 功能測試流程已完成，正式自動化串接前 |
| 專案完成度 | 58% |
| 下一個預計完成階段 | 完成三台公務車資源行事曆讀取確認，並建立正式行事曆同步流程 |

## 二、本次新增完成內容

### SharePoint

- 已在 `ALP_TW_AD` 建立 SharePoint List：`公務車借用管理`。
- 已建立並驗證主要欄位。
- 已新增 `是否整天` 欄位，用於支援 Outlook 整天預約。
- 已新增 `預計通知時間` 欄位，用於支援借用前通知規則。
- 已確認清單網址為 `https://alpglobal.sharepoint.com/sites/ALP_TW_AD/Lists/List6/AllItems.aspx`。

### Power Automate

- 已建立功能測試流程：`公務車功能測試-SharePoint清單連線`。
- 已確認 SharePoint 標準連接器可讀取 `公務車借用管理` 清單。
- 已確認 Office 365 Outlook 標準連接器基本連線可用。
- 已將正式流程拆分為兩條建議流程：行事曆同步、Teams 通知與回覆。

### Teams Adaptive Card

- 已完成卡片內容設計。
- 已納入固定版「公司公務車使用管理規範」文字。
- 已設計借用人必填欄位：共乘人數、規範確認。
- 已建立 Adaptive Card JSON 範例。

### Outlook

- 已確認三台公務車資源信箱資訊。
- 已確認 Outlook 標準連接器基本可用。
- 尚需確認流程帳號 `ad.general@alp.global` 是否可讀取三台資源信箱的完整行事曆預約內容。

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

## 三、目前各功能狀態

| 功能 | 狀態 | 完成度 | 備註 |
|---|---|---:|---|
| 需求分析 | 已完成 | 100% | 核心需求、限制與例外情境已整理 |
| SharePoint List | 已完成 | 100% | 已在 `ALP_TW_AD` 建立並驗證 |
| Power Automate | 進行中 | 35% | 功能測試流程完成，正式自動化尚未完成 |
| Adaptive Card | 設計完成 | 80% | JSON 範例完成，尚未正式串接發送 |
| Teams 通知 | 設計完成，待串接 | 30% | 通知時間規則已確認 |
| Outlook | 部分完成 | 40% | 標準連線可用，資源信箱完整讀取待確認 |
| 測試案例 | 進行中 | 45% | 基礎連線測試通過，端到端測試待執行 |
| 維護文件 | 已更新 | 85% | Master 文件已建立，後續需隨實作更新 |
| 操作手冊 | 待建立 | 20% | 尚需整理行政人員日常操作手冊 |
| Git 版本 | 已建立 | 70% | 目前在 `main` 分支，文件已更新，是否提交由專案負責人決定 |

## 四、本次異動摘要

### 新增功能

- 新增 SharePoint 欄位 `是否整天`。
- 新增 SharePoint 欄位 `預計通知時間`。
- 新增通知時間規則：最早 08:00，整天借用 08:00。
- 新增 Master 專案紀錄文件。
- 新增里程碑、完成清單、待辦事項文件。

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
| 1 | 確認資源信箱讀取權限 | 確認 `ad.general@alp.global` 可讀取三台公務車資源行事曆完整內容 |
| 2 | 建立正式行事曆同步流程 | 將 Outlook 預約同步到 SharePoint List |
| 3 | 建立通知時間計算 | 依 `是否整天`、`借用起始時間` 計算 `預計通知時間` |
| 4 | 建立 Teams Adaptive Card 發送流程 | 到達 `預計通知時間` 後發送給借用人 |
| 5 | 建立回覆寫回 SharePoint | 寫入共乘人數、規範確認狀態、Teams 回覆時間 |
| 6 | 建立狀態更新邏輯 | 未完成填寫、已完成確認、取消、已領鑰 |
| 7 | 執行端到端測試 | 從 Outlook 預約到 Teams 回覆到 SharePoint 後台確認 |
| 8 | 建立操作手冊 | 提供行政、總務人員日常使用與異常處理 |
| 9 | 上線前驗收 | 確認三台車、不同時段、整天借用、取消與時間異動皆通過 |

## 六、Master 專案紀錄更新狀態

| 項目 | 狀態 |
|---|---|
| README | 已更新 |
| CHANGELOG | 已更新 |
| 專案里程碑 | 已更新 |
| 完成清單 | 已更新 |
| 待辦事項 | 已更新 |

## 七、目前判斷

本專案目前已完成後台資料結構與基礎連線測試，已具備進入正式自動化串接的條件。

正式上線前的最大關鍵仍是：確認流程帳號可讀取三台 Outlook 資源信箱的完整預約內容。此項確認完成後，即可開始建立正式同步與 Teams 通知流程。
