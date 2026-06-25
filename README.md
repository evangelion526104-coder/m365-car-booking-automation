# M365 公務車借用自動通知與後台管理流程

Repository：`m365-car-booking-automation`

本專案用來管理「Outlook 資源行事曆 + Power Automate + Teams Adaptive Card + SharePoint List」的設計、測試、操作與版本紀錄。

目標是讓行政／總務同仁可以追蹤每一階段完成內容，並在功能完成後由 Codex 協助整理異動、建立 Git commit、推送到 GitHub。

## 專案內容

- `docs/`：流程設計、版控規則、操作與維護文件
- `sharepoint/`：SharePoint List 欄位設計與設定紀錄
- `adaptive-cards/`：Teams Adaptive Card JSON 與說明
- `power-automate/`：Power Automate 流程設計與設定紀錄
- `test-cases/`：功能測試案例與驗收清單
- `release-notes/`：每個版本的發行紀錄
- `CHANGELOG.md`：所有版本與異動摘要

## 目前版本

- 目前版本：`v0.1.1`
- 目前狀態：已建立專案版控規則、資料夾結構、功能完成檢查清單，以及階段版本保留規則
- 下一階段：整理 SharePoint List、Adaptive Card、Power Automate 測試版流程文件

## 階段版本原則

本專案採用「每個流程階段完成後都建立新版本」的方式：

- 每完成一個階段，建立新的 Git tag。
- 舊版本不覆蓋、不刪除。
- 每個版本都有對應 release note。
- `CHANGELOG.md` 集中記錄每個版本完成內容。

例如：

- `v0.2`：完成 SharePoint List 階段
- `v0.3`：完成 Teams Adaptive Card 階段
- `v0.4`：完成 Power Automate 主流程階段
- `v1.0`：完成正式驗收版本

詳細規則請見 [docs/version-control-workflow.md](docs/version-control-workflow.md) 與 [docs/stage-versioning-policy.md](docs/stage-versioning-policy.md)。

## 功能完成後的標準動作

每完成一個功能或階段成果後，Codex 應協助執行：

1. 檢查本次異動內容
2. 整理完成項目與測試結果
3. 更新 `CHANGELOG.md`
4. 必要時更新 `README.md` 或 `docs/`
5. 建立清楚的 Git commit
6. 推送到 GitHub 對應 branch
7. 若達到階段完成條件，建立 tag 與 release note
