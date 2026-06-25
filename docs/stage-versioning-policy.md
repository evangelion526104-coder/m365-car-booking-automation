# 階段版本與舊版本保留規則

本文件用來說明：本專案不是等到最後完成才建立版本，而是每個流程階段完成後都建立一個可追蹤版本。

## 核心原則

1. 每個流程階段完成後都要建立新版本。
2. 舊版本必須保留，不覆蓋、不刪除。
3. 每個版本要能清楚回答：「這個版本完成了什麼？」
4. 尚未完成測試或驗收的內容，不升正式版本。
5. 若只是同一階段內的小調整，用小版號。

## 本專案階段版本表

| 版本 | 流程階段 | 建立版本的條件 |
|---|---|---|
| `v0.1` | 專案初始化 | 建立基本文件、版控規則、資料夾結構 |
| `v0.1.1` | 版控規則補強 | 補上階段版本與舊版本保留規則 |
| `v0.2` | SharePoint List | 清單欄位、測試欄位、後台檢視確認完成 |
| `v0.3` | Teams Adaptive Card | 卡片 JSON、規範文字、必填驗證確認完成 |
| `v0.4` | Power Automate 主流程 | Outlook 行事曆讀取、SharePoint 寫入、通知排程完成 |
| `v0.5` | 狀態更新流程 | Teams 回覆後可即時更新 SharePoint 狀態 |
| `v0.6` | 異常處理 | 未回覆、取消、改期、重複預約等情境完成 |
| `v0.7` | 功能測試 | 測試案例完成，問題已修正 |
| `v1.0` | 正式驗收 | 行政／總務確認可正式使用 |

## 同一階段內的小版號

若某個階段已建立版本，但後續有小調整，使用小版號：

| 範例 | 用途 |
|---|---|
| `v0.2` | SharePoint List 初版完成 |
| `v0.2.1` | 補充測試欄位 |
| `v0.2.2` | 修正欄位名稱 |
| `v0.2.3` | 補充清單檢視或承辦人備註欄位 |

## 每次建立版本時要做的事

1. 確認階段完成內容。
2. 確認是否完成測試。
3. 更新 `CHANGELOG.md`。
4. 新增 `release-notes/vX.X.md`。
5. 建立 commit。
6. 推送到 GitHub。
7. 建立並推送 tag。

## 指令範例

```powershell
.\tools\Git\cmd\git.exe status --short --branch
.\tools\Git\cmd\git.exe add .
.\tools\Git\cmd\git.exe commit -m "feat: 完成 SharePoint List 階段"
.\tools\Git\cmd\git.exe push origin main
.\tools\Git\cmd\git.exe tag v0.2
.\tools\Git\cmd\git.exe push origin v0.2
```

## 舊版本如何查看

查看所有版本：

```powershell
.\tools\Git\cmd\git.exe tag --list
```

查看某一版內容：

```powershell
.\tools\Git\cmd\git.exe checkout v0.2
```

回到最新版本：

```powershell
.\tools\Git\cmd\git.exe checkout main
```
