# 專案開發與版本控管規則

本文件為「M365 公務車借用自動通知與後台管理流程」的永久開發規範。後續所有功能分支、修正分支與文件更新，都必須依照本文件執行，確保專案進度、測試結果與版本紀錄可以被追蹤。

## 一、專案定位

本專案為「M365 公務車借用自動通知與後台管理流程」，目標是使用 Microsoft 365 標準功能，建立公務車預約、通知、填寫確認與後台管理流程。

本專案內容包含：

- Power Automate 流程設計。
- Teams Adaptive Card JSON。
- SharePoint List 欄位設計。
- Outlook / M365 行事曆整合。
- 測試案例。
- 操作與維護文件。
- GitHub 版本控管。

## 二、Branch 規則

| 分支 | 用途 |
|---|---|
| `main` | 正式穩定版本，只放已確認可保留的專案成果 |
| `develop` | 開發整合版本，功能完成後先整合到此分支 |
| `feature/xxx` | 功能開發分支，例如 `feature/teams-adaptive-card` |
| `fix/xxx` | 修正問題分支，例如 `fix/calendar-cancel-status` |
| `docs/xxx` | 文件更新分支，例如 `docs/admin-manual` |

分支命名應簡短、清楚，讓行政、總務與工程協作者都能看出該分支目的。

## 三、Commit 規則

Commit message 應使用「類型: 說明」格式，讓每次異動可以快速被理解。

範例：

- `feat: 完成 SharePoint List 欄位設計`
- `feat: 新增 Teams Adaptive Card 初版`
- `update: 調整公務車管理規範文字`
- `fix: 修正行事曆取消後狀態未更新問題`
- `docs: 新增承辦人操作手冊`

常用類型：

| 類型 | 用途 |
|---|---|
| `feat` | 新增功能 |
| `fix` | 修正錯誤 |
| `update` | 調整既有內容或規則 |
| `docs` | 文件新增或更新 |
| `test` | 測試案例或測試紀錄 |
| `chore` | 不影響功能的維護工作 |

## 四、功能完成後標準流程

每次完成一個功能、修正或階段成果後，必須依序執行以下流程：

1. `git status`
2. 檢查異動檔案。
3. 確認功能完成內容。
4. 更新 `README.md`。
5. 更新 `CHANGELOG.md`。
6. 更新相關 `docs`、`test-cases`、`release-notes`。
7. `git add .`
8. `git commit`
9. `git push`
10. 視需要建立 tag。
11. 回寫主專案最新進度。
12. 檢查功能分支是否同步主專案最新版本。

若目前分支不是預期分支，應先停止提交，確認是否需要切換到 `develop` 或建立新的功能分支。

## 五、版本號規則

本專案使用簡單階段版本號，讓非工程背景人員也能理解每個版本代表的成果。

| 版本 | 代表成果 |
|---|---|
| `v0.1` | 完成需求與資料表設計 |
| `v0.2` | 完成 Adaptive Card 初版 |
| `v0.3` | 完成 Power Automate 主流程 |
| `v0.4` | 完成 SharePoint 狀態更新 |
| `v0.5` | 完成異常處理 |
| `v1.0` | 完成正式驗收版本 |

版本號應搭配 `CHANGELOG.md` 或 `release-notes` 說明該版本內容。

## 六、舊版本保留規則

為確保專案歷程可追蹤，必須遵守以下規則：

- 不得直接覆蓋舊版本。
- 不得刪除歷史 commit。
- 不得任意刪除已建立的 tag。
- 每一階段成果都必須可透過 commit、tag 或 CHANGELOG 追蹤。
- 若需修改舊內容，應建立新 commit，不得直接覆蓋歷史紀錄。

若發現舊版本內容有錯，應以新的修正 commit 說明原因與修正內容。

## 七、每次功能完成後 Codex 檢查清單

每次請 Codex 協助完成功能後，應逐項確認：

- [ ] 本次功能是否完成
- [ ] 是否已完成測試
- [ ] 是否有更新 `README.md`
- [ ] 是否有更新 `CHANGELOG.md`
- [ ] 是否有更新相關文件
- [ ] 是否有更新測試紀錄
- [ ] 是否已檢查 `git status`
- [ ] 是否已建立 commit
- [ ] 是否已 push 到 GitHub
- [ ] 是否需要建立 tag
- [ ] 是否需要建立 release note
- [ ] 是否已回寫主專案最新進度
- [ ] 是否已檢查功能分支與主專案同步狀態
- [ ] 舊版本是否已保留且未覆蓋

## 八、給行政與總務人員的簡短說明

每完成一項功能後，不能只看畫面是否能用，也要確認文件、測試紀錄與版本紀錄都有更新。這樣未來如果流程被調整、發生異常，或需要交接給其他同仁，才能知道每一次改動的原因、時間與負責內容。

# Project Governance（永久生效）

本專案採用「Completion-driven Versioning（完成即版本化）」管理模式。

除非專案負責人明確要求停止或跳過，否則本規則永久生效。後續所有新需求、新功能、新文件、新測試、新修正與新分支，都必須遵守本章規則。

## 一、新工作啟動前檢查（New Work Startup Rule）

每次開始任何新工作、新需求、新功能、新分支或新對話前，Codex 必須先確認目前專案狀態，不得直接開始開發。

開始工作前請先讀取：

- `README.md`
- `CHANGELOG.md`
- `/docs/Project_Workflow.md`

並確認：

- 目前專案版本
- Git Branch
- 專案完成度
- 已完成功能
- 待辦事項
- 最新 CHANGELOG
- 是否有未 Commit 的異動
- 是否有未 Push 的 Commit
- 是否需要同步 `develop` 或 `main`

開始工作前固定回覆：

```text
【目前版本】

【目前 Branch】

【目前完成度】

【目前已完成功能】

【目前待辦事項】

【Git 狀態】

【是否需要同步 GitHub】
```

確認後才開始工作。

## 二、自動判斷功能完成

若符合以下任一條件，即視為完成一個可提交階段：

- 完成一項功能
- 完成一項流程
- 完成一份文件
- 完成一項需求
- 完成一項修正（Bug Fix）
- 完成一份測試案例
- 完成一個可獨立驗收成果
- 完成一個里程碑

不需要等待專案負責人另外要求 commit。

## 三、功能完成後自動 Workflow

完成後請依序自動執行：

1. `git status`
2. 檢查異動內容
3. 整理本次完成內容
4. 更新 `README.md`（如有需要）
5. 更新 `CHANGELOG.md`
6. 更新 `docs` 文件
7. 更新 `test-cases`（如有需要）
8. 更新 `release-notes`（如有需要）
9. Git Commit
10. Push GitHub
11. 視需要建立 Git Tag
12. 更新專案完成度
13. 更新專案里程碑
14. 更新待辦事項
15. 更新下一階段工作
16. 檢查目前 Branch 是否與 Master 同步

不得等待專案負責人再次提醒。

## 四、自動更新專案進度

每完成一個功能，自動更新：

- 專案完成百分比
- 已完成功能
- 已完成流程
- 已完成文件
- 已完成測試
- `README.md`
- `CHANGELOG.md`
- `Project Workflow`
- Release Notes
- 下一步工作

Master 專案永遠保持最新狀態。

## 五、自動版本控管

若完成內容已達可驗收階段，請自動：

- 建立 Commit
- 視需要建立 Tag
- 視需要建立 Release Note
- Push GitHub

不得覆蓋：

- Commit
- Tag
- Release
- 歷史版本

所有版本均需保留。

若未達建立 Tag 條件，仍須建立 Commit，並說明原因。

## 六、自動同步 GitHub

完成 Workflow 後，若沒有衝突或權限問題，請自動：

- Push 到目前 Branch
- 同步 GitHub 最新版本

若 Push 失敗，請說明原因並提出修正方式。

## 七、自動同步 Master

功能完成後請同步：

- `README.md`
- `CHANGELOG.md`
- `docs`
- Project Workflow
- 專案完成度
- 專案里程碑
- 待辦事項

保持 Master 為專案唯一最新版本。

## 八、功能分支同步檢查

每次 Workflow 完成後，請確認目前 Branch 是否已同步 Master。

請檢查：

- 是否缺少 Commit
- 是否缺少文件
- 是否缺少需求
- 是否缺少 CHANGELOG
- 是否需要 Merge
- 是否需要 Rebase

若不同步，請直接提出改善方式。

## 九、固定回覆格式

每次 Workflow 執行完成後，固定回覆：

```text
【本次完成功能】

【更新文件】

【目前 Branch】

【Commit ID】

【Commit Message】

【是否已 Push GitHub】

【是否建立 Tag】

【目前版本】

【目前完成度】

【目前 Master 是否同步】

【下一步工作】
```

## 十、專案最高原則

本專案以「專案狀態永遠保持最新」為最高原則。

所有功能完成後，都應立即同步：

- GitHub
- `README.md`
- `CHANGELOG.md`
- Project Workflow
- 專案里程碑
- 專案完成度
- 功能分支
- Master 專案

任何版本均不得覆蓋歷史紀錄。

所有 Commit、Tag、Release、CHANGELOG 必須可追溯。

## 十一、行政與總務人員可讀摘要

這套治理規則的意思是：每做完一件可驗收的事，都要留下紀錄、更新文件、提交版本並推送到 GitHub。這樣未來交接、查問題、回復舊版本或確認某個流程何時被調整，都有清楚依據。

## 十二、System Safeguards 檢查規則

後續只要異動 Power Automate、Teams Adaptive Card、SharePoint List、Outlook 同步、測試案例或操作文件，都必須同步檢查是否納入五項系統防呆設計：

- 統一時區為 `Asia/Taipei`。
- 以 Outlook Event ID / iCalUId 作為唯一識別碼。
- Outlook 取消與異動需同步 SharePoint。
- 舊 Teams Adaptive Card 回覆前需重新檢查並可失效。
- Flow 必須避免重複建立、重複通知與同時執行造成的資料衝突。

若本次異動會影響上述任一項，必須同步更新 [system-safeguards.md](system-safeguards.md)、測試案例與相關設計文件。
