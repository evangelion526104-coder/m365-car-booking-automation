# 循環預約功能完整實作 — 規劃文件（#136）

更新日期：2026-08-31
對應決策：docs/company-account-and-booking-rules-change-decisions.md 第五節「循環預約功能」
文件狀態：規劃階段，尚未開始實作，待專案負責人確認後進入 Phase 0

## 一、背景與範圍

依據決策文件第五節，已用 Graph API 實測確認：

- Office 365 Outlook 連接器（標準，O365 E1 授權）不支援 calendarView，這是 Microsoft 官方建議的循環展開方式，但不在連接器允許呼叫的物件清單內。
- 現行『公務車行事曆同步至SharePoint』流程使用的 `/calendar/events?$filter=...` 端點，對循環預約只會回傳未展開的 seriesMaster 物件（代表整個系列的定義，start/end 僅為系列第一天），不會自動展開成每一天的實際發生日（occurrence）。
- 真實案例驗證：Michael Chiu 平日循環借車，目前系統只抓到系列第一天，其餘所有工作日的借用完全抓不到，既不會同步至 SharePoint，也不會觸發 Teams 通知。

決策範圍（一次完整實作，不分階段拆分循環類型）：

- 循環類型：每日、每週（含指定星期幾）、每月／數月（固定日期，interval 可大於 1）
- 自行解析 Graph 回傳的 `recurrence` 規則物件，在 Power Automate 流程中手動計算每個 occurrence 的日期
- 逐日建檔、逐日發卡（沿用 v0.3.5 已拍板的「不自動延用、每日各自確認共乘人數與規範」設計，本次不重新討論）
- 支援系列中「單日修改」「單日取消」的同步

## 二、核心策略：視窗限定展開，而非全系列展開

關鍵簡化：現行同步流程本來就是每 15 分鐘、以固定滾動視窗（往前 24 小時～往後 14 天，v0.3.7 #133 已調整）重新查詢一次，不是一次性建檔。因此**不需要展開循環系列的全部生命週期**（可能是無限期 noEnd 或上百次 numbered），只需要針對視窗內最多約 15 個候選日期，逐一判斷該日期是否命中循環規則即可。

具體做法：以視窗內每個候選日期，對照 seriesMaster 的 `recurrence.pattern`（type、interval、daysOfWeek、dayOfMonth、firstDayOfWeek）與 `recurrence.range`（type、startDate、endDate/numberOfOccurrences、recurrenceTimeZone）逐一比對是否命中：

| 循環類型 | 命中條件 |
|---|---|
| daily | 候選日期與 range.startDate 相差天數是否為 interval 的倍數 |
| weekly | 候選日期的星期幾是否在 daysOfWeek 內，且候選日期所在週與 range.startDate 所在週相差週數是否為 interval 的倍數 |
| absoluteMonthly | 候選日期的日（day of month）是否等於 dayOfMonth，且候選日期與 range.startDate 相差月數是否為 interval 的倍數 |

range.type 為 endDate 或 numbered 時，額外檢查候選日期是否超出 range.endDate，或該候選日期是第幾次發生、是否已超過 numberOfOccurrences；range.type 為 noEnd 時不需特別處理（視窗本身已限制在 14 天內，天然安全）。

此策略把「展開循環系列」問題，簡化成「固定約 15 個候選日期各自判斷是否命中」的固定次數迴圈，可用 Power Automate 既有的 Apply to each + 條件判斷／運算式實作，不需要額外連接器、Premium 授權或自訂程式碼元件。

## 三、單日例外（修改／取消）處理

Graph 對循環系列的單日修改／取消，是以獨立的 `type = exception` 事件物件呈現，並帶有 `seriesMasterId` 指回原系列。這類物件理論上會與 seriesMaster 一併被現行 `/events?$filter=...` 查詢撈到（因為其自身 start/end 落在查詢時間窗內）。

規劃邏輯：

1. 先將本次查詢結果依 `type` 分三類：`singleInstance`（一般單次借用，沿用現行邏輯不變）、`seriesMaster`（循環系列定義）、`exception`（單日例外）。
2. 對每個 `seriesMaster`，先用第二節的視窗限定展開法算出候選命中日期清單。
3. 用 `exception` 物件的 `seriesMasterId` 比對回其所屬系列；再比對日期，將該候選日期標記為「已被例外覆蓋」——
   - 若例外為修改（時間、主旨變動），則以例外物件本身的實際時間建檔，取代原本用規則算出的推算時間。
   - 若例外為取消，該天不建檔，或將既有 SharePoint 紀錄標記為已取消（沿用 v0.2.15 既有取消偵測邏輯的寫法）。
4. 其餘沒有被例外覆蓋的候選日期，才用系列規則推算出的時間建檔。

**本節目前僅為規劃，尚未經真實 Graph API 回應驗證**，需要在正式撰寫流程邏輯前，先建立一個小型測試循環預約（含一天修改、一天取消），實際呼叫 Graph API 檢視回傳 JSON，確認：

- `exception` 物件的實際欄位名稱與結構
- 「取消單日」的實際回傳行為——是完全從查詢結果消失，還是以 `isCancelled` 等欄位標記出現在結果中

此驗證方式比照專案過去每個階段的既定慣例（例如 v0.2.7～v0.2.11 針對 ATA-9627 事件讀取的實測驗證），先以真實 API 回應為準，不依賴文件推測或記憶中的 API 行為直接編碼。

## 四、資料結構變更

- **預約唯一鍵**：現行為「資源信箱 + 行事曆事件 ID」。循環系列所有 occurrence 共用同一個系列事件 ID，若沿用現行鍵值，同一系列的每一天會被視為同一筆、彼此覆蓋寫入。需改為「資源信箱 + 系列事件 ID（或 occurrence／exception 自身 ID）+ occurrence 日期」的組合鍵，確保每天各自是獨立一筆 SharePoint 紀錄。
- 現行「事件取消偵測」補強邏輯（v0.2.15）是逐鍵值比對「本次讀取到的鍵值集合」與「既有未取消紀錄」，鍵值改為複合鍵後，此邏輯預期可直接沿用、不需重新設計，但需以測試驗證單日取消情境下是否正確觸發。
- 新增 SharePoint 欄位（暫定，待確認畫面呈現需求後定案）：
  - 是否為循環預約（是／否）
  - 所屬系列事件 ID（供承辦人後台辨識、追蹤同一系列所有日期）
- 「建立項目」「更新項目」動作的借用起訖時間，循環情境下改用計算出的 occurrence 實際時間，而非 seriesMaster 本身的起訖時間（僅代表系列第一天）。

## 五、對其他流程的影響

- 『公務車借用前Teams通知與回覆』流程：不需要變更。此流程本來就是逐筆 SharePoint 紀錄獨立處理，只要同步流程正確地把每個 occurrence 拆成獨立一筆，通知流程會自然地逐日各自發卡（符合 v0.3.5 決策），無需額外開發。
- 既有防呆欄位與機制（是否測試資料、Concurrency Control、SG-T09 重複資料偵測、FT-113 同車時段重疊偵測）沿用不變，惟需以循環情境重新跑一次既有測試案例，確認複合鍵不影響既有比對邏輯。

## 六、分階段實作計畫

| 階段 | 內容 | 產出 |
|---|---|---|
| 0 | Graph API 驗證 spike：建立含「一天修改、一天取消」的測試循環預約，實際呼叫 API 確認 exception／取消回傳樣態 | 驗證紀錄文件 |
| 1 | 視窗限定展開邏輯：以獨立測試流程，針對 daily／weekly／absoluteMonthly 三種規則試算候選命中日期 | 測試流程＋驗證結果 |
| 2 | 例外覆蓋邏輯：整合 Phase 0 驗證結果，處理單日修改／取消 | 測試流程擴充 |
| 3 | 複合鍵與 SharePoint 欄位調整 | SharePoint schema 變更 |
| 4 | 整合進正式『公務車行事曆同步至SharePoint』流程（先在關閉排程狀態下開發測試） | 正式流程修改 |
| 5 | 端到端測試：依決策文件第六節，涵蓋系列、Occurrence、單日例外完整情境 | 測試紀錄，更新 test-cases/function-test-plan.md |
| 6 | 文件更新與 GitHub 同步 | README／CHANGELOG／todo／project-master-record.md |

## 七、風險與未決事項

- relativeMonthly（例如「每月第三個星期二」）與 yearly 循環規則不在本次決策範圍內。若真實資料出現此類規則，規劃在解析階段明確標記為「未支援循環類型」並提示承辦人另行人工處理，而非嘗試靜默解析，避免算錯日期造成漏同步或誤同步。
- dayOfMonth 遇到月份實際天數不足（例如設定 31 號，但當月只有 28～30 天）時，Graph API 的實際對應行為需於 Phase 0 一併驗證，不預先假設。
- Power Automate 標準連接器沒有程式碼執行元件，所有規則比對都需以巢狀運算式／多個 Compose 動作組成，複雜度較高；規劃拆解為多個具名 Compose 步驟以利除錯與維護，避免單一超長運算式難以排查錯誤（比照專案過去除錯經驗，例如 v0.2.14、v0.3.3 皆因單一複雜運算式或欄位參照錯誤耗費大量除錯時間）。
- 本文件為規劃階段產出，尚未實際修改任何 Power Automate 流程或 SharePoint 欄位；待專案負責人確認本規劃方向後，再進入 Phase 0 實際驗證工作。
