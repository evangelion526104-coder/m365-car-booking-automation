# 日常維護說明

更新日期：2026-08-04（對應版本 v0.3.3）

本文件說明系統上線後，維運人員（IT 或熟悉 Power Automate 的承辦人）應定期執行的檢查項目，確保兩條正式流程持續穩定運作。

## 一、系統組成速覽

| 元件 | 名稱 | 說明 |
|---|---|---|
| 正式流程一 | 公務車行事曆同步至SharePoint | 每 15 分鐘讀取三台公務車資源信箱行事曆，同步至 SharePoint |
| 正式流程二 | 公務車借用前Teams通知與回覆 | 每 5 分鐘檢查待通知紀錄，發送 Adaptive Card 並處理回覆 |
| 後台資料庫 | SharePoint「公務車借用管理」清單 | `https://alpglobal.sharepoint.com/sites/ALP_TW_AD/Lists/List6/AllItems.aspx` |
| 流程執行帳號 | `ad.general@alp.global` | 對三台資源信箱具 Editor 委派權限 |

## 二、定期檢查項目

### 每週檢查

1. 開啟 Power Automate「我的流程」，確認兩條正式流程狀態皆為「開啟」，且最近執行紀錄無連續失敗。
2. 檢查「我的流程」清單中是否又出現重複或非預期的背景流程（v0.2.16 曾發現重複與遺留測試流程並清理，建議定期複查避免再度累積）。
3. 抽查 SharePoint 清單近期新增紀錄，確認欄位皆有正確寫入（借用起訖時間、預計通知時間、事件同步狀態等）。

### 每月檢查

1. 抽查「重複資料檢查結果」欄位是否有未處理的「疑似重複」或「異常」紀錄長期未清理，若有應依 `docs/incident-handling-guide.md` 處理或移除測試殘留資料。
2. 確認三台公務車資源信箱（Altis／Camry／Cross）之 Editor 委派權限仍然有效，未因帳號或權限異動而失效。
3. 檢查是否有測試資料（`是否測試資料 = 是`）遺留在正式清單中未清理。

### 版本或組態異動後

1. 依 `docs/Project_Workflow.md` 版本控管規則，任何流程修改都必須先在測試流程或副本驗證，確認零錯誤後才套用至正式流程。
2. 修改完成後更新 README、CHANGELOG、對應 release note，並建立新的 Git tag／GitHub Release。
3. 若修改涉及 SharePoint 欄位新增或型別調整，需同步更新 `sharepoint/list-schema.md` 與 `docs/system-safeguards.md`。

## 三、測試資料清理原則

- 測試紀錄應勾選「是否測試資料」，並於測試完成後盡快清理對應 Outlook 測試事件與 SharePoint 測試紀錄，避免與正式借用紀錄混淆。
- 每次版本開發過程中產生的測試資料清理清單，會記錄於對應的 `release-notes/vX.X.md` 檔案中；維運人員可依此清單逐一核對是否已清理完畢。
- 目前尚待清理（截至 v0.3.3）：FT-113 測試事件與 SharePoint 紀錄（詳見 `release-notes/v0.3.2.md`）、SG-T09 測試事件 TEST-SGT09-A 與 SharePoint 紀錄 ID 19、ID 21（詳見 `release-notes/v0.3.3.md`）。此批清理由專案負責人自行執行，不由維運人員或自動化流程代為清理。

## 四、常見維護操作

### 暫停單一流程

若需暫時關閉某條正式流程（例如系統維護、SharePoint 欄位調整期間），於 Power Automate「我的流程」找到對應流程並關閉排程，完成調整後記得重新開啟，並手動觸發一次確認零錯誤。

### 檢查 Concurrency Control 設定

兩條正式流程之 Recurrence 觸發程序皆應維持「並行控制」啟用、平行處理原則程度為 1，避免流程重疊執行造成 SharePoint 重複或競爭寫入。若因故被重設，需重新啟用（見 `docs/system-safeguards.md` SG-05）。

### 確認 Editor 委派權限

三台公務車資源信箱權限異動需由 IT 執行，維運人員僅需確認 `ad.general@alp.global` 仍可於 Outlook 正常查看、新增、修改、刪除各資源信箱行事曆事件，如有異常應請 IT 協助確認 Calendar Folder Permission 設定。

## 五、相關文件

- [docs/operation-manual.md](operation-manual.md) — 日常操作手冊
- [docs/incident-handling-guide.md](incident-handling-guide.md) — 異常處理說明
- [docs/system-safeguards.md](system-safeguards.md) — 系統防呆設計原理
- [docs/Project_Workflow.md](Project_Workflow.md) — 版本控管規則
- [test-cases/go-live-acceptance-record.md](../test-cases/go-live-acceptance-record.md) — 上線前驗收紀錄
