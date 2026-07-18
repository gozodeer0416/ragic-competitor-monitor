# 交接文件（HANDOVER）— Ragic Competitor Monitor

> 給接手維護本系統的 AI（或人類工程師）。本 repo 已有完整文件，這份是**導覽與維護注意事項**，不重複既有內容。
> 姊妹專案：[pr-news-radar](https://github.com/gozodeer0416/pr-news-radar)（PR 新聞觀測 dashboard，另有自己的 HANDOVER.md）。

## 0. 文件地圖（先知道去哪找什麼）

| 你想做的事 | 看哪份文件 |
|---|---|
| 改判讀邏輯、每週報告的產出規則與 JSON 格式 | `prompts/weekly-update.md`（排程 agent 的工作說明書，**權威規格**） |
| 改競品清單、來源 URL、關鍵字、參數 | `config/monitor-config.json` |
| 了解觀測原則的設計理由（給人讀的版本） | `docs/METHODOLOGY.md` |
| 改 dashboard 外觀與互動 | `index.html`（單檔，vanilla JS，無建置步驟） |
| 系統架構總覽 | `README.md` |

改 prompt 或 config 時，記得同步 `docs/METHODOLOGY.md`，三者必須一致。

## 1. 最重要的三件事

1. **使用者（Lillian）做 Ragic 的 PR，非工程背景。** 溝通用繁體中文、逐步說明、需要她操作時給網頁 UI 步驟（不要給終端機指令）。她回報版面問題的方式通常是丟截圖——修改後要截圖驗證再上線。
2. **資料由「排程雲端 agent」每週一 08:00（台北時間）產生**，依 `prompts/weekly-update.md` 執行後直接 push 週次 JSON 到 main。前端只是讀 `data/*.json` 的靜態頁。**改前端不影響資料，改 prompt 才影響下週資料。**
3. **前端與資料之間有幾條隱含契約**（見 §2），改任一邊都要確認另一邊還成立。

## 2. 前端契約（改 index.html 或 prompt 時必讀）

| 契約 | 兩邊分別是誰 |
|---|---|
| `conclusion.body` 支援 `\n\n` 分段、`- ` 開頭行列點 | 前端 `formatBody()` ↔ prompt 的 conclusion 條目（2026-07-18 加入；短結論可單段，不強制列點） |
| 定位觀察卡片的官網連結 | 前端讀 `config/monitor-config.json` 中各競品 `sources` 內 `type:"homepage"` 的 URL，以 `competitors[].name` 與週檔 `positioning_watch[].comp` **字串完全比對**。改公司顯示名時兩處要同步，否則連結默默消失（有 fallback 不會壞頁）。 |
| implications 項目允許 `<b>` 粗體 | 前端 `escB()` 只放行 `<b>`；prompt 產出不要用其他 HTML |
| 週檔欄位結構 | `prompts/weekly-update.md` 末段的 JSON 範本是唯一權威；前端所有 render 函式依它取值 |
| KPI 四張卡的 label 順序固定 | prompt 有明確規定，前端不依賴但版面按 4 張設計 |

## 3. 版面設計決策（2026-07-18 與使用者逐項確認，不要無故回退）

- **1440px 版面寬**：她嫌 1080px 留白太多；選項比較後她選 1440（非滿版，保閱讀行長）。
- **文字不設行寬上限**（結論卡、各區 lede）：她明確要求「不要在中間就換行」。
- **本週重點在行動建議之前**（先脈絡後建議）。
- **deep-dive 網格 `auto-fit`**：卡片數 2 或 3 都填滿整行，不留空欄。
- **定位觀察是灰藍內嵌面板＋「參考追蹤區・不判讀優先級」標籤**：她要求視覺上與分析區塊區隔，因為這區性質是被動追蹤。區塊小標刻意用灰色不用橘色。

## 4. 已否決與待辦

- **已否決：行動建議的網頁端回饋按鈕（localStorage）**——因為標記只存單一瀏覽器，她不要。不要再提這個方案。
- **待辦（她說未來考慮）：行動建議逐條經 Ragic API 寫入她自建的 Ragic 表單做追蹤**（欄位構想：週次/組別/內容/狀態；API key 走 GitHub secret；需去重防重跑重複寫入）。她開口時再做。

## 5. 維護操作備忘

- **本機預覽**：`python3 -m http.server`（直接開檔案會因 fetch 失敗白頁，前端有錯誤提示）。
- **截圖驗證**：headless Chrome 加 `--virtual-time-budget=8000`，否則 async fetch 未完成就拍到空區塊。
- **資料修正**：週檔是 agent 產的，人工修資料（如重排結論分段）直接改 `data/{週次}.json` 後 commit 即可，前端無快取問題（`cache:'no-cache'`）。
- **Pages**：main 分支根目錄發布，push 後 1–2 分鐘生效。
- **歷史相容**：改前端渲染邏輯時，記得舊週次檔（如 2026-W28）也要能正常顯示——週次下拉選單可切回任何一週。
