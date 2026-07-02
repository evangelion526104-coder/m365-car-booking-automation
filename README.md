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
| 目前開發階段 | 後台清單已完成，功能測試流程已建立，正式自動化串接前 |
| 專案完成度 | 58% |
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
