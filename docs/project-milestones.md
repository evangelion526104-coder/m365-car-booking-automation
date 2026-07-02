# 專案里程碑

更新日期：2026-07-02

| 里程碑 | 目標 | 狀態 | 完成日期或預計日期 |
|---|---|---|---|
| M0 專案啟動 | 建立專案架構與初版文件 | 已完成 | 2026-06-25 |
| M1 需求確認 | 確認 Outlook、Power Automate、Teams、SharePoint 架構 | 已完成 | 2026-07-01 |
| M2 SharePoint 後台建立 | 在 `ALP_TW_AD` 建立 `公務車借用管理` 清單與欄位 | 已完成 | 2026-07-01 |
| M3 功能測試流程建立 | 建立 Power Automate 測試流程並驗證 SharePoint、Outlook 標準連線 | 已完成 | 2026-07-01 |
| M4 通知規則補強 | 加入最早 08:00 與整天借用 08:00 通知規則 | 已完成 | 2026-07-02 |
| M4.5 系統防呆設計 | 納入時區、唯一鍵、取消異動、舊卡失效、Flow 防重複設計 | 已完成 | 2026-07-02 |
| M4.6 SharePoint 防呆欄位建置 | 在正式 SharePoint 清單新增並驗證防呆欄位 | 已完成 | 2026-07-02 |
| M5 資源信箱讀取確認 | 以 Reviewer 權限取得三台 Calendar ID 並讀取未來 7 天事件 | 進行中 | 2026-07-02 已確認不需要 Exchange Administrator，待網頁編輯器恢復後實測 |
| M6 正式同步流程 | 建立 Outlook 行事曆同步至 SharePoint 流程，並納入防呆欄位與取消異動邏輯 | 待開始 | 待排程 |
| M7 Teams 通知與回覆 | 建立 Adaptive Card 發送、回覆寫回與舊卡失效機制 | 待開始 | 待排程 |
| M8 端到端與防呆測試 | 完成 Outlook 預約、Teams 回覆、SharePoint 後台與 SG-T01 至 SG-T10 測試 | 待開始 | 待排程 |
| M9 上線驗收 | 行政、總務人員確認可日常使用 | 待開始 | 待排程 |

## 目前所在里程碑

目前位於 M5。已確認 `ad.general@alp.global` 具有三台車 Calendar Reviewer 權限，本專案不需要 Exchange Administrator。直接以資源信箱 Email 測試回傳「ID 格式不正確」，下一步須以 `取得行事曆 (V2)` 取得真正 Calendar ID，再用 `取得事件的行事曆檢視 (V3)` 測試。2026-07-02 本輪因 Power Automate 網頁編輯器自動控制持續逾時，實際 ID 與事件欄位尚未取得。
