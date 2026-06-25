# Power Automate 流程設計

此資料夾存放 Power Automate 流程設計、設定紀錄與測試結果。

## 建議流程

1. Outlook 資源行事曆同步至 SharePoint。
2. 借用前 1 小時發送 Teams Adaptive Card。
3. 借用人送出卡片後更新 SharePoint List。
4. 承辦人依 SharePoint List 狀態決定是否發放鑰匙。

## 測試優先原則

正式啟用前，先以測試資料執行：

- 資源行事曆是否讀取得到
- SharePoint 是否可新增／更新
- Teams Adaptive Card 是否可送達
- 借用人回覆是否能寫回 SharePoint
- 未回覆、取消、改期、未勾選規範等異常情境是否正確

## 目前注意事項

- 流程帳號 `ad.general@alp.global` 需具備 SharePoint 站台編輯權限。
- 三台公務車資源行事曆需讓流程帳號可讀取。
- 不使用 Power Automate Premium 連接器。
