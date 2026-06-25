# GitHub 版本控管規則

本文是本專案的固定版控流程。核心原則是：每完成一個功能或階段成果，就由 Codex 協助檢查異動、整理說明、commit、push 到 GitHub，並建立可追蹤的版本紀錄。

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

## 三、階段版本規則

本專案採用「階段完成即建立新版本」的方式。每一個重要流程階段完成後，都要建立新版本，不覆蓋舊版本。

### 固定原則

- 每完成一個功能階段，就建立一個新的 tag。
- 舊 tag 永遠保留，不修改、不刪除。
- 每個版本都要有對應 release note。
- `CHANGELOG.md` 要記錄每個版本完成內容。
- `main` 保存已確認可作為正式參考的版本。
- `develop` 保存下一階段整合中的版本。
- 功能尚未完成或尚未測試通過時，不建立正式版本 tag。

### 階段版本建議

| 版本 | 階段 | 完成條件 |
|---|---|---|
| `v0.1` | 專案初始化 | 完成需求、版控規則與資料夾結構 |
| `v0.1.1` | 版控規則補強 | 完成階段版本與舊版本保留規則 |
| `v0.2` | SharePoint List | 完成清單欄位、測試欄位、後台檢視 |
| `v0.3` | Teams Adaptive Card | 完成卡片 JSON、規範文字、必填欄位 |
| `v0.4` | Power Automate 主流程 | 完成行事曆讀取、SharePoint 寫入、通知排程 |
| `v0.5` | SharePoint 狀態更新 | 完成回覆寫回、領鑰狀態判斷 |
| `v0.6` | 異常處理 | 完成未回覆、取消、改期、重複預約處理 |
| `v0.7` | 功能測試 | 完成全流程測試並修正問題 |
| `v1.0` | 正式驗收 | 完成正式上線與承辦人驗收 |

### 小版號使用方式

若同一階段內有補強或修正，使用小版號：

- `v0.2`：SharePoint List 初版完成
- `v0.2.1`：補上測試欄位或檢視
- `v0.2.2`：修正欄位名稱或狀態選項

## 四、舊版本保留方式

每個版本都至少保留三種紀錄：

| 保留方式 | 目的 |
|---|---|
| Git tag | 可以回到當時的完整檔案狀態 |
| `release-notes/vX.X.md` | 給非工程人員閱讀該版完成什麼 |
| `CHANGELOG.md` | 集中查詢所有版本異動 |

查詢舊版本：

```powershell
.\tools\Git\cmd\git.exe tag --list
```

切換到舊版本查看：

```powershell
.\tools\Git\cmd\git.exe checkout v0.2
```

回到最新正式版本：

```powershell
.\tools\Git\cmd\git.exe checkout main
```

## 五、每完成一個功能後的標準流程

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
- 是否達到可建立新版本的階段完成條件

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
.\tools\Git\cmd\git.exe push origin main
```

7. 若達到階段完成條件，建立 tag：

```powershell
.\tools\Git\cmd\git.exe tag v0.3
.\tools\Git\cmd\git.exe push origin v0.3
```

8. 自動進入 GitHub Releases 網頁建立版本：

- 開啟 `https://github.com/evangelion526104-coder/m365-car-booking-automation/releases/new`
- 選擇剛建立的 tag
- 填入版本標題
- 貼上 `release-notes/vX.X.md` 內容
- 發布 GitHub Release

這一步是正式版本控管流程的一部分，不可只建立 tag 而沒有 GitHub Release。

## 六、GitHub Repository 結構

```text
/
├─ README.md
├─ CHANGELOG.md
├─ docs/
│  ├─ version-control-workflow.md
│  ├─ stage-versioning-policy.md
│  ├─ github-release-web-workflow.md
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
   ├─ v0.1.md
   └─ v0.1.1.md
```

## 七、常用 Git 指令範例

建立功能分支：

```powershell
.\tools\Git\cmd\git.exe checkout -b feature/adaptive-card
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
