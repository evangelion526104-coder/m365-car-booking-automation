# 行政／總務操作手冊

更新日期：2026-08-04（對應版本 v0.3.3）

## 一、本手冊適用對象與範圍

本手冊提供給日常負責公務車借用後台管理的行政／總務人員（承辦人）使用，說明系統正常運作下的日常操作方式。系統架構為：

`Outlook 資源行事曆 + Power Automate + Teams Adaptive Card + SharePoint List`

借用人透過 Outlook 建立公務車借用事件後，系統會自動同步至 SharePoint 後台，並於借用前以 Teams 卡片通知借用人確認共乘人數與規範，承辦人只需於 SharePoint 後台查看狀態即可判斷是否可發放鑰匙。若遇到系統未涵蓋的例外情況，請參閱本手冊「異常處理」章節或 `docs/incident-handling-guide.md`。

## 二、正常借用流程說明

1. 借用人於 Outlook 對應公務車資源行事曆（Altis／Camry／Cross）建立借用事件，填寫借用起訖時間、主旨、是否整天。
2. 系統每 15 分鐘自動執行「公務車行事曆同步至SharePoint」流程，將 Outlook 事件同步為 SharePoint「公務車借用管理」清單中的一筆紀錄。
3. 系統依規則自動計算「預計通知時間」：一般借用為借用起始時間前 1 小時；不得早於借用當天 08:00；整天借用一律於借用當天 08:00 通知。
4. 到達預計通知時間後，「公務車借用前Teams通知與回覆」流程會發送 Teams Adaptive Card 給借用人，請借用人填寫共乘人數並勾選規範確認。
5. 借用人送出卡片後，SharePoint「領鑰狀態」自動更新為「已完成確認」，承辦人即可依此發放鑰匙。
6. 若借用人未完整填寫（未填共乘人數或未勾選規範），Adaptive Card 端會直接擋下無法送出，SharePoint 不會被不完整資料覆蓋，「領鑰狀態」會維持在待確認狀態。

## 三、承辦人日常查看方式

SharePoint 清單：[公務車借用管理](https://alpglobal.sharepoint.com/sites/ALP_TW_AD/Lists/List6/AllItems.aspx)

承辦人於借用人前來領鑰時，應查看以下欄位判斷是否可發放：

| 欄位 | 判斷方式 |
|---|---|
| 領鑰狀態 | 須為「已完成確認」才可發放鑰匙；「未完成填寫」「待通知」不應發放 |
| 事件同步狀態 | 須為「有效」；「已取消」「已異動」「已失效」不應發放 |
| 已取消 / 已失效 | 須皆為「否」 |
| 重複資料檢查結果 | 須為「正常」；「疑似重複」或「異常」須先確認（見下方異常處理） |
| 共乘人數 / 規範確認狀態 | 供核對借用人是否確實填寫完整 |

## 四、例外情況：借用人未透過 Teams 卡片回覆

若借用人因未收到通知、Teams 帳號問題或臨時借用等原因，未透過 Adaptive Card 完成回覆，但承辦人已透過電話、當面或 Email 等站外方式確認借用人已知悉規範且資訊完整，可由承辦人直接於 SharePoint 後台手動確認。

完整操作步驟、必須同步更新的欄位清單與常見錯誤，請務必參閱：[docs/admin-manual-confirmation-procedure.md](admin-manual-confirmation-procedure.md)。

務必留意：手動確認時「領鑰狀態」與「通知狀態」兩欄位必須同時更新，只改其中一項會導致系統誤判並重複發送 Teams 通知，或已確認的紀錄仍被視為待通知。

## 五、名詞與欄位對照

| 欄位 | 說明 |
|---|---|
| 預約唯一鍵 | 資源信箱 + Outlook Event ID 組成，系統用來判斷新增或更新的唯一依據 |
| 是否整天 | 是否為整天借用，影響通知時間計算規則 |
| 預計通知時間 | 系統自動計算，Teams 通知實際發送依據此欄位 |
| 通知狀態 | 未通知／已通知／不需通知／通知失敗 |
| 領鑰狀態 | 待通知／未完成填寫／已完成確認／已領鑰／已取消／已失效 |
| 事件同步狀態 | 有效／已異動／已取消／已失效／重複異常 |
| 重複資料檢查結果 | 正常／疑似重複（同一預約唯一鍵重複，見 SG-T09）／異常（同車時段重疊，見 FT-113） |
| 承辦人備註 | 供承辦人記錄手動確認方式、時間或其他備註，供後續稽核追溯 |

## 六、相關文件

- [docs/admin-manual-confirmation-procedure.md](admin-manual-confirmation-procedure.md) — 手動確認詳細操作程序
- [docs/incident-handling-guide.md](incident-handling-guide.md) — 異常處理說明
- [docs/maintenance-guide.md](maintenance-guide.md) — 日常維護說明
- [docs/system-safeguards.md](system-safeguards.md) — 系統防呆設計原理
- [test-cases/go-live-acceptance-record.md](../test-cases/go-live-acceptance-record.md) — 上線前驗收紀錄
