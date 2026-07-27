# CHANGELOG

本檔案記錄本專案的主要異動。最新 Master 狀態以 `docs/project-master-record.md` 為準。

## v0.2.15 - 新增事件取消偵測補強邏輯並驗證通過

* v0.2.14 追加更新的 P3 情境測試發現架構缺口：流程從未包含「偵測 Outlook 事件消失」的比對步驟，事件被刪除或取消後，SharePoint 紀錄不會被標記為已取消，後續通知也不會停止（FT-103／SG-T04）
* 設計並實作補強邏輯：每次執行以陣列變數記錄本次實際讀取到的所有事件唯一鍵，再對該車輛「未取消且借用起始時間落在同一讀取時間窗」的既有 SharePoint 紀錄逐筆比對，唯一鍵未出現在本次讀取結果中者，判定為對應 Outlook 事件已消失，更新為 `已取消 = 是`、`事件同步狀態 = 已取消`、`取消時間 = utcNow()`
* 儲存流程確認零錯誤，手動觸發測試並以 SharePoint REST API 驗證：既定測試案例（Id=4）正確標記為已取消；額外發現並正確標記一筆先前測試階段遺留、未被追蹤到的孤兒紀錄（Id=1，經 Graph API 查詢結果與 Outlook 網頁行事曆雙重確認其來源事件確實已刪除）；仍然有效的未來借用（Id=2、Id=3）未受影響
* FT-103／SG-T04 狀態由「未通過」更新為「通過」
* 詳見 `release-notes/v0.2.15.md`

## v0.2.14 - 修正「更新項目」時間欄位 +8 小時位移嚴重錯誤

* P3 情境測試過程中發現：SharePoint「更新項目」（Update，PATCH/MERGE）動作寫入借用起始/結束時間、預計通知時間、事件最後修改時間時，比「建立項目」（Create，POST）多了 8 小時（Taipei 時區 UTC 位移量），即使兩者運算式完全相同、來源資料完全相同
* 以測試紀錄連續三次比對（建立時正確 → 第一次自動更新後 +8 小時 → 第二次自動更新後維持不變）證實為固定位移，並以剖析 JSON 原始輸出確認來源資料未變，鎖定成因為 SharePoint 連接器 Create／Update 對「無時區後綴的模糊時間字串」解讀不一致
* 修正「更新項目」動作：移除不必要的 `convertTimeZone` 轉換，改為直接輸出 UTC 原始時間並明確加上 `Z` 時區後綴；預計通知時間因參照與 Create 共用的 Compose 輸出，改為在引用處以 `addHours(...,-8)` 反推回 UTC 再補 `Z` 後綴，避免影響 Create 端既有正確行為
* 重新手動觸發測試並以 SharePoint REST API 驗證：借用起始/結束時間、預計通知時間皆恢復與 Create 當下一致的正確值，確認修正有效
* 本版本修正並補充 v0.2.13「判定：通過」結論之不足——該結論僅驗證了 Create 路徑，未涵蓋排程自動觸發的 Update 路徑
* 詳見 `release-notes/v0.2.14.md`

## v0.2.13 - 正式同步流程執行期錯誤修正並完成首次成功執行驗證

* v0.2.12 儲存驗證雖為零錯誤，但排程開啟後所有執行紀錄皆顯示失敗；本版本逐一排查並修正四種執行期錯誤：
  * 兩處 Apply to each 迴圈的 `foreach` 參數誤加大括號（`@{items(...)}`），型別被轉為字串，修正為 `@items(...)`
  * 「取得多個項目」篩選查詢誤用中文欄位顯示名稱，修正為 SharePoint REST API 驗證後的正確內部名稱 `OData__x9810__x7d04__x552f__x4e00__x93`
  * 「條件」判斷式的 `greater` 比較誤加大括號，修正為 `@length(...)`
  * 「建立項目」「更新項目」的「是否整天」欄位誤寫入中文字串，修正為直接參照原生布林值 `@items('套用至各項_1')?['isAllDay']`
* 修正後手動觸發測試，運行紀錄首次出現「測試成功」，並逐層展開執行追蹤確認三台車、內層事件迴圈、SharePoint 取得/建立/更新項目皆成功
* 以 SharePoint REST API 直接查詢清單，確認三筆借用紀錄正確寫入，「是否整天」欄位為真正布林值；後續排程再次執行後項目數未增加，確認防重複比對邏輯正常
* 詳見 `release-notes/v0.2.13.md`

## v0.2.12 - 建立正式公務車行事曆同步至SharePoint流程並啟用排程

* 完成正式流程 `公務車行事曆同步至SharePoint`（Flow ID `b6d1ec5c-0fc5-46c1-85b0-d8d68c72c0ce`）：Recurrence（15 分鐘）→ 車輛清單 → 套用至各項（三台車）→ 傳送 HTTP 要求 → 剖析 JSON → 套用至各項 1（逐筆事件）→ 計算預約唯一鍵／預計通知時間 → SharePoint 取得多個項目 → 條件 → 更新項目／建立項目
* 三台公務車共用同一組讀取、解析、寫入邏輯，依 `sharepoint/list-schema.md` 完整寫入約 40 個欄位（含系統防呆欄位預設值）
* 排除設計問題：Power Automate 動作／迴圈顯示名稱含空格時，從外部以名稱參照（`items()`、`outputs()`）會被轉換為底線，例如 `套用至各項 1` 需寫成 `items('套用至各項_1')`；修正「更新項目」約 13 個受影響欄位
* 流程儲存驗證零錯誤（「您的流程已準備就緒」），流程檢查程式錯誤 0、警告 0
* 已將流程狀態由「關閉」切換為「開啟」，正式進入每 15 分鐘排程運作
* `是否測試資料` 暫時維持 `是` 作為上線前安全預設值，待端到端測試通過後再切換
* 詳見 `release-notes/v0.2.12.md`

## v0.2.11 - Resource Calendar 讀取阻擋解除（HTTP 直連 Graph）

* IT 已將 `ad.general@alp.global` 對三台公務車 Resource Mailbox 的權限提升為 Mailbox Full Access
* 重測 `取得行事曆 (V2)` 與 `取得事件的行事曆檢視 (V3)`：兩者仍只能存取連線帳號自己的行事曆，判定為連接器動作限制，非權限問題
* 驗證 `取得電子郵件 (V3)` 的 `原始信箱地址` 參數可成功指定 Resource Mailbox 並讀信，確認 Full Access 對郵件類動作有效
* 找到正式解法：Office 365 Outlook 標準連接器（非 Premium）的「傳送 HTTP 要求」動作可直連 Graph，路徑限定於 `messages`、`mailFolders`、`events`、`calendar`、`calendars`、`outlook`、`inferenceClassification` 白名單物件
* 以 `GET /v1.0/users/room_nhb4_car@alp.global/calendar/events?$filter=...` 實測成功，取得完整借用紀錄（借用人、起訖時間、主旨、是否全天、iCalUId 等）
* 停用測試流程 `公務車功能測試-ATA9627事件讀取` 的每分鐘排程（先前已知待辦，本次確認已完成）
* M5 Resource Calendar 事件讀取正式解除阻擋，可進入 `公務車行事曆同步至 SharePoint` 正式建置階段
* 詳見 `release-notes/v0.2.11.md`

## v0.2.10 - ATA-9627 候選 GUID V3 實測未通過

* 2026-07-18 於 `公務車功能測試-ATA9627事件讀取` 完成真正的 GUID 輸入實測
* 執行紀錄 `08584172227294871773330429620CU04` 確認 Calendar Id 已送出為 `6049e1d1-b34c-4cca-b530-2c7c4b77abe9`
* Office 365 Outlook V3 回傳 `400 BadRequest / ErrorInvalidIdMalformed / The Id is invalid.`
* 判定此值為候選物件 GUID，不是 V3 連接器可接受的 Calendar ID；未出現 Access Denied
* CAL-V3-06 標記為未通過，CAL-V3-07 與 CAL-V3-08 因未取得事件而阻擋
* 正式 SharePoint 同步仍不得啟用；下一步為取得連接器可接受的 Calendar ID 或評估 O365 E1 標準替代方案

## 文件修正 - 瀏覽器操作判定規則

* 修正 `docs/calendar-access-test.md` 中已過時的 Codex 側邊欄登入快取敘述
* 確認 Codex 內建瀏覽器與 Chrome 均可作為 Power Automate 操作後端
* 新增瀏覽器與電腦操作規則，禁止僅依歷史紀錄推定目前瀏覽器不可用
* 明確區分專案文件規範與 Codex 執行環境的 Browser Plugin、Computer Use、App Approval 權限

## v0.2.9 - ATA-9627 V3 測試實作版

* 新增 `power-automate/ata9627-v3-test-implementation.md`
* 將 `公務車功能測試-ATA9627事件讀取` 的 Power Automate 動作、命名、欄位與 Expression 寫成可照做版本
* 明確定義 V3 測試成功、部分成功與失敗時的判斷方式
* 明確規定 V3 成功前不得進入正式 SharePoint 同步流程
* 新增成功後下一步 `公務車行事曆同步至 SharePoint - ATA9627 MVP` 草案
* 更新 README、Master、里程碑、完成清單、待辦、Power Automate 說明與測試紀錄

## v0.2.8 - ATA-9627 Calendar ID 取得

* 取得 ATA-9627 公務車真正 Calendar ID：`6049e1d1-b34c-4cca-b530-2c7c4b77abe9`
* 將 V3 事件讀取測試值由資源信箱 Email 改為真正 Calendar ID
* 更新 README、Master、里程碑、待辦、Power Automate 說明與測試紀錄
* 明確標示 V3 事件讀取仍待實際執行，不得先標記為通過
* 下一步為執行 `公務車功能測試-ATA9627事件讀取`，確認是否可取得 subject、start、end、Event ID 與 iCalUId

## v0.2.6 - Outlook 共用行事曆加入後重測

* 確認 `ad.general@alp.global` 的 Outlook 網頁可顯示三台公務車共用行事曆
* 當時曾切換瀏覽器後端確認 Power Automate 詳細資料、連線與 28 天執行歷程可正常載入；此項僅為該次測試紀錄，不代表 Codex 內建瀏覽器目前不可用
* 重新執行 `取得行事曆 (V2)`，輸出仍只回傳一個個人 `Calendar`
* 確認三台 Resource Mailbox 尚未出現在 Power Automate Calendar 清單，仍無法取得 Calendar ID
* 將下一步調整為評估 Editor 權限或其他不使用 Premium 連接器的標準讀取方案
* 更新 README、Master、里程碑、待辦、Power Automate 說明與測試紀錄

## v0.2.5 - Exchange 管理角色存取驗證

* 以 `ad.general@alp.global` 開啟 Exchange 管理中心並直接驗證收件者信箱管理頁
* 確認目前帳號只具備「我的帳戶」自助管理介面，沒有收件者／信箱管理功能
* 確認無法由目前帳號變更三台資源行事曆權限
* 將 M5 阻擋條件明確更新為「需 Exchange 管理員登入或代為執行權限設定」
* 更新 Master、里程碑、待辦、權限操作單與測試紀錄

## v0.2.4 - Outlook 資源行事曆存取閘門實測

* 建立 `公務車功能測試-資源信箱讀取`，Flow ID：`7aff726f-0f9d-4b7e-a128-baeb368ab1ce`
* 確認 Office 365 Outlook 連接器登入正常，但 Calendar ID 選單只有個人 `Calendar`
* 實測直接使用 `room_nhb4_car@alp.global` 作為 Calendar ID，流程回傳「ID 格式不正確」
* 將測試流程關閉，避免排程持續產生失敗執行
* 新增 Exchange 資源行事曆 Reviewer 權限設定與驗證文件
* 更新 Master、里程碑、完成清單、待辦、Power Automate 說明與測試紀錄
* M5 尚未完成；待三台資源行事曆出現在 Calendar ID 選單後重測

## v0.2.3 - SharePoint 防呆欄位建置完成

* 已在 SharePoint `公務車借用管理` 清單實際新增並驗證系統防呆欄位
* 新增 Event ID / iCalUId、事件同步、卡片狀態、通知狀態、Flow 執行鎖與重複資料檢查相關欄位
* 新增 `已取消`、`已失效` 防呆旗標欄位
* 更新 Master 專案紀錄、SharePoint 欄位設計、完成清單、待辦事項、里程碑與 release note
* 目前仍待完成正式 Power Automate 同步、Teams 通知回覆與 SG-T01 至 SG-T10 實測

## System Safeguards

* 新增系統防呆設計正式文件
* 新增 Asia/Taipei 統一時區規則
* 新增 Event ID / iCalUId 唯一識別規則
* 新增 Outlook 取消與異動同步 SharePoint 規則
* 新增舊 Teams Adaptive Card 回覆前失效檢查
* 新增 Flow Concurrency Control 與重複資料防止規則
* 新增防呆欄位設計與測試案例
* 明確標示五項防呆機制為正式上線前必須完成

## Project Governance

* 新增 New Work Startup Rule
* 新增 Completion-driven Versioning
* 新增自動 Workflow
* 新增 Master 同步規則
* 新增 Branch 同步檢查
* 新增 GitHub 自動同步規則
* 新增固定回覆格式

## v0.1 - 專案版本控管規則建立

* 建立專案開發與版本控管規則
* 新增功能完成後標準流程
* 新增 Branch / Commit / Tag 規則
* 新增舊版本保留規則
* 新增 Codex 功能完成檢查清單

## [v0.2.0-master] - 2026-07-02

### 新增

- 建立 SharePoint List：`公務車借用管理`。
- 將正式建立位置調整為 `ALP_TW_AD` 站台。
- 新增完整 SharePoint 欄位設計，包含車輛、資源信箱、借用資訊、Teams 回覆、領鑰狀態、測試欄位。
- 新增 `是否整天` 欄位，用於判斷 Outlook 整天預約。
- 新增 `預計通知時間` 欄位，用於記錄 Power Automate 計算後的 Teams 預定通知時間。
- 建立 Power Automate 功能測試流程：`公務車功能測試-SharePoint清單連線`。
- 確認 SharePoint 標準連接器可讀取 `ALP_TW_AD` 清單。
- 確認 Office 365 Outlook 標準連接器可執行基本連線測試。
- 新增 Teams 通知時間規則：借用前 1 小時通知，但不得早於借用當天 08:00；整天借用一律 08:00 通知。
- 新增 Master 專案紀錄：`docs/project-master-record.md`。
- 新增專案里程碑：`docs/project-milestones.md`。
- 新增完成清單：`docs/completion-checklist.md`。
- 新增待辦事項：`docs/todo.md`。
- 新增 v0.2.0 release note：`release-notes/v0.2.0.md`。

### 修改

- 重寫 `README.md`，改為唯一最新專案總覽。
- 重寫 `sharepoint/list-schema.md`，更新為實際已建立欄位。
- 重寫 `power-automate/README.md`，補上目前已完成測試與待完成正式流程。
- 重寫 `adaptive-cards/README.md`，補上 Teams Adaptive Card 內容與 JSON 範例。
- 重寫 `test-cases/function-test-plan.md`，補上目前測試狀態與後續測試案例。

### 修正

- 修正原專案文件部分文字亂碼問題。
- 修正專案建立位置紀錄，從 `ALP_TW_AD-AD` 改為 `ALP_TW_AD`。
- 明確標示正式自動化尚未啟用，避免誤判為已完成上線。

### 尚未完成

- 尚未完成三台資源行事曆的正式讀取驗證。
- 尚未完成正式 Outlook 行事曆同步流程。
- 尚未完成 Teams Adaptive Card 正式發送與回覆寫回流程。
- 尚未完成端到端測試。

## [v0.1.2] - 2026-06-25

### 新增

- 建立初版版本控管與 GitHub Release 流程文件。
- 建立階段版本策略。
- 建立 Codex 功能完成檢查清單。

## [v0.1.1] - 2026-06-25

### 新增

- 補充版本命名與階段交付規則。
- 補充 README 內的開發階段說明。

## [v0.1] - 2026-06-25

### 新增

- 建立初始專案資料夾。
- 建立初版 SharePoint、Power Automate、Adaptive Card、測試文件骨架。
## v0.2.7 - ATA-9627 V3 Resource Calendar 事件讀取測試流程建立

* 紀錄 IT 已將 `ad.general@alp.global` 對 `room_nhb4_car@alp.global` 的 Calendar Folder Permission 調整為 Editor。
* 紀錄 Outlook 端已可看到 ATA-9627 行事曆，並可新增、修改、刪除測試事件。
* 紀錄 Power Automate `取得行事曆 (V2)` 執行成功但僅回傳 `ad.general@alp.global` 自身 Calendar。
* 明確判定 `取得行事曆 (V2)` 不能作為 Resource Mailbox 是否可讀取的最終判斷依據。
* 新增 `取得事件的行事曆檢視 (V3)` 直接讀取 ATA-9627 事件的測試流程設計。
* 新增 `docs/ata9627-v3-calendar-event-test.md`，作為下一階段 Power Automate 測試紀錄。
* 第一輪 Calendar Id 測試值設定為 `room_nhb4_car@alp.global`。
* 本版本尚未宣告 V3 實測成功，需待 Power Automate 執行後回填結果。
