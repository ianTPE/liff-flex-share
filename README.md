# liff-share-test

用 LIFF 的 `liff.shareTargetPicker()`，讓一般 LINE 個人帳號直接發送 Flex Message 卡片，不需要官方帳號。

Send Flex Message cards from a regular LINE account with `liff.shareTargetPicker()`. No official account needed.

實測日期：2026-09-25

## 為什麼要用 shareTargetPicker

LINE App 本身不能讓個人帳號發 Flex Message。要從個人帳號發，只能透過 LIFF 小程式的 `liff.shareTargetPicker()`。訊息由使用者自己的帳號送出，算一般聊天，不計入任何官方帳號的訊息額度。

| 分享方式 | 對方看到的 |
|---|---|
| 分享 LIFF 網址 | 連結或連結預覽，要點進去才看得到 |
| `shareTargetPicker` | Flex 卡片本身，直接顯示在聊天室 |

## 設定步驟

1. 在 LINE Developers 的 Provider 底下建立一個 **LINE Login channel**。Messaging API channel 不能掛 LIFF。
2. 在該 channel 的 LIFF 分頁新增小程式：
   - Size：Full
   - Endpoint URL：網頁網址（必須是 https）
   - Scopes：勾選 `profile`
3. 在 LIFF 分頁打開 **Share target picker**。
4. 網頁載入 LIFF SDK，`liff.init()` 之後呼叫 `liff.shareTargetPicker([flexMessage])`。本 repo 的網頁放在 GitHub Pages。
5. 在手機 LINE 裡打開 `https://liff.line.me/<LIFF ID>`，按分享、選收件人、送出。

```js
await liff.init({ liffId: "<LIFF ID>" });

const result = await liff.shareTargetPicker([
  {
    type: "flex",
    altText: "活動通知",
    contents: {
      type: "bubble",
      body: {
        type: "box",
        layout: "vertical",
        contents: [{ type: "text", text: "週六技術分享會", weight: "bold" }]
      },
      footer: {
        type: "box",
        layout: "vertical",
        contents: [
          {
            type: "button",
            style: "primary",
            action: { type: "uri", label: "加入 Zoom", uri: "https://example.com" }
          }
        ]
      }
    }
  }
]);

if (result) console.log(`sent: ${result.status}`);
```

## 實測結果

- 對方聊天室直接顯示完整的 Flex 卡片，不是連結。
- 依 LINE 官方文件，這類訊息看起來就像使用者本人送出。
- 聊天列表和通知只顯示 `altText`，要點進聊天室才看得到卡片。

## 按鈕動作的限制

LINE 官方文件寫明：用 `shareTargetPicker` 分享 Flex 時，按鈕只能用「開網址」這一種動作。

| 按鈕動作 | 一般帳號用 LIFF 分享能用嗎 |
|---|---|
| 開網址（`uri`） | ✅ 可以 |
| 回傳資料給機器人（`postback`） | ❌ 需要官方帳號 |
| 替使用者送出一句話（`message`） | ❌ 需要官方帳號 |
| 複製文字（`clipboard`） | ❌ 需要官方帳號 |
| 選日期時間（`datetimepicker`） | ❌ 需要官方帳號 |

所以「加入 Zoom」、「填寫表單」、「看活動詳情」這類連到網址的按鈕，一般帳號用 LIFF 就能做，不需要 Messaging API。網址最長 1,000 字元。

## 其他限制

- 每次都要本人在分享畫面選收件人、按送出，無法全自動。
- 不能發到社群（OpenChat）。
- channel 預設是開發中狀態，只有管理員和測試人員能打開這個小程式；收件人不需要任何設定。
- LINE 官方建議新做的 LIFF 改做成 LINE MINI App，兩者將整合（2025-02-12 公告）。目前掛在 LINE Login channel 上仍可使用。
