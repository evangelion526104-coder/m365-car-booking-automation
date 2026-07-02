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
3. Power Automate 詳細頁與編輯頁可導向，但本輪瀏覽器自動控制在讀取畫面、互動元素及截圖時持續逾時。
4. 本機未安裝或登入 `pac`、Azure CLI、Microsoft 365 CLI，無可用的等價命令列控制路徑。
5. 因此本輪未修改雲端流程、未取得 Calendar ID、未讀取三台車事件，也未將任何待測欄位誤標為通過。

## 恢復後固定測試步驟

1. 在測試流程加入 `取得行事曆 (V2)`。
2. 執行流程並保存回傳的 `name`、`owner.address`、`id`。
3. 依三個 Resource Mailbox Email 配對 Calendar ID。
4. 分別加入或執行三個 `取得事件的行事曆檢視 (V3)`。
5. 查詢區間使用執行當下到未來 7 天，並以 `Asia/Taipei` 顯示測試結果。
6. 驗證七項必要欄位並更新本文件。
7. 三台均通過後更新 CHANGELOG、Master、里程碑與完成度，再提交下一個 commit。

## 本輪結論

測試狀態：**未完成，等待 Power Automate 網頁編輯器恢復可控制狀態**。  
權限結論：**不需要 Exchange Administrator，現有 Calendar Reviewer 為正確方向**。
