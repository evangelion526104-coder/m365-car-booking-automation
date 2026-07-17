# CHANGELOG

本檔案記錄本專案的主要異動。最新 Master 狀態以 `docs/project-master-record.md` 為準。

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
