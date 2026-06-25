# GitHub 網頁建立版本規則

本文件規定：每個流程階段完成後，Codex 不只要建立 Git tag，也要自動進入 GitHub Releases 網頁建立可閱讀的版本頁。

## 何時要進入 GitHub 網頁建立新版本

當任一流程階段完成且可被保留為版本時，Codex 應自動執行 GitHub Releases 建版流程。

適用階段包含：

- SharePoint List 階段完成
- Teams Adaptive Card 階段完成
- Power Automate 主流程完成
- SharePoint 狀態更新完成
- 異常情境處理完成
- 功能測試完成
- 正式驗收完成

## Codex 應自動執行的順序

1. 確認該階段已完成。
2. 確認測試或驗收狀態。
3. 更新 `CHANGELOG.md`。
4. 新增 `release-notes/vX.X.md`。
5. 建立 Git commit。
6. 推送 `main` 與 `develop`。
7. 建立並推送 Git tag。
8. 開啟 GitHub Releases 網頁。
9. 以該 tag 建立 GitHub Release。
10. 將 `release-notes/vX.X.md` 內容放入 Release description。
11. 發布 Release。
12. 回報版本網址與完成內容。

## GitHub Releases 網頁位置

Repository：

```text
https://github.com/evangelion526104-coder/m365-car-booking-automation
```

建立新版本頁面：

```text
https://github.com/evangelion526104-coder/m365-car-booking-automation/releases/new
```

## Release 欄位規則

| 欄位 | 填寫規則 |
|---|---|
| Choose a tag | 選擇本次建立的 tag，例如 `v0.2` |
| Release title | 使用 `v0.2 - SharePoint List 階段完成` |
| Description | 貼上 `release-notes/v0.2.md` 內容 |
| Set as latest release | 若是最新完成階段，保持最新版本 |
| Pre-release | 測試版可勾選；正式驗收版不勾選 |

## 版本命名範例

| 階段 | Tag | GitHub Release title |
|---|---|---|
| SharePoint List | `v0.2` | `v0.2 - SharePoint List 階段完成` |
| Teams Adaptive Card | `v0.3` | `v0.3 - Teams Adaptive Card 階段完成` |
| Power Automate 主流程 | `v0.4` | `v0.4 - Power Automate 主流程完成` |
| 功能測試 | `v0.7` | `v0.7 - 功能測試完成` |
| 正式驗收 | `v1.0` | `v1.0 - 正式驗收版本` |

## 重要原則

- GitHub Release 是給非工程人員看的版本頁。
- Git tag 是給版本追蹤與還原用。
- 兩者都要保留，不能只做其中一個。
- 舊版 Release 不刪除、不覆蓋。
- 若只是修正同一階段的小問題，使用小版號，例如 `v0.2.1`。
