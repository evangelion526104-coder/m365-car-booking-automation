# Exchange 資源行事曆權限設定與驗證

更新日期：2026-07-02

## 目的

讓 Power Automate 流程帳號 `ad.general@alp.global` 可用 Office 365 Outlook 標準連接器讀取三台公務車資源行事曆。正式流程只需讀取事件，建議採最小權限 `Reviewer`，不需 Premium 連接器。

## 目前實測結果

- 測試流程：`公務車功能測試-資源信箱讀取`
- Flow ID：`7aff726f-0f9d-4b7e-a128-baeb368ab1ce`
- Calendar ID 選單只有個人 `Calendar`，未列出三台公務車。
- 直接輸入 `room_nhb4_car@alp.global` 會回傳：`ID 格式不正確`。
- 測試流程已關閉，待權限完成後再開啟重測。

## 需設定的資源信箱

| 車輛 | 資源信箱 |
|---|---|
| Altis ATA-9627 | `room_nhb4_car@alp.global` |
| Camry BKX-2370 | `room_nhb4_car_camry@alp.global` |
| Cross BKY-0762 | `room_nhb4_car_cross@alp.global` |

## Exchange 管理員操作

先確認各信箱的行事曆資料夾實際名稱；部分環境可能顯示為 `Calendar` 或 `行事曆`：

```powershell
Get-MailboxFolderStatistics -Identity "room_nhb4_car@alp.global" -FolderScope Calendar |
  Select-Object Name, FolderPath, Identity
```

若資料夾名稱為 `Calendar`，依序授予唯讀權限：

```powershell
Add-MailboxFolderPermission -Identity "room_nhb4_car@alp.global:\Calendar" -User "ad.general@alp.global" -AccessRights Reviewer
Add-MailboxFolderPermission -Identity "room_nhb4_car_camry@alp.global:\Calendar" -User "ad.general@alp.global" -AccessRights Reviewer
Add-MailboxFolderPermission -Identity "room_nhb4_car_cross@alp.global:\Calendar" -User "ad.general@alp.global" -AccessRights Reviewer
```

若使用者權限項目已存在，請改用 `Set-MailboxFolderPermission`，避免重複項目錯誤。若實際資料夾名稱是 `行事曆`，請將命令中的 `Calendar` 換成 `行事曆`。

## 驗證步驟

1. 用 `ad.general@alp.global` 在 Outlook 開啟三台共享行事曆。
2. 等待權限生效後，重新整理 Power Automate；必要時重新建立 Office 365 Outlook 連線。
3. 編輯測試流程，在 Calendar ID 選單確認三台資源行事曆均可選取。
4. 逐台執行 `取得事件的行事曆檢視 (V3)`，確認可讀取主旨、開始／結束時間、是否整天、Event ID、iCalUId 與借用人資訊。
5. 三台都通過後，記錄測試結果並再次關閉測試流程。

## 驗收條件

- [ ] 三台資源行事曆都出現在 Calendar ID 選單。
- [ ] 三台都能成功讀取測試事件。
- [ ] 事件時間可轉換為 `Asia/Taipei`。
- [ ] 可取得 Event ID 或 iCalUId。
- [ ] 測試完成後流程維持關閉，正式流程通過端到端驗收後才啟用。
