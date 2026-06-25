# Codex 功能完成後執行檢查清單

每次功能或階段成果完成後，Codex 應照此清單檢查並執行。

## A. 功能完成確認

| 檢查項目 | 結果 |
|---|---|
| 本次功能是否已完成 | 待確認 |
| 是否清楚列出完成內容 | 待確認 |
| 是否有未完成或待補事項 | 待確認 |
| 是否影響正式流程 | 待確認 |
| 是否需要使用者確認後才能啟用 | 待確認 |

## B. 文件與測試確認

| 檢查項目 | 結果 |
|---|---|
| 是否更新 `README.md` | 待確認 |
| 是否更新 `CHANGELOG.md` | 待確認 |
| 是否更新相關 `docs/` 文件 | 待確認 |
| 是否有測試紀錄 | 待確認 |
| 是否有功能測試結果 | 待確認 |
| 是否需要新增 release note | 待確認 |

## C. Git 執行確認

| 檢查項目 | 結果 |
|---|---|
| 是否已執行 Git status | 待確認 |
| 是否已檢查異動內容 | 待確認 |
| 是否已建立清楚 commit | 待確認 |
| 是否已 push 到 GitHub | 待確認 |
| 是否需要建立 tag | 待確認 |
| 是否需要建立 GitHub release | 待確認 |

## D. Codex 每次應執行的 Git 指令

```powershell
.\tools\Git\cmd\git.exe status --short --branch
.\tools\Git\cmd\git.exe diff
.\tools\Git\cmd\git.exe add .
.\tools\Git\cmd\git.exe commit -m "類型: 本次完成內容"
.\tools\Git\cmd\git.exe push
```

## E. 完成回報格式

Codex 完成後應回報：

- 本次完成內容
- 本次異動檔案
- 測試或驗證結果
- Commit ID
- Push 目標 branch
- 是否建立 tag / release
- 是否有阻礙或需使用者補充資訊
