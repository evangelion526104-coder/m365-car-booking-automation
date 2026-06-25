# GitHub 版本控管規則

本文是本專案的固定版控流程。核心原則是：每完成一個功能或階段成果，就由 Codex 協助檢查異動、整理說明、commit、push 到 GitHub，讓版本與進度都可追蹤。

## 一、Branch 規則

| Branch | 用途 | 給非工程人員的理解 |
|---|---|---|
| `main` | 正式穩定版本 | 已驗收、可當正式參考的版本 |
| `develop` | 開發整合版本 | 功能仍在整合與測試中 |
| `feature/sharepoint-list` | SharePoint 清單欄位與後台 | 調整資料表或欄位 |
| `feature/adaptive-card` | Teams Adaptive Card | 調整 Teams 通知卡片 |
| `feature/power-automate-flow` | Power Automate 流程 | 調整自動化流程 |
| `feature/function-test` | 功能測試流程 | 加入測試資料、測試欄位、測試案例 |
| `fix/calendar-update-handling` | 修正行事曆異動問題 | 修正取消、改時間、重複預約等問題 |
| `docs/user-manual` | 文件與操作手冊 | 調整給承辦人或使用者看的文件 |

### Branch 命名原則

- 新功能：`feature/功能名稱`
- 問題修正：`fix/問題名稱`
- 文件更新：`docs/文件名稱`
- 測試調整：`test/測試名稱`

範例：

```powershell
.\tools\Git\cmd\git.exe checkout -b feature/adaptive-card
.\tools\Git\cmd\git.exe checkout -b docs/user-manual
.\tools\Git\cmd\git.exe checkout -b fix/calendar-cancel-status
```

若電腦已安裝 Git，也可以直接使用 `git`：

```powershell
git checkout -b feature/adaptive-card
```

## 二、Commit 規則

Commit message 使用「類型: 完成內容」格式，讓紀錄一眼看懂。

| 類型 | 使用時機 | 範例 |
|---|---|---|
| `feat` | 新增功能 | `feat: 完成 SharePoint List 欄位設計` |
| `update` | 調整既有內容 | `update: 調整公務車管理規範文字` |
| `fix` | 修正問題 | `fix: 修正行事曆取消後狀態未更新問題` |
| `docs` | 文件更新 | `docs: 新增承辦人操作手冊` |
| `test` | 測試案例或測試紀錄 | `test: 新增功能測試驗收清單` |
| `chore` | 版控、資料夾、設定整理 | `chore: 建立 GitHub 版控流程` |

## 三、每完成一個功能後的標準流程

Codex 每次在功能完成後，應依序做以下事情：

1. 檢查目前異動：

```powershell
.\tools\Git\cmd\git.exe status --short --branch
```

2. 檢查異動內容：

```powershell
.\tools\Git\cmd\git.exe diff
```

3. 確認本次完成內容：

- 本次完成哪些功能
- 是否有影響 SharePoint 欄位
- 是否有影響 Power Automate 流程
- 是否有影響 Teams Adaptive Card
- 是否有新增或更新測試案例

4. 更新文件：

- 更新 `CHANGELOG.md`
- 必要時更新 `README.md`
- 必要時更新 `docs/`
- 必要時新增 `release-notes/vX.X.md`

5. 建立 commit：

```powershell
.\tools\Git\cmd\git.exe add .
.\tools\Git\cmd\git.exe commit -m "feat: 新增 Teams Adaptive Card 初版"
```

6. 推送到 GitHub：

```powershell
.\tools\Git\cmd\git.exe push origin feature/adaptive-card
```

7. 需要時建立 tag：

```powershell
.\tools\Git\cmd\git.exe tag v0.2
.\tools\Git\cmd\git.exe push origin v0.2
```

## 四、功能完成後自動整理、commit、push 的工作方式

每次使用者說「功能完成，請整理並推送」時，Codex 應執行：

1. 讀取 `docs/codex-feature-done-checklist.md`
2. 檢查 Git 狀態
3. 列出本次異動檔案
4. 彙整本次完成摘要
5. 確認是否需要更新測試紀錄
6. 更新 `CHANGELOG.md`
7. 建立 commit
8. 推送到目前 branch
9. 若是階段版本，建立 tag 與 release note

## 五、GitHub Repository 結構

建議結構：

```text
/
├─ README.md
├─ CHANGELOG.md
├─ docs/
│  ├─ version-control-workflow.md
│  └─ codex-feature-done-checklist.md
├─ adaptive-cards/
│  └─ README.md
├─ power-automate/
│  └─ README.md
├─ sharepoint/
│  └─ list-schema.md
├─ test-cases/
│  └─ function-test-plan.md
└─ release-notes/
   └─ v0.1.md
```

## 六、版本號規則

| 版本 | 完成條件 |
|---|---|
| `v0.1` | 完成需求、版控規則與資料夾結構 |
| `v0.2` | 完成 Adaptive Card 初版 |
| `v0.3` | 完成 Power Automate 主流程 |
| `v0.4` | 完成 SharePoint 狀態更新 |
| `v0.5` | 完成異常處理 |
| `v0.6` | 完成功能測試並完成修正 |
| `v1.0` | 完成正式驗收版本 |

## 七、常用 Git 指令範例

建立功能分支：

```powershell
.\tools\Git\cmd\git.exe checkout -b feature/adaptive-card
```

檢查目前異動：

```powershell
.\tools\Git\cmd\git.exe status --short --branch
```

加入全部異動：

```powershell
.\tools\Git\cmd\git.exe add .
```

建立 commit：

```powershell
.\tools\Git\cmd\git.exe commit -m "feat: 新增 Teams Adaptive Card 初版"
```

推送分支：

```powershell
.\tools\Git\cmd\git.exe push origin feature/adaptive-card
```

建立 tag：

```powershell
.\tools\Git\cmd\git.exe tag v0.2
.\tools\Git\cmd\git.exe push origin v0.2
```

設定 GitHub remote：

```powershell
.\tools\Git\cmd\git.exe remote add origin https://github.com/ORG/REPO.git
```

第一次推送 main：

```powershell
.\tools\Git\cmd\git.exe push -u origin main
```
