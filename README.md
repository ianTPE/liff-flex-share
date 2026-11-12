# liff-flex-share

把 Markdown 或 Flex JSON 變成 LINE 卡片，用一般個人帳號直接分享到聊天室，不需要官方帳號。

Turn Markdown or Flex JSON into LINE Flex Message cards and share them from a regular LINE account with LIFF `shareTargetPicker()`. No official account needed.

📖 **圖文教學**：[從個人 LINE 發 Flex 卡片](https://iantpe.github.io/liff-flex-share/guide.html)，寫給不會寫程式的人，一步一步請 AI 幫你做好。

## 它能做什麼

- **Markdown 轉卡片**：貼上 Markdown，自動排版成 Flex 卡片。支援粗體、斜體、行內程式碼、清單、表格、引用和分隔線，連結會變成卡片底部的按鈕。
- **Flex JSON 模式**：內容以 `{` 開頭就原樣送出，可以填完整訊息（`type: flex`），也可以只填一張 bubble 或 carousel。
- **預設卡片**：網址加上 `?f=名稱`，會載入 `flex/名稱.json`，適合重複發送同一張卡片。
- **即時預覽**：輸入後馬上看到排版結果和實際送出的 Flex JSON，格式錯誤會直接提示。
- **電腦也能發**：在電腦瀏覽器打開，登入 LINE 後就能直接分享，不用手機。
- **AI 也能操作**：頁面寫有給 AI agent 的操作說明，AI 可以填好內容、確認預覽，但選收件人和送出這一步一定留給人。

## 使用方式

1. 打開 `https://liff.line.me/<LIFF ID>`：
   - **手機**：在 LINE 裡點開（例如先把網址傳到任一聊天室再點）。
   - **電腦**：用 Chrome 等瀏覽器打開，頁面會先導去 LINE 登入（帳號密碼或掃 QR code），登入後回到工具頁。
2. 貼上 Markdown 或 Flex JSON，確認預覽。
3. 按「分享卡片」，選收件人、送出。對方會在聊天室直接看到卡片，不是連結。

### 搭配 Claude Code／Codex

用電腦發卡片時，可以整段交給 AI：

1. 跟 AI 描述卡片內容，請它產生 Flex JSON。
2. 有瀏覽器操作能力的 AI（例如 Claude Code 的 Claude in Chrome）可以直接打開工具頁、把 JSON 填進去、確認預覽沒有錯誤。
3. 你按「分享卡片」，選收件人、送出。

常發的卡片也可以請 AI 存成 `flex/名稱.json` 並 push，之後打開 `https://liff.line.me/<LIFF ID>?f=名稱` 就會直接載入。

## 檔案

| 檔案 | 用途 |
|---|---|
| `index.html` | 主程式：Markdown／Flex JSON 轉卡片並分享 |
| `flex/` | 預設卡片的 Flex JSON |
| `assets/` | 卡片用的圖片 |
| `guide.html`、`demo.html` | 教學：讓 AI 製作網頁並發布到 GitHub Pages |
| `jev-proof.html` | 卡片底部按鈕連到的文章網頁版 |

## 自己架設

不會寫程式的話，建議直接照[圖文教學](https://iantpe.github.io/liff-flex-share/guide.html)做，第 6 節有請 AI 複製這個工具的 prompt。自己動手的話：

1. 在 LINE Developers 的 Provider 底下建立一個 **LINE Login channel**。Messaging API channel 不能掛 LIFF。
2. 在該 channel 的 LIFF 分頁新增小程式：
   - Size：Full
   - Endpoint URL：網頁網址（必須是 https，例如 GitHub Pages）
   - Scopes：勾選 `profile`
3. 在 LIFF 分頁打開 **Share target picker**。
4. 把 `index.html` 裡的 LIFF ID 換成你自己的。

```js
await liff.init({ liffId: "<LIFF ID>" });
const result = await liff.shareTargetPicker([flexMessage]);
if (result) console.log(`sent: ${result.status}`);
```

## 實測紀錄（2026-09-25）

LINE App 本身不能讓個人帳號發 Flex Message，只能透過 LIFF 小程式的 `liff.shareTargetPicker()`。訊息由使用者自己的帳號送出，算一般聊天，不計入任何官方帳號的訊息額度。

| 分享方式 | 對方看到的 |
|---|---|
| 分享 LIFF 網址 | 連結或連結預覽，要點進去才看得到 |
| `shareTargetPicker` | Flex 卡片本身，直接顯示在聊天室 |

- 對方聊天室直接顯示完整的 Flex 卡片。
- 依 LINE 官方文件，這類訊息看起來就像使用者本人送出。
- 聊天列表和通知只顯示 `altText`，要點進聊天室才看得到卡片。

### 電腦瀏覽器（2026-11-12）

- 在電腦 Chrome 打開 `https://liff.line.me/<LIFF ID>`，用 `liff.login()` 登入 LINE 後，`shareTargetPicker` 可以正常使用。
- 選收件人視窗會在瀏覽器裡跳出，送出後對方一樣直接看到卡片。
- 由 Claude Code 透過 Claude in Chrome 填入 Flex JSON、人按分享並送出，實測成功。

### 按鈕動作的限制

LINE 官方文件寫明：用 `shareTargetPicker` 分享 Flex 時，按鈕只能用「開網址」這一種動作。

| 按鈕動作 | 一般帳號用 LIFF 分享能用嗎 |
|---|---|
| 開網址（`uri`） | ✅ 可以 |
| 回傳資料給機器人（`postback`） | ❌ 需要官方帳號 |
| 替使用者送出一句話（`message`） | ❌ 需要官方帳號 |
| 複製文字（`clipboard`） | ❌ 需要官方帳號 |
| 選日期時間（`datetimepicker`） | ❌ 需要官方帳號 |

所以「加入 Zoom」、「填寫表單」、「看活動詳情」這類連到網址的按鈕，一般帳號用 LIFF 就能做。網址最長 1,000 字元。

### 其他限制

- 每次都要本人在分享畫面選收件人、按送出，無法全自動。
- 不能發到社群（OpenChat）。
- 單則訊息最多 12 張泡泡、約 30KB；altText 最多 400 字；按鈕文字最多 20 字。
- channel 預設是開發中狀態，只有管理員和測試人員能打開這個小程式；收件人不需要任何設定。
- LINE 官方建議新做的 LIFF 改做成 LINE MINI App，兩者將整合（2025-02-12 公告）。目前掛在 LINE Login channel 上仍可使用。

## 需要全自動定時發送？

個人帳號的 `shareTargetPicker` 每次都要本人選收件人、按送出，不能排程自動發；用程式去點選人畫面，等於把個人帳號當機器人，有違反 LINE 使用條款的風險。

需要每週固定通知之類的全自動發送，要改用 LINE 官方帳號（Messaging API）。官方帳號的訊息額度是照「收到的人數」計算，人多的群組免費額度很快就會用完。省額度的做法：

| 發送方式 | 計入額度？ |
|---|---|
| 主動發送（push／multicast／broadcast） | ✅ 計入，發到群組依人數計算 |
| 回覆訊息（reply，用 webhook 收到的 `replyToken`） | ❌ 不計入 |

回覆訊息一樣可以送 Flex 卡片（一次最多 5 則），卡片格式和這個工具用的 Flex JSON 相同，按鈕也能用 `postback` 等全部動作。可以設計成：群裡有人打關鍵字，或某個時段第一則訊息出現時，機器人用回覆送出通知卡片。限制是一定要有人先說話，而且 `replyToken` 很快會過期，要盡快回覆。

| 方法 | 自動 | 費用 | 發送者 |
|---|---|---|---|
| 本工具（個人帳號） | 半自動，人按送出 | 免費 | 你本人 |
| 官方帳號 reply | 有人說話就自動回 | 免費 | 官方帳號 |
| 官方帳號 push | 全自動定時 | 超過免費額度要付費 | 官方帳號 |
