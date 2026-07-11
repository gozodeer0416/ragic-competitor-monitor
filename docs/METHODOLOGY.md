# Ragic 競品觀測 — 資料來源、抓取與判斷原則

本文件說明競品觀測 dashboard 的資料從哪裡來、怎麼抓、怎麼判讀，以及如何修改觀測範圍與邏輯。

- 觀測頁面：`index.html`（GitHub Pages 公開發布）
- 資料檔：`data/` 目錄，每週一檔 JSON
- 機器設定：`config/monitor-config.json`（競品、來源、參數）
- 判讀規則（agent 工作說明書）：`prompts/weekly-update.md`

## 1. 監測範圍

| 競品 | 主要市場 | 為什麼追蹤 |
|---|---|---|
| Airtable | Global | no-code 資料庫直接競品，已轉向 AI-native 定位 |
| Notion | Global | 工作空間＋資料庫混合型，agent 平台化聲量最大 |
| monday.com | Global | work OS 龍頭，AI 變現策略最激進，定價比較基準 |
| Smartsheet | Global | 表格型工作平台，MCP／AI 入口分發策略參考案例 |
| Kintone | JP / TW | 與 Ragic 客群（日台中小企業、製造業）重疊度最高 |

觀測市場：Global / JP / TW。競品清單與各家來源 URL 以 `config/monitor-config.json` 為準（本表為說明用途）。

## 2. 資料來源與抓取方式

**執行者**：Claude 排程雲端 agent（routine），每週一 08:00 台北時間（週一 00:00 UTC）自動執行。

**來源層級**（優先序）：

1. **官方一手來源**：各競品的 release notes / changelog、官方 newsroom / blog、定價頁、help center 更新頁——config 中每家競品的 `sources` 清單。
2. **搜尋補漏**：官方來源沒涵蓋的重大新聞（募資、併購、市場動作、社群反應），以 web 搜尋補足；Kintone 額外用日文關鍵字。
3. 來源頁抓取失敗（改版、擋爬蟲）時改用搜尋找替代來源，並在報告中註明資料缺口。
4. 若執行環境當次完全無法使用 WebFetch（曾發生過雲端環境的網路政策封鎖所有外部連線，連測試網域都連不上）：改以 WebSearch 為主，用兩個以上獨立結果交叉查證，報告中會如實註明此狀況。

**回溯範圍**：每次執行抓過去 7 天（`lookback_days`）；會比對上一期報告，同一訊號不重複報導，除非有新進展。

**查證要求**：每筆訊號必須附可公開查證的來源 URL；能用 WebFetch 直接存取驗證就驗證，環境限制無法使用時以多來源交叉查證取代；禁止編造來源或日期。所有引用來源列在頁面的 Sources 區。

## 3. 判斷原則

### 3.1 訊號篩選

只收「對 Ragic 有判讀價值」的更新，每週至多 8 筆；純 bug fix、細瑣 UI 調整不收。判讀價值 = 涉及 Ragic 核心戰場：AI agent 功能與定價、MCP 整合與分發、no-code 資料庫核心能力、中小企業（台灣、日本）市場動作。

### 3.2 關注指數（1–5）

關注指數 ＝ 更新頻率 × 訊號強度 × 與 Ragic 核心戰場的重疊度。屬**判讀性指標**，反映「這家競品此刻對 Ragic 施加的壓力」，非業務絕對值。（JSON 資料欄位名為 `attention_index`。）這是報告目前唯一的量化排序依據——不做 P0/P1/P2 這類優先級分類，「這則訊號要不要現在回應」留給讀者人工判讀。

### 3.3 事實與判讀分離

每筆訊號拆成三層，事實與意見不混寫：

- **signal（更新訊號）**：客觀事實——日期、功能、定價數字，可回溯到來源。
- **read（市場解讀）**：這代表競品在想什麼、市場往哪走——分析師判讀。
- **impact（對 Ragic 影響）**：對 Ragic 的具體影響與建議動作。

不確定的推測必須用「可能」「待確認」明確標示。

### 3.4 週報固定產出

每週必產出：跨競品主軸結論（conclusion）、4 張 KPI 卡、訊號記分板（signals）、三組團隊行動建議（Marketing / PR、Product / R&D 提問、Content 排程建議）、2–3 張深讀卡（deepdives）、來源清單（sources）。沒有重大訊號的平靜週也照常出報告並如實註明。

## 4. 如何修改指標、競品、邏輯

排程 agent 每次執行時都會**重新讀取** repo 內的設定與規則檔，所以修改以下檔案並 push 到 `main`，下週起自動生效，排程本身不用動：

| 想改什麼 | 改哪個檔案 |
|---|---|
| 新增/移除競品、調整來源 URL、觀測市場 | `config/monitor-config.json` 的 `competitors` |
| 每週訊號上限、回溯天數 | `config/monitor-config.json` 的 `report` |
| 關注指數公式 | `config/monitor-config.json` 的 `attention_index`（並同步更新本文件） |
| 判讀邏輯、產出格式、篩選標準 | `prompts/weekly-update.md` |
| 頁面版型與欄位 | `index.html`（新增資料欄位時需同步改 schema 與 render） |
| 執行時間、頻率 | Claude 排程 routine 設定（https://claude.ai/code/routines） |

改法：直接在 GitHub 上編輯，或在本機請 Claude Code 修改後 push。

## 5. 已知限制

- 來源頁改版或擋爬蟲時可能漏抓，agent 會以搜尋補漏並註明缺口，但無法保證 100% 涵蓋。
- 執行環境偶爾會整體封鎖 WebFetch（雲端網路政策問題），此時報告改以 WebSearch 交叉查證產生，查證嚴謹度略低於直接存取原始頁面；報告中若有此註記，重大決策前建議自行查證來源。
- read / impact 為 AI 分析師判讀，供內部討論起點，重大決策前請回到原始來源查證。
- 資料與判讀內容存於公開 repo，任何人可見；請勿在 config 或 prompts 中加入內部機密資訊。
