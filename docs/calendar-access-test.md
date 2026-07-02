# 公務車資源行事曆存取測試紀錄

測試日期：2026-07-02  
時區：Asia/Taipei  
分支：`feature/calendar-access-test`

## 測試目的

使用 `ad.general@alp.global` 的 Office 365 Outlook 標準連線，先執行 `取得行事曆 (V2)` 取得三台公務車真正 Calendar ID，再用 `取得事件的行事曆檢視 (V3)` 讀取未來 7 天事件。

## 已確認前提

- `ad.general@alp.global` 已具有三台 Resource Mailbox 的 Calendar Reviewer 權限。
- 本流程只讀取行事曆事件。
- 不修改 Resource Mailbox、Calendar Processing 或 Exchange 組態。
- 不需要 Exchange Administrator。
- 資源信箱 Email 不能直接當成 Calendar ID；先前 Altis 測試因此回傳「ID 格式不正確」。

## 測試資源

| 車輛 | Resource Mailbox | Calendar ID | 未來 7 天讀取 |
|---|---|---|---|
| Altis ATA-9627 B4-16 | `room_nhb4_car@alp.global` | 尚未取得 | 尚未執行 |
| Camry BKX-2370 B4-17 | `room_nhb4_car_camry@alp.global` | 尚未取得 | 尚未執行 |
| Cross BKY-0762 B4-44 | `room_nhb4_car_cross@alp.global` | 尚未取得 | 尚未執行 |

## 欄位驗證結果

| 欄位 | Altis | Camry | Cross |
|---|---|---|---|
| 主旨 | 待測 | 待測 | 待測 |
| 起始時間 | 待測 | 待測 | 待測 |
| 結束時間 | 待測 | 待測 | 待測 |
| 是否全天事件 | 待測 | 待測 | 待測 |
| 建立者／借用人 | 待測 | 待測 | 待測 |
| Event ID | 待測 | 待測 | 待測 |
| iCalUId | 待測 | 待測 | 待測 |

## 本輪實際執行情況

1. 已建立功能分支 `feature/calendar-access-test`。
2. 已嘗試開啟既有測試流程 `公務車功能測試-資源信箱讀取`（Flow ID：`7aff726f-0f9d-4b7e-a128-baeb368ab1ce`）。
3. 已以 `ad.general@alp.global` 的 Office 365 Outlook 標準連線成功執行 `取得行事曆 (V2)`；2026-07-02 15:34（Asia/Taipei）的執行結果為成功，動作耗時約 0.4 秒。
4. V2 實際輸出只有一筆 `Calendar`，擁有者為 `ad.general@alp.global`，未包含 Altis、Camry、Cross 三台資源行事曆。
5. 因三台資源行事曆未出現在輸出，現階段無法取得其 Calendar ID，也無法執行三台車的 `取得事件的行事曆檢視 (V3)`。
6. 測試流程曾為手動測試暫時開啟，確認結果後已於 2026-07-02 關閉，避免每分鐘重複執行。
7. 已取得使用者允許，下一步可在 `ad.general@alp.global` 的 Outlook 中明確加入三台共用行事曆後重測；本輪 Outlook 網頁未在內嵌瀏覽器完成載入，因此尚未完成加入動作。
8. 本輪未讀取或記錄任何實際借用事件內容，也未將待測欄位誤標為通過。

## 權限與連接器判定

- 不需要登入 Exchange Administrator，也不修改 Resource Mailbox、Calendar Processing 或 Exchange 組態。
- Calendar Reviewer 已足以讓 `ad.general@alp.global` 在 Outlook 查看資源行事曆，但本次實測證明它不會自動讓三台行事曆出現在 `取得行事曆 (V2)` 輸出。
- Microsoft 的 Office 365 Outlook 連接器限制說明指出，共用行事曆要出現在行事曆相關動作的 Calendar ID 清單中，通常需同時具備檢視與編輯權限。
- 因此後續先採最小變更：在 Outlook 明確加入三台共用行事曆並重測。若仍未列出，則需評估將流程帳號的行事曆資料夾權限由 Reviewer 調整為 Editor；這是資料夾權限調整，不代表流程日常執行需要 Exchange Administrator。

## 恢復後固定測試步驟

1. 在測試流程加入 `取得行事曆 (V2)`。
2. 執行流程並保存回傳的 `name`、`owner.address`、`id`。
3. 依三個 Resource Mailbox Email 配對 Calendar ID。
4. 分別加入或執行三個 `取得事件的行事曆檢視 (V3)`。
5. 查詢區間使用執行當下到未來 7 天，並以 `Asia/Taipei` 顯示測試結果。
6. 驗證七項必要欄位並更新本文件。
7. 三台均通過後更新 CHANGELOG、Master、里程碑與完成度，再提交下一個 commit。

## 本輪結論

測試狀態：**部分完成；V2 標準連線與動作執行成功，但三台資源行事曆未被列出，V3 尚未執行**。

權限結論：**流程不需要 Exchange Administrator；但 Reviewer 目前不足以讓三台行事曆自動出現在 Power Automate 清單，需先完成 Outlook 明確加入測試，必要時再評估 Editor 權限**。
