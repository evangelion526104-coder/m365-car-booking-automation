# 循環預約功能完整實作 — 規劃文件（#136）

更新日期：2026-09-15
對應決策：docs/company-account-and-booking-rules-change-decisions.md 第五節「循環預約功能」
文件狀態：Phase 1 驗證已完成（動態視窗、daily、absoluteMonthly、分頁限制）；Phase 2 Stage 2（例外覆蓋邏輯）技術設計已完成、Stage 3（SharePoint欄位）已完成，准備進入 Stage 4 正式整合實作

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

**（2026-09-08 更新）此節邏輯建議視為備援方案**：見第三節 Phase 0 驗證結果第 5 點，`events/{id}/instances` 端點已證實可由 Graph API 伺服器端直接完成完整展開（含例外覆蓋），Phase 1 應優先評估直接採用該端點，本節手動比對邏輯僅在該端點因故不可用時才需要實作。

## 三、單日例外（修改／取消）處理

Graph 對循環系列的單日修改／取消，是以獨立的 `type = exception` 事件物件呈現。這類物件理論上會與 seriesMaster 一併被現行 `/events?$filter=...` 查詢撈到（因為其自身 start/end 落在查詢時間窗內）。

規劃邏輯：

1. 先將本次查詢結果依 `type` 分三類：`singleInstance`（一般單次借用，沿用現行邏輯不變）、`seriesMaster`（循環系列定義）、`exception`（單日例外）。
2. 對每個 `seriesMaster`，先用第二節的視窗限定展開法算出候選命中日期清單。
3. 將呼叫 `events/{id}/instances` 時代入的 seriesMasterId，對應回其所屬系列；再比對日期，將該候選日期標記為「已被例外覆蓋」——
   - 若例外為修改（時間、主旨變動），則以例外物件本身的實際時間建檔，取代原本用規則算出的推算時間。
   - 若例外為取消，該天不建檔，或將既有 SharePoint 紀錄標記為已取消（沿用 v0.2.15 既有取消偵測邏輯的寫法）。
4. 其餘沒有被例外覆蓋的候選日期，才用系列規則推算出的時間建檔。

### Phase 0 驗證結果（2026-09-01 補充，2026-09-01 再更新，2026-09-08 完成驗證）

1. **connector 限制再次確認**：以獨立測試流程新增「Office 365 Outlook－傳送 HTTP 要求」動作，URI 指向 `.../room_nhb4_car@alp.global/calendarView?startDateTime=...&endDateTime=...`，實際執行後動作失敗，錯誤訊息為：「URI path is not a valid Graph endpoint...Invalid resource,Allowed values: me,users. Invalid Object,Allowed values: messages,mailFolders,events,calendar,calendars,outlook,inferenceClassification.」，證實此連接器的 Object 白名單本身不含 calendarView，與第一節既有結論一致，並非暫時性錯誤或設定錯誤。
2. **替代路徑已實測確認可行**：將測試流程的 URI 改為 `/users/room_nhb4_car@alp.global/events/{seriesMasterId}/instances?startDateTime=...&endDateTime=...`（Graph 官方取得單一循環系列展開實例的端點），先以假的佔位 ID（非真實系列 ID）執行以驗證連接器路徑層級是否放行。實際執行結果：動作失敗，但錯誤已改為 Graph API 本身回傳的 400 `ErrorInvalidIdMalformed`（「The Id is invalid.」），而非連接器層級的「Invalid Object」白名單錯誤。這證實請求已通過連接器的路徑驗證、確實送達 Graph API，僅因測試用的假 ID 格式不正確才在 Graph 端被拒絕。結論：此端點路徑前綴符合白名單中的 `events`，可作為不依賴 calendarView 取得展開後 occurrence 清單的替代方案。
3. **真實 seriesMasterId 取得與 instances 端點端到端驗證（2026-09-08）**：於 room_nhb4_car@alp.global 建立新測試循環系列「循環預約Phase0測試v3」（每週六、日，2026-09-19～2026-09-27，共 4 次發生）。註：先前測試（每日、23:00-23:30，2026/9/8-9/12）曾遭資源信箱自動拒絕，經檢視實際拒絕通知信確認為與 Michael Chiu 既有連續多日借用衝突所致，屬正常防呆行為，非連接器或程式問題；改用假日時段後正常被接受，排除此疑慮。透過測試流程呼叫 `/users/room_nhb4_car@alp.global/events?$top=50&$select=id,subject,type,seriesMasterId,start,end`（注意：`$filter` 內含中文字元會導致連接器回傳 `Request headers must contain only ASCII characters`，故改為取前 50 筆後以主旨關鍵字篩選）取得該系列 `seriesMaster` 物件真實 `id`，再以「篩選陣列」＋ `concat`／`first()` 運算式組成第二個 HTTP 要求動作之 URI（避免手動複製貼上長 base64 ID 的操作風險），呼叫 `/users/room_nhb4_car@alp.global/events/{id}/instances?startDateTime=2026-09-01T00:00:00Z&endDateTime=2026-10-15T00:00:00Z&$select=id,type,start,end,subject`，執行成功（HTTP 200）。修改與取消前，回傳 4 筆皆為 `type: "occurrence"`。
4. **exception／取消真實回傳樣態（2026-09-08）**：將該系列 9/19 該次由 10:00-10:30 修改為 11:00-11:30（僅此活動），並將 9/26 該次直接取消（僅此活動），重新呼叫上述 `/instances` 端點後確認：
   - **修改（時間異動）**：該筆物件的 `type` 由 `occurrence` 變為 `exception`，`start`／`end` 欄位直接反映異動後的新時間（本例為 `2026-09-19T03:00:00 UTC`／`2026-09-19T03:30:00 UTC`，對應台北時間 11:00-11:30），而非原排程時間；`id`、`subject` 等其餘欄位維持不變。實際回傳範例（`$select` 已限定欄位）：
     ```json
     {
       "id": "AAMkADRjMDYw...(略)",
       "subject": "Tina Yang 楊婉婷 循環預約Phase0測試v3-請忽略",
       "type": "exception",
       "start": { "dateTime": "2026-09-19T03:00:00.0000000", "timeZone": "UTC" },
       "end": { "dateTime": "2026-09-19T03:30:00.0000000", "timeZone": "UTC" }
     }
     ```
   - **取消**：該筆物件完全從 `/instances` 回傳的 `value` 陣列中消失，不會以 `isCancelled` 或任何欄位標記的方式出現在結果中——確認本節原先列出的兩種可能行為（消失 vs. 欄位標記）中，屬於前者「完全消失」。
   - 補充：本次 `$select` 僅取 `id,type,start,end,subject`，故未確認 exception 物件是否含其他欄位（例如原始時間 originalStart）；若後續實作需要此類欄位，需另行以完整欄位（不加 `$select`）呼叫驗證。
5. **結論與對前四節的影響**：
   - `/events/{id}/instances` 端點由 Graph API 伺服器端完成完整的循環展開（含例外覆蓋），只要先用現行 `/events?$filter=...`（或不帶中文的 `$select` 查詢）撈到 `seriesMaster` 物件取得其 `id`，再用該 `id` 呼叫 `/events/{id}/instances`（帶入與現行同步流程一致的 -24hr～+14天視窗），即可直接取得視窗內所有已展開的 occurrence／exception，**不需要在 Power Automate 中自行解析 `recurrence.pattern`／`recurrence.range` 規則**（第二節手動比對邏輯降級為備援方案，見第二節末段更新）。
   - 第三節原規劃「用 exception 物件的 seriesMasterId 比對回其所屬系列」需修正：`/events/{id}/instances` 回傳的 exception 物件本身**不含** `seriesMasterId` 欄位（因為呼叫時已針對特定系列查詢，不需要此欄位反查）；比對回系列的邏輯應改為以「呼叫時使用的 seriesMasterId」為準，而非依賴回傳物件自帶欄位（已反映於本節第 3 點文字）。
   - 取消偵測仍需注意：由於取消的 occurrence 會完全從 `/instances` 結果消失，現有「事件取消偵測」邏輯（比對本次讀取鍵值集合 vs. 既有未取消紀錄）預期可直接沿用；但因為是「整個系列中某一天」消失而非整個系列 ID 消失，第四節複合鍵必須包含 occurrence 日期，否則可能誤判整個系列已取消。

此驗證方式比照專案過去每個階段的既定慣例（例如 v0.2.7～v0.2.11 針對 ATA-9627 事件讀取的實測驗證），以真實 API 回應為準，不依賴文件推測或記憶中的 API 行為直接編碼。測試資料（v3 測試循環系列）已於驗證完成後清理；測試用 Power Automate 流程「公務車功能測試-ATA9627事件讀取」已確認關閉，不會持續排程執行。

### Phase 1 驗證結果（2026-09-08）

沿用同一支獨立測試流程「公務車功能測試-ATA9627事件讀取」，改用與現行『公務車行事曆同步至SharePoint』正式流程完全一致的動態視窗（`addHours(utcNow(),-24)` ～ `addDays(utcNow(),14)`，v0.3.7 #133 已定案之範圍）呼叫 `/events/{id}/instances`，針對 Phase 0 尚未驗證的項目逐一實測：

1. **分頁限制（新發現，重要風險，須寫入正式實作）**：`/events/{id}/instances` 端點預設每頁僅回傳 10 筆，超過 10 筆的視窗會在回應中夾帶 `@odata.nextLink` 進行分頁；若未處理，會導致視窗內同一系列超過 10 筆的 occurrence 被靜默截斷、部分日期漏同步。以每日循環系列驗證（2026-09-08～2026-09-20，共 13 次，落在 -24hr～+14天視窗內）：不加 `$top` 參數時僅回傳 10 筆且含 `nextLink`；加上 `$top=100` 後，13 筆全數於單一頁面回傳、無 `nextLink`。**結論：正式流程實作時，`/events/{id}/instances` 呼叫必須加上 `$top=100`（或依實際最大可能 occurrence 數評估更大值），否則單一系列在視窗內 occurrence 數超過 10 筆時會漏同步。**
2. **daily 循環類型（正式動態視窗）**：每日循環系列（interval=1，range.type=endDate，range.endDate=2026-09-20）於動態視窗下，回傳結果與預期完全相符：13 筆皆為 `type: occurrence`，日期自 9/8 至 9/20 連續無缺漏，最後一筆 start/end 日期正確對應系列的 `range.endDate`。
3. **absoluteMonthly 循環類型（含月份天數不足案例）**：以 dayOfMonth=31、range.type=numbered、numberOfOccurrences=6 的系列驗證（起始 2026-09-08，涵蓋 2026-09～2027-02），確認 Graph 伺服器端會自動將超出當月天數的 `dayOfMonth` 裁切為當月最後一天，6 筆結果全數為 `type: occurrence`、無分頁：
   - 2026-09 → 30 日（9月僅30天，裁切）
   - 2026-10 → 31 日（10月31天，不裁切）
   - 2026-11 → 30 日（11月僅30天，裁切）
   - 2026-12 → 31 日（12月31天，不裁切）
   - 2027-01 → 31 日（1月31天，不裁切）
   - 2027-02 → 28 日（2027非閏年，2月僅28天，裁切）

   結論：`dayOfMonth` 超出當月天數時，由 Graph API 伺服器端自動裁切至當月最後一天，Power Automate 端不需自行處理此邊界情況，可直接信任 `/instances` 回傳結果（呼應第七節原列風險，該項風險已解除）。
4. **建立循環預約時 `patternedRecurrence.range` 的必填欄位（實作細節，非 Graph 查詢行為，但實測中發現值得記錄）**：以程式（HTTP 動作直接 POST）建立測試用循環預約時發現，`range.type` 為 `numbered` 時，即使邏輯上不需要明確的結束日期，Graph API 仍要求 `range` 物件內必須包含 `startDate` 欄位，否則回傳 400 BadRequest（「The recurrence start date is too early.」）。此為建立循環預約時的必填欄位限制，與 Phase 1 展開／查詢邏輯本身無關，先記錄備查。

以上兩組測試資料（daily、absoluteMonthly 測試系列）已於驗證完成後清理；測試流程「公務車功能測試-ATA9627事件讀取」已再次確認關閉排程、暫存的查詢/測試動作已還原或移除。

### Stage 2 技術設計：例外覆蓋邏輯與複合鍵組成公式（2026-09-15）

本節先以文字規格記錄 Stage 2（例外覆蓋邏輯整合）的實作設計，供 Stage 4 整合進正式流程時直接採用，避免在一次性測試流程中重複搭建、拆除相同邏輯。設計內容完全基於第三節 Phase 0、Phase 1 已用真實 API 回應驗證過的行為，不新增未經驗證的假設。

1. **判斷是否需要展開**：正式流程逐筆處理 `/events?$filter=...` 回傳結果時，先依 `type` 分流——`singleInstance` 沿用現行邏輯不變（是否為循環預約＝否，所屬系列事件ID留空）；`type = seriesMaster` 才進入以下展開步驟；`type = exception` 若被現行查詢直接撈到，暫不單獨處理（因其必然屬於某個已展開的 seriesMaster，會在該系列展開時一併涵蓋，直接略過以避免重複建檔）。

2. **展開呼叫**：對每個 `seriesMaster`，以其 `id` 呼叫：
   `GET https://graph.microsoft.com/v1.0/users/{資源信箱}/events/{seriesMasterId}/instances?startDateTime={視窗起始}&endDateTime={視窗結束}&$top=100&$select=id,type,start,end,subject`
   視窗起始/結束沿用正式流程既有的動態視窗算式（`addHours(utcNow(),-24)` ～ `addDays(utcNow(),14)`，v0.3.7 #133 已定案）。**`$top=100` 為必要參數**，否則超過 10 筆會被分頁截斷（見 Phase 1 驗證結果第 1 點）。

3. **回應結構（剖析 JSON 用 schema）**：
   ```json
   {
     "type": "object",
     "properties": {
       "value": {
         "type": "array",
         "items": {
           "type": "object",
           "properties": {
             "id": { "type": "string" },
             "type": { "type": "string" },
             "start": { "type": "object", "properties": { "dateTime": {"type": "string"}, "timeZone": {"type": "string"} } },
             "end": { "type": "object", "properties": { "dateTime": {"type": "string"}, "timeZone": {"type": "string"} } },
             "subject": { "type": "string" }
           }
         }
       }
     }
   }
   ```

4. **逐筆處理（套用至各項，遍歷 `body('剖析_JSON')?['value']`）**：
   - `occurrence` 與 `exception` 兩種 `type` 皆視為需要建檔的一天，**不需要分支處理不同建檔邏輯**——因為 Graph 伺服器端已經把最終時間算好（`occurrence` 是規則推算時間，`exception` 是異動後實際時間），流程只需直接採用該筆的 `start`／`end`，無需比對或合併兩種來源。
   - 借用起訖時間＝該筆 `start.dateTime`／`end.dateTime`（**取代** seriesMaster 本身的起訖時間，後者僅代表系列第一天）。
   - 複合鍵（取代現行「資源信箱＋行事曆事件ID」）：
     `concat(資源信箱, '|', seriesMasterId, '|', formatDateTime(item()?['start']?['dateTime'], 'yyyy-MM-dd'))`
     寫入「預約唯一鍵」欄位；`seriesMasterId` 一律使用呼叫 `/instances` 時所用的系列 `id`（即步驟 2 的來源），不依賴回傳物件自帶欄位（Phase 0 已確認 exception 物件本身不含 `seriesMasterId`，見第三節第 5 點）。
   - 新欄位寫入：「是否為循環預約」＝是；「所屬系列事件ID」＝該 seriesMasterId（供承辦人後台辨識同一系列所有日期，見第四節）。

5. **取消偵測沿用既有邏輯**：某一天被取消時，該筆物件會完全從 `/instances` 回傳的 `value` 陣列中消失（Phase 0 已確認，見第三節第 4 點），不會有 `isCancelled` 之類欄位可判斷。由於複合鍵已把 occurrence 日期包含在內，現行「比對本次讀取鍵值集合 vs 既有未取消紀錄」的取消偵測邏輯（v0.2.15）預期可直接沿用：某一天的複合鍵若本次未出現在讀取結果中，該天會被判定為已取消，而不會誤判整個系列都取消（因為同系列其他日期的複合鍵仍會正常出現在本次讀取結果中）。此點仍需在 Stage 5 端到端測試中以真實情境驗證一次。

6. **與第二節手動比對邏輯的關係**：本設計完全採用 `/events/{id}/instances` 端點展開，第二節「視窗限定展開」的手動規則比對邏輯（自行解析 `recurrence.pattern`／`recurrence.range`）維持第二節末段已註記的「備援方案」定位，正式實作不會使用到，僅在該端點未來若因故不可用時才需要啟用。

### Stage 4 實作筆記：正式流程現況逐動作記錄與修改對照（2026-09-15）

為避免在正式流程副本（`複本 - 公務車行事曆同步至SharePoint`，flow id `2780d22d-d33c-42f4-9db7-60a12c1f2b6d`，狀態關閉，已於本次工作階段建立）中反覆重新探索既有邏輯，本節逐一記錄目前每個相關動作的實際運算式（透過「預覽程式碼」逐一確認，非猜測），以及 Stage 4 需要的精確修改點。

**目前巢狀結構**（`套用至各項` = 逐車輛，`套用至各項_1` = 逐事件）：

```
套用至各項（車輛，item()=車輛紀錄）
  設定變數 → 傳送 HTTP 要求（呼叫 /calendar/events?$filter=...，見下方 URI）→ 剖析 JSON
  套用至各項_1（事件，item()=單一事件，可能是 singleInstance/seriesMaster/exception）
    編輯（組合鍵）→ 附加至陣列變數 → 編輯_1（通知時間）
    → 取得多個項目（比對既有紀錄）→ 取得多個項目_2（同車其他未取消紀錄）
    → 篩選陣列（時段重疊）→ 條件（既有紀錄長度>0？是→更新項目／否→建立項目）
    → 條件_2（重疊筆數>1？是→套用至各項_3／否→無動作）
  取得多個項目_1 → 套用至各項_2（略，與 Stage 2 無關）
取得多個項目_3 → 套用至各項_4（取消偵測，比對本次讀取鍵值集合，與 Stage 2 無關、預期不需修改）
```

**現有精確運算式**（`套用至各項` 層級 item() = 車輛紀錄）：

- 「傳送 HTTP 要求」URI：
  `concat('https://graph.microsoft.com/v1.0/users/', item()?['mailbox'], '/calendar/events?$filter=start/dateTime ge ''', formatDateTime(addHours(utcNow(),-24),'yyyy-MM-ddTHH:mm:ss'), ''' and start/dateTime le ''', formatDateTime(addDays(utcNow(),14),'yyyy-MM-ddTHH:mm:ss'), '''')`

**現有精確運算式**（`套用至各項_1` 層級，item() = 單一事件，即目前的 seriesMaster/singleInstance 物件本身）：

- 「編輯」（組合鍵）：`concat(items('套用至各項')?['mailbox'], '|', item()?['id'])`
- 「編輯_1」（預計通知時間）：以 `item()?['isAllDay']`、`item()?['start']?['dateTime']` 計算整天／非整天通知時間
- 「取得多個項目」：SharePoint GetItems，`$filter: OData__x9810...('預約唯一鍵') eq '@{outputs('編輯')}'`
- 「取得多個項目_2」：`$filter: 資源信箱 eq items('套用至各項')?['mailbox'] and 已取消 eq 0 and 預約唯一鍵 ne outputs('編輯')`
- 「篩選陣列」：`from = body('取得多個項目_2')?['value']`，`where = and(less(其他紀錄借用起始, formatDateTime(convertTimeZone(items('套用至各項_1')?['end']?['dateTime'],...))), greater(其他紀錄借用結束, formatDateTime(convertTimeZone(items('套用至各項_1')?['start']?['dateTime'],...))))`
- 「條件」：`length(body('取得多個項目')?['value']) 大於 0` → 是=更新項目，否=建立項目
- 「建立項目」／「更新項目」欄位對照（皆已含「是否為循環預約」「所屬系列事件ID」欄位但目前留空未寫值）：
  - 沿用 `items('套用至各項_1')`（事件層級固定資訊，不隨 occurrence 變動）：Title/使用事由（subject）、借用人姓名／Email（organizer.emailAddress）、是否整天（isAllDay）、iCalUId、事件最後修改時間（lastModifiedDateTime）
  - 目前用 `items('套用至各項_1')?['start'/'end']`、將於 Stage 4 改為使用 occurrence 層級時間：借用日期／起始／結束時間、原始開始／結束時間UTC、同車時段重疊檢查結果（經篩選陣列）、預計通知時間（經編輯_1）
  - 行事曆事件 ID：`items('套用至各項_1')?['id']`
  - 預約唯一鍵：`outputs('編輯')`
  - 資源信箱：`items('套用至各項')?['mailbox']`；車輛名稱：`items('套用至各項')?['name']`

**Stage 4 修改方案**（尚未實作，供下次工作階段依此直接施工）：

1. 在 `套用至各項_1` 最前面新增一個 Switch 動作，切換依據 `item()?['type']`：
   - case `singleInstance`：移入現有「編輯…條件_2」整條動作鏈（不需重寫，僅搬移），並在「建立項目」「更新項目」補上兩個新欄位：是否為循環預約＝否、所屬系列事件ID＝留空。
   - case `seriesMaster`：新建動作鏈（見下方第 2 點）。
   - default（含 `exception`，理由見第三節第 1 點）：不執行任何動作。
2. `seriesMaster` case 內新建：
   - 「傳送 HTTP 要求」新動作，URI：
     `concat('https://graph.microsoft.com/v1.0/users/', items('套用至各項')?['mailbox'], '/events/', item()?['id'], '/instances?startDateTime=', formatDateTime(addHours(utcNow(),-24),'yyyy-MM-ddTHH:mm:ss'), '&endDateTime=', formatDateTime(addDays(utcNow(),14),'yyyy-MM-ddTHH:mm:ss'), '&$top=100&$select=id,type,start,end,subject')`
   - 「剖析 JSON」，schema 見第三節 Stage 2 技術設計第 3 點。
   - 新的巢狀「套用至各項」（命名待定，下稱套用至各項_seriesInstances），來源 `body(新剖析JSON)?['value']`，此層級 item() = 單一 occurrence/exception。
   - 於此新迴圈內，使用 Power Automate「複製到我的剪貼簿」功能複製既有「編輯…條件_2」整條動作鏈並貼上，然後只需修改以下 6 處運算式（其餘欄位維持原樣即可，因為 items('套用至各項_1') 在此巢狀層級仍有效、可正確取回原始 seriesMaster 事件的固定資訊）：
     - 編輯（組合鍵）改為：`concat(items('套用至各項')?['mailbox'], '|', items('套用至各項_1')?['id'], '|', formatDateTime(item()?['start']?['dateTime'],'yyyy-MM-dd'))`
     - 編輯_1（通知時間）內的 `item()?['isAllDay']` 改為 `items('套用至各項_1')?['isAllDay']`（occurrence 物件無此欄位，沿用系列本身設定）；`item()?['start']?['dateTime']` 維持不變（此時已正確指向 occurrence）
     - 篩選陣列的 `items('套用至各項_1')?['start'/'end']` 改為 `item()?['start'/'end']`（改用 occurrence 本身時間，而非系列第一天）
     - 建立項目／更新項目：借用日期／起始／結束時間、原始開始／結束時間UTC 改為讀取 `item()?['start'/'end']?['dateTime']`；行事曆事件ID 改為 `item()?['id']`（occurrence 自身 id，而非系列 id）
     - 建立項目／更新項目新增：是否為循環預約＝是、所屬系列事件ID＝`items('套用至各項_1')?['id']`
     - 其餘欄位（Title/使用事由/借用人姓名/Email/是否整天來源/iCalUId/事件最後修改時間/資源信箱/車輛名稱）維持指向 `items('套用至各項_1')` 或 `items('套用至各項')`，不需更動。
3. 貼上後務必逐一用「預覽程式碼」確認 Power Automate 是否已自動把內部動作參照（如 `outputs('編輯')`、`body('取得多個項目')`）正確改指向複製後的新動作名稱，不可只憑畫面顯示判斷（比照本次除錯經驗，畫面顯示可能與實際值不一致）。
4. 條件_2／套用至各項_3（同車重疊>1筆的額外處理）預期隨複製動作一併帶入，不需特別修改，但仍需在 Stage 5 端到端測試中針對循環情境驗證一次。

## 四、資料結構變更

- **預約唯一鍵**：現行為「資源信箱 + 行事曆事件 ID」。循環系列所有 occurrence 共用同一個系列事件 ID，若沿用現行鍵值，同一系列的每一天會被視為同一筆、彼此覆蓋寫入。需改為「資源信箱 + 系列事件 ID（或 occurrence／exception 自身 ID）+ occurrence 日期」的組合鍵，確保每天各自是獨立一筆 SharePoint 紀錄。
- 現行「事件取消偵測」補強邏輯（v0.2.15）是逐鍵值比對「本次讀取到的鍵值集合」與「既有未取消紀錄」，鍵值改為複合鍵後，此邏輯預期可直接沿用、不需重新設計，但需以測試驗證單日取消情境下是否正確觸發（呼應第三節第 5 點）。
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
| 0 | Graph API 驗證 spike：建立含「一天修改、一天取消」的測試循環預約，實際呼叫 API 確認 exception／取消回傳樣態 | 驗證紀錄文件（本文件第三節，已完成） |
| 1 | 展開邏輯：直接採用 `events/{id}/instances` 端點（見第三節第 5 點與 Phase 1 驗證結果），確認 daily／absoluteMonthly 皆可正確展開，且已找出分頁必須加 `$top=100` 的實作要點 | 測試流程＋驗證結果（已完成，見第三節 Phase 1 驗證結果） |
| 2 | 例外覆蓋邏輯：整合 Phase 0 驗證結果，處理單日修改／取消 | 技術設計已完成（已完成，見第三節 Stage 2 技術設計），實作併入 Stage 4 |
| 3 | 複合鍵與 SharePoint 欄位調整 | SharePoint schema 變更 |
| 4 | 整合進正式『公務車行事曆同步至SharePoint』流程（先在關閉排程狀態下開發測試） | 正式流程修改 |
| 5 | 端到端測試：依決策文件第六節，涵蓋系列、Occurrence、單日例外完整情境 | 測試紀錄，更新 test-cases/function-test-plan.md |
| 6 | 文件更新與 GitHub 同步 | README／CHANGELOG／todo／project-master-record.md |

## 七、風險與未決事項

- relativeMonthly（例如「每月第三個星期二」）與 yearly 循環規則不在本次決策範圍內。若真實資料出現此類規則，規劃在解析階段明確標記為「未支援循環類型」並提示承辦人另行人工處理，而非嘗試靜默解析，避免算錯日期造成漏同步或誤同步。
- ~~dayOfMonth 遇到月份實際天數不足（例如設定 31 號，但當月只有 28～30 天）時，Graph API 的實際對應行為需於 Phase 1 一併驗證~~ **（已於 Phase 1 驗證解除，見第三節 Phase 1 驗證結果第 3 點）**：Graph 伺服器端會自動裁切至當月最後一天，Power Automate 端不需額外處理。
- **（Phase 1 新發現）`/events/{id}/instances` 端點分頁限制**：預設每頁僅回傳 10 筆，若視窗內單一系列 occurrence 數超過 10 筆，未加 `$top` 參數會透過 `@odata.nextLink` 分頁、造成靜默漏同步。Phase 2 正式整合時，所有呼叫此端點的動作皆須加上 `$top=100`（或更保守的更大值），並建議在文件/程式碼註解中明確標註此限制，避免日後有人在別處新增類似呼叫時重蹈覆轍。
- Power Automate 標準連接器沒有程式碼執行元件，若最終仍需採用第二節手動比對邏輯，所有規則比對都需以巢狀運算式／多個 Compose 動作組成，複雜度較高；規劃拆解為多個具名 Compose 步驟以利除錯與維護，避免單一超長運算式難以排查錯誤（比照專案過去除錯經驗，例如 v0.2.14、v0.3.3 皆因單一複雜運算式或欄位參照錯誤耗費大量除錯時間）。
- Graph API `$filter` 參數若包含中文等非 ASCII 字元，會被 Office 365 Outlook 連接器擋下（`Request headers must contain only ASCII characters`），Phase 1 實作時應避免在 `$filter`／URI 參數中直接使用中文，改以 `$select` 取回較少欄位、或於流程內以「篩選陣列」等動作在取得資料後於本地端進行文字比對。
- 本文件 Phase 0、Phase 1 驗證皆已完成；後續 Phase 2 起（例外覆蓋邏輯整合、複合鍵與 SharePoint 欄位調整、整合進正式同步流程）的實作工作，待專案負責人確認本次驗證結論與 Phase 2 方向後再進行。
