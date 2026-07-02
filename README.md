# M365 公務車借用自動通知與後台管理流程

本專案以 Microsoft 365 標準功能建置公務車借用後台與自動通知流程，主要架構為：

`Outlook 資源行事曆 + Power Automate + Teams Adaptive Card + SharePoint List`

設計原則：

- 使用 O365 E1 可行的標準連接器。
- 不使用 Power Automate Premium 連接器。
- 以 SharePoint List 作為後台管理資料來源。
- 以 Teams Adaptive Card 作為借用人回覆介面。
- 保留測試階段，確認功能符合需求後再啟用正式自動化。

## 目前 Master 狀態

| 項目 | 內容 |
|---|---|
| 專案名稱 | M365 公務車借用自動通知與後台管理流程 |
| 目前開發階段 | 後台清單已完成，功能測試流程已建立，系統防呆設計已納入，正式自動化串接前 |
| 專案完成度 | 62% |
| 最新更新日期 | 2026-07-02 |
| Master 紀錄 | [docs/project-master-record.md](docs/project-master-record.md) |
| SharePoint 清單 | [公務車借用管理](https://alpglobal.sharepoint.com/sites/ALP_TW_AD/Lists/List6/AllItems.aspx) |
| Power Automate 測試流程 | 公務車功能測試-SharePoint清單連線 |

## 已確認環境

| 類別 | 內容 |
|---|---|
| SharePoint 站台 | `https://alpglobal.sharepoint.com/sites/ALP_TW_AD` |
| SharePoint List | `公務車借用管理` |
| 流程帳號 | `ad.general@alp.global` |
| 承辦人 | AD總機 / `ad.general@alp.global` |
| 已確認權限 | 流程帳號可讀取 `ALP_TW_AD` 清單 |
| 已確認連線 | SharePoint 標準連接器、Office 365 Outlook 標準連接器 |

## 公務車資源

| 車輛 | 資源信箱 |
|---|---|
| 公務車Altis ATA-9627 B4-16-永聯內湖辦公室 | `room_nhb4_car@alp.global` |
| 公務車Camry BKX-2370 B4-17-永聯內湖辦公室 | `room_nhb4_car_camry@alp.global` |
| 公務車Cross BKY-0762 B4-44-永聯內湖辦公室 | `room_nhb4_car_cross@alp.global` |

## 最新通知規則

Teams Adaptive Card 通知規則已更新為：

- 一般借用：借用起始時間前 1 小時通知。
- 最早通知時間：不得早於借用當天早上 08:00。
- 整天借用：一律於借用當天早上 08:00 通知。
- SharePoint 後台新增 `是否整天` 與 `預計通知時間` 欄位，用來支援此規則。

## 專案文件

| 文件 | 用途 |
|---|---|
| [docs/project-master-record.md](docs/project-master-record.md) | 唯一最新 Master 專案紀錄 |
| [docs/project-milestones.md](docs/project-milestones.md) | 專案里程碑 |
| [docs/completion-checklist.md](docs/completion-checklist.md) | 完成清單 |
| [docs/todo.md](docs/todo.md) | 待辦事項 |
| [docs/system-safeguards.md](docs/system-safeguards.md) | 系統防呆設計，上線前必須完成 |
| [sharepoint/list-schema.md](sharepoint/list-schema.md) | SharePoint List 欄位設計 |
| [power-automate/README.md](power-automate/README.md) | Power Automate 流程設計與現況 |
| [adaptive-cards/README.md](adaptive-cards/README.md) | Teams Adaptive Card 設計 |
| [test-cases/function-test-plan.md](test-cases/function-test-plan.md) | 功能測試計畫 |
| [CHANGELOG.md](CHANGELOG.md) | 版本異動紀錄 |

## 下一階段

下一階段目標是完成正式自動化串接：

1. 確認 `ad.general@alp.global` 可讀取三台公務車資源行事曆內容。
2. 建立 `公務車行事曆同步至 SharePoint` 流程。
3. 建立 `公務車借用前 Teams 通知與回覆` 流程。
4. 串接 Teams Adaptive Card 回覆寫回 SharePoint。
5. 完成端到端功能測試後，再啟用正式流程。

## 專案版本控管規則

本專案已建立永久性的開發與版本控管規則，後續所有功能分支、修正分支與文件更新都必須遵循。

每完成一個功能或階段成果後，必須確認功能完成、完成必要測試、更新文件、檢查異動、建立清楚的 Git commit，並視需要 push 到對應分支、建立 tag 或 release note。

完整規則請參考：[docs/Project_Workflow.md](docs/Project_Workflow.md)。

## Project Governance（專案治理規則）

本專案採用「Completion-driven Versioning（完成即版本化）」管理模式。除非專案負責人明確要求停止或跳過，否則每次開始新工作前，都必須先確認目前專案狀態；每完成一項功能、流程、文件、修正、測試或里程碑，都必須更新相關文件、建立 Git commit，並推送到 GitHub。

治理規則重點：

- 新工作開始前，必須先讀取 `README.md`、`CHANGELOG.md`、`docs/Project_Workflow.md`。
- 完成功能後，必須自動更新 README、CHANGELOG、docs、測試紀錄與下一步工作。
- 可驗收成果必須建立 commit，必要時建立 tag 或 release note。
- 所有 commit、tag、release、CHANGELOG 必須可追溯。
- 不得覆蓋或刪除舊版本。
- Workflow 完成後，必須確認目前分支是否與 Master 專案同步。

完整治理規則請參考：[docs/Project_Workflow.md](docs/Project_Workflow.md#project-governance永久生效)。

## 系統防呆設計（正式上線前必須完成）

本專案已將五項系統防呆設計列為正式上線前必須完成的門檻，後續 Power Automate、Teams Adaptive Card、SharePoint 欄位、測試案例與文件都必須納入。

| 防呆項目 | 要求 |
|---|---|
| 統一時區 | 所有日期時間統一轉為 `Asia/Taipei`，不得直接使用未轉換 UTC |
| 唯一識別碼 | 每筆預約以 Outlook `Event ID` 或 `iCalUId` 作為唯一識別 |
| 取消／異動同步 | Outlook 修改或取消時，SharePoint 必須同步更新 |
| 舊卡片失效 | Adaptive Card 回覆前需重新檢查最新狀態，避免舊卡覆蓋新資料 |
| Flow 防重複 | 啟用 Concurrency Control，避免重複建立與重複通知 |

完整防呆設計請參考：[docs/system-safeguards.md](docs/system-safeguards.md)。
