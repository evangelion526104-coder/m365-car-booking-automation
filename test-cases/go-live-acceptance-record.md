# 上線前驗收紀錄

更新日期：2026-08-04（對應版本 v0.3.3）

本文件彙整「M365 公務車借用自動通知與後台管理流程」全部功能測試（FT）與系統防呆測試（SG-T）之最終結果，作為正式上線前的驗收證明。詳細測試過程與證據請見 `test-cases/function-test-plan.md` 與各版本 `release-notes/vX.X.md`。

## 一、功能測試（FT）結果彙總

| 測試編號 | 測試項目 | 結果 |
|---|---|---|
| FT-101 | Outlook 建立一般公務車預約，SharePoint 同步新增 | 通過 |
| FT-102 | Outlook 修改借用時間，SharePoint 同步更新並重算通知時間 | 通過 |
| FT-103 | Outlook 取消預約，SharePoint 標記已取消並停止通知 | 通過（v0.2.15） |
| FT-104 | 借用時間 09:30，預計通知時間 08:30 | 通過 |
| FT-105 | 借用時間 08:30，預計通知時間 08:00 | 通過 |
| FT-106 | 借用時間早於 08:00，預計通知時間 08:00 | 通過 |
| FT-107 | 整天借用，是否整天=是、預計通知時間 08:00 | 通過 |
| FT-108 | 到達預計通知時間，Teams Adaptive Card 發送 | 通過 |
| FT-109 | 借用人填寫共乘人數並勾選規範，SharePoint 更新為已完成確認 | 通過 |
| FT-110 | 借用人未填共乘人數，卡片端擋下無法送出 | 通過 |
| FT-111 | 借用人未勾選規範，卡片端擋下無法送出 | 通過 |
| FT-112 | 承辦人依領鑰狀態判斷是否可發放鑰匙 | 通過 |
| FT-113 | 同一台車重複預約，系統標記異常供承辦人判斷 | 通過（v0.3.2，已知限制見下方第三節） |
| FT-114 | 以取得行事曆(V2)取得三台車 Calendar ID 逐台讀取 | 受阻（已由 HTTP 直連 Graph 之替代方案取代，不影響正式流程；正式流程改採「傳送 HTTP 要求」直連 Graph API，三台車皆已驗證可行） |

## 二、系統防呆測試（SG-T）結果彙總

| 測試編號 | 測試項目 | 結果 |
|---|---|---|
| SG-T01 | 時區轉換測試，統一顯示 Asia/Taipei | 通過（v0.2.14） |
| SG-T02 | 全天事件測試，預計通知時間為當天 08:00 | 通過 |
| SG-T03 | 全天事件避免前一天通知 | 通過 |
| SG-T04 | 行事曆取消測試 | 通過（v0.2.15） |
| SG-T05 | 行事曆異動測試（時間／車輛／主旨／借用人） | 部分通過：時間異動已驗證通過；車輛、主旨、借用人異動與舊卡片失效機制之獨立測試尚未逐項執行，但其底層機制（比對 Event ID、卡片版本、事件最後修改時間）已於 SG-T06 驗證有效 |
| SG-T06 | 舊 Teams Adaptive Card 送出測試（正向／負向路徑） | 通過（v0.3.0） |
| SG-T07 | 重複觸發測試 | 通過（以歷史執行記錄佐證） |
| SG-T08 | Event ID 唯一鍵測試 | 通過 |
| SG-T09 | 重複資料異常測試（相同預約唯一鍵） | 通過（v0.3.3） |
| SG-T10 | Flow Concurrency 測試 | 通過（v0.2.16） |

## 三、已知限制（不阻擋上線，已於文件明確記錄）

| 項目 | 說明 | 對應文件 |
|---|---|---|
| 通知發送時間欄位未實際寫入 | 欄位存在但未寫入精確時間戳，以通知狀態欄位反映是否已通知 | `docs/incident-handling-guide.md` |
| 重複資料檢查結果欄位共用 | FT-113 與 SG-T09 共用同一欄位，同時符合兩種異常時後執行者覆蓋前者 | `docs/incident-handling-guide.md` |
| SharePoint DispForm 顯示時間偶爾疊加時區偏移 | 顯示層現象，重新整理後恢復正常，不影響底層資料 | `docs/incident-handling-guide.md` |
| SG-T05 車輛／主旨／借用人異動情境未逐項獨立測試 | 底層比對機制已於 SG-T06 驗證有效，但未針對此三種異動類型個別建立測試案例 | 本文件第二節 |

## 四、測試資料清理狀態

| 測試批次 | Outlook 測試事件 | SharePoint 測試紀錄 | 狀態 |
|---|---|---|---|
| FT-113（v0.3.2） | TEST-FT113-A/B、A2/B2、A3/B3 | ID 13、14、16、17 | 已清理（2026-08-04）：SharePoint ID 13/14/16/17 確認不存在；Outlook 側 A/B、A3/B3 於資源行事曆已不存在，A2/B2（因資源信箱自動拒絕而殘留於 AD General 帳號之邀請副本）已取消並移除 |
| SG-T09（v0.3.3） | TEST-SGT09-A（2026/8/4 14:00-14:30，ATA-9627） | ID 19、ID 21 | 待清理，專案負責人自行清理 |

## 五、系統防呆設計驗收標準對照（依 `docs/system-safeguards.md` 第九節）

- [x] 所有日期時間已轉為 Asia/Taipei（SG-T01 通過）。
- [x] Event ID / iCalUId 唯一識別邏輯已完成（SG-T08、SG-T09 通過）。
- [x] 行事曆新增、修改、取消同步 SharePoint 已完成（FT-101～FT-103 通過）。
- [x] 舊 Adaptive Card 回覆前檢查已完成（SG-T06 通過）。
- [x] Flow Concurrency Control 與重複資料防止已完成（SG-T10、SG-T09 通過）。
- [x] 測試案例 SG-T01 至 SG-T10 已通過（SG-T05 為部分通過，說明見第二節與第三節）。

## 六、驗收結論

功能測試（FT-101～FT-113）與系統防呆測試（SG-T01～SG-T10）核心項目均已通過，FT-114 已由替代技術方案（HTTP 直連 Graph）解決並驗證，不影響正式流程運作。FT-113 本輪測試資料已於 2026-08-04 完成清理。剩餘事項為已明確記錄之非阻擋性已知限制，以及 SG-T09 測試資料清理（由專案負責人自行執行）。待該批清理完成後，系統即符合正式上線條件。

## 七、相關文件

- [test-cases/function-test-plan.md](function-test-plan.md) — 完整測試過程與證據
- [docs/system-safeguards.md](../docs/system-safeguards.md) — 系統防呆設計原理與驗收標準
- [docs/operation-manual.md](../docs/operation-manual.md) — 日常操作手冊
- [docs/incident-handling-guide.md](../docs/incident-handling-guide.md) — 異常處理說明
- [docs/maintenance-guide.md](../docs/maintenance-guide.md) — 日常維護說明
