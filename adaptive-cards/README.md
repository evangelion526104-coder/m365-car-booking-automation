# Teams Adaptive Card 設計

## 目的

在公務車借用前，由 Power Automate 透過 Teams 發送 Adaptive Card 給借用人本人，請借用人完成以下確認：

- 填寫共乘人數。
- 勾選「我已閱讀並同意遵守公務車使用管理規範」。

完成後，資料即時寫回 SharePoint List，供承辦人判斷是否可發放公務車鑰匙。

## 發送規則

- 一般借用：借用起始時間前 1 小時發送。
- 最早通知時間：借用當天早上 08:00。
- 整天借用：借用當天早上 08:00。

## 卡片內容

| 區塊 | 內容 |
|---|---|
| 標題 | 公務車借用確認 |
| 借用資訊 | 車輛名稱、借用日期、起訖時間、使用事由 |
| 規範文字 | 固定版公司公務車使用管理規範 |
| 輸入欄位 | 共乘人數 |
| 確認欄位 | 我已閱讀並同意遵守公務車使用管理規範 |
| 送出按鈕 | 送出確認 |

## 固定規範文字

請借用人確認：公務車僅供公務用途使用，借用期間應遵守公司公務車使用管理規範，妥善保管車輛、鑰匙及相關設備。使用前應確認車況，使用後應依規定停放並歸還鑰匙。如有事故、異常、違規、損壞或其他特殊情形，應立即通知承辦單位。送出本表單即表示已閱讀並同意遵守相關規範。

## 回覆驗證規則

| 條件 | 結果 |
|---|---|
| 共乘人數已填，且規範確認已勾選 | SharePoint `領鑰狀態` 更新為 `已完成確認` |
| 共乘人數未填 | SharePoint `領鑰狀態` 維持或更新為 `未完成填寫` |
| 規範確認未勾選 | SharePoint `領鑰狀態` 維持或更新為 `未完成填寫` |

## 回覆前防呆檢查邏輯

Teams Adaptive Card 送出前看起來是借用人的回覆，但系統不能直接寫回 SharePoint。正式流程必須先重新讀取 SharePoint 最新資料，確認這張卡片仍然有效。

回覆前必須檢查：

| 檢查項目 | 必須符合 |
|---|---|
| SharePoint 項目 | `sharePointItemId` 找得到同一筆資料 |
| Event ID | 卡片送回的 `eventId` 等於 SharePoint 最新 `行事曆事件 ID` |
| iCalUId | 若有使用，卡片送回的 `iCalUId` 必須一致 |
| 卡片版本 | 卡片送回的 `cardVersion` 等於 SharePoint 最新 `卡片版本` |
| 領鑰狀態 | 不可為 `已取消`、`已完成確認`、`已領鑰`、`已失效` |
| 事件同步狀態 | 必須仍為 `有效` |

若任一檢查不通過，不得更新 SharePoint，並回覆借用人：

```text
此借用已取消或已變更，請重新確認最新借用資訊。
```

此機制可避免借用人送出舊卡片後，覆蓋最新預約資料或重新啟用已取消案件。

## Adaptive Card JSON 範例

以下為設計範例，正式流程中車輛、日期、時間、借用人與事由會由 Power Automate 帶入。

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "text": "公務車借用確認",
      "weight": "Bolder",
      "size": "Large"
    },
    {
      "type": "FactSet",
      "facts": [
        {
          "title": "車輛",
          "value": "${車輛名稱}"
        },
        {
          "title": "借用日期",
          "value": "${借用日期}"
        },
        {
          "title": "借用時間",
          "value": "${借用起始時間} - ${借用結束時間}"
        },
        {
          "title": "使用事由",
          "value": "${使用事由}"
        }
      ]
    },
    {
      "type": "TextBlock",
      "text": "請借用人確認：公務車僅供公務用途使用，借用期間應遵守公司公務車使用管理規範，妥善保管車輛、鑰匙及相關設備。使用前應確認車況，使用後應依規定停放並歸還鑰匙。如有事故、異常、違規、損壞或其他特殊情形，應立即通知承辦單位。送出本表單即表示已閱讀並同意遵守相關規範。",
      "wrap": true
    },
    {
      "type": "Input.Number",
      "id": "carpoolCount",
      "label": "共乘人數",
      "min": 0,
      "isRequired": true,
      "errorMessage": "請填寫共乘人數。"
    },
    {
      "type": "Input.Toggle",
      "id": "policyConfirmed",
      "title": "我已閱讀並同意遵守公務車使用管理規範",
      "valueOn": "true",
      "valueOff": "false",
      "isRequired": true,
      "errorMessage": "請勾選規範確認。"
    }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "送出確認",
      "data": {
        "action": "submitCarBookingConfirmation",
        "sharePointItemId": "${SharePointItemId}",
        "eventId": "${行事曆事件ID}",
        "iCalUId": "${iCalUId}",
        "cardVersion": "${卡片版本}",
        "eventLastModifiedTime": "${事件最後修改時間}"
      }
    }
  ]
}
```

## 目前狀態

| 項目 | 狀態 |
|---|---|
| 卡片需求 | 已確認 |
| 規範固定文字 | 已放入設計 |
| JSON 範例 | 已完成 |
| Teams 正式發送 | 待實作 |
| 回覆寫回 SharePoint | 待實作 |
| 舊卡片失效檢查 | 待實作，上線前必須完成 |
