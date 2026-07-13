# Ragic 競品觀測 — 資料來源、抓取與判斷原則

本文件說明競品觀測 dashboard 的資料從哪裡來、怎麼抓、怎麼判讀，以及如何修改觀測範圍與邏輯。

- 觀測頁面：`index.html`（GitHub Pages 公開發布）
- 資料檔：`data/` 目錄，每週一檔 JSON
- 首頁文案快照：`data/homepage-snapshots.json`（非週次歸檔，持續更新的最新狀態，供「定位觀察」比對用，見 §2、§3.5）
- 機器設定：`config/monitor-config.json`（競品、來源、參數）
- 判讀規則（agent 工作說明書）：`prompts/weekly-update.md`

## 1. 監測範圍

| 競品 | 主要市場 | 為什麼追蹤 |
|---|---|---|
| Airtable | Global | no-code 資料庫直接競品，已轉向 AI-native 定位 |
| Notion | Global | 工作空間＋資料庫混合型，agent 平台化聲量最大 |
| monday.com | Global | 主打 work OS，AI 變現策略最激進，定價比較基準 |
| Smartsheet | Global | 表格型工作平台，MCP／AI 入口分發策略參考案例 |
| Kintone | JP / TW | 日本龍頭，近年積極進入台灣市場 |
| Odoo | Global / TW | 開源 ERP 套件，內建 Odoo Studio 無程式碼工具，與 Ragic no-code 核心重疊，台灣代理商網絡活躍 |
| LLM 平台（Claude／ChatGPT／Gemini） | Global | **窄篩選、非全面追蹤**：只收應用層動作中被市場明確視為威脅 Notion／Airtable／傳統 ERP 的訊號，見下方說明 |

觀測市場：Global / TW（Kintone 個別追蹤延伸至 JP）。競品清單與各家來源 URL 以 `config/monitor-config.json` 為準（本表為說明用途）。

## 2. 資料來源與抓取方式

**執行者**：Claude 排程雲端 agent（routine），每週一 08:00 台北時間（週一 00:00 UTC）自動執行。

**來源層級**（優先序）：

1. **官方一手來源**：各競品的 release notes / changelog、官方 newsroom / blog、定價頁、help center 更新頁——config 中每家競品的 `sources` 清單。
2. **搜尋補漏**：官方來源沒涵蓋的重大新聞（募資、併購、市場動作、社群反應），以 web 搜尋補足；Kintone 額外用日文關鍵字。
   - **指定關鍵字監測**：部分競品在 `config` 設有 `watch_keywords`——特定產品線／功能名稱，每次執行會額外主動搜尋這些字詞有無變動，不只是靠通用搜尋帶到。目前設定：Airtable（Hyperagent、Omni、Field Agents、AI automation）、monday.com（Vibe、Agent Factory、Sidekick、AI workflow）。
3. 來源頁抓取失敗（改版、擋爬蟲）時改用搜尋找替代來源，並在報告中註明資料缺口。
4. 若執行環境當次完全無法使用 WebFetch（曾發生過雲端環境的網路政策封鎖所有外部連線，連測試網域都連不上）：改以 WebSearch 為主，用兩個以上獨立結果交叉查證，報告中會如實註明此狀況。

**回溯範圍**：每次執行抓過去 7 天（`lookback_days`）；會比對上一期報告，同一訊號不重複報導，除非有新進展。

**查證要求**：每筆訊號必須附可公開查證的來源 URL；能用 WebFetch 直接存取驗證就驗證，環境限制無法使用時以多來源交叉查證取代；禁止編造來源或日期。所有引用來源列在頁面的 Sources 區。

**首頁定位追蹤**：部分競品在 `sources` 設有 `homepage` 類型來源（目前：Airtable、Notion、monday.com、Smartsheet、Kintone、Odoo；LLM 平台項目不追蹤），每次執行會抓取首頁主標語與 CTA 文案，跟 `data/homepage-snapshots.json` 存的上次版本比對，寫入當週報告的「定位觀察」區塊，並更新快照檔供下次比對。**只追蹤文字內容，不追蹤視覺／版面改版**（工具限制，無截圖比對能力）。

## 3. 判斷原則

### 3.1 訊號篩選

只收「對 Ragic 有判讀價值」的更新，每週至多 10 筆；純 bug fix、細瑣 UI 調整不收。判讀價值 = 涉及 Ragic 核心戰場（與 `config/monitor-config.json` 的 `ragic_context.battlegrounds` 保持一致）：AI agent 功能與定價、AI agent 部署模式與治理、AI 建立系統／App、MCP 整合與分發、no-code 資料庫核心、全球市場（特別著重台灣與歐美）。

**LLM 平台（Claude／ChatGPT／Gemini）另套用更窄的收錄量尺**：不追蹤模型能力發布、定價、API 更新等一般動態，只收「應用層動作，且市場（媒體、社群、分析師）明確視為威脅 Notion、Airtable 或傳統 ERP」的訊號，例如可直接用自然語言生成試算表／資料庫應用、agent 自主建置完整工作流程與資料結構的新功能。三家當週都沒有通過此量尺的動作是正常情況，報告中不會勉強湊數。

### 3.2 關注指數（1–5）

關注指數 ＝ 更新頻率 × 訊號強度 × 與 Ragic 核心戰場的重疊度。屬**判讀性指標**，反映「這家競品此刻對 Ragic 施加的壓力」，非業務絕對值。（JSON 資料欄位名為 `attention_index`。）這是報告目前唯一的量化排序依據——不做 P0/P1/P2 這類優先級分類，「這則訊號要不要現在回應」留給讀者人工判讀。

### 3.3 事實與判讀分離

每筆訊號拆成三層，事實與意見不混寫：

- **signal（更新訊號）**：客觀事實——日期、功能、定價數字，可回溯到來源。
- **read（市場解讀）**：這代表競品在想什麼、市場往哪走——分析師判讀。
- **impact（對 Ragic 影響）**：對 Ragic 的具體影響與建議動作。

寫 read／impact 時須對照 `config/monitor-config.json` 的 `ragic_context`（`positioning`、`key_narrative`、`ai_agent_differentiators`、`battlegrounds`），分兩類判讀：

- **AI agent 操作類訊號**：檢查是否踩中 Ragic 的四個 AI Agent 差異化重點——原生資料庫整合、零部署、一人配置全公司用、內建護欄／治理。競品若走「需安裝」或「個人助理」路線，屬於與 Ragic 不同賽道；若走「零部署、多人共用、或治理／稽核」路線，才是真正貼近戰場、該拉高關注指數的訊號。
- **AI 建系統類訊號**（同等重要）：競品若推出用 AI 直接生成資料庫結構、表單、workflow 的功能（取代手動建構本身，而非只操作既有資料），屬於挑戰 Ragic no-code 建構方式的替代性威脅，關注指數不應低於 AI agent 操作類訊號。

不確定的推測必須用「可能」「待確認」明確標示。

### 3.4 週報固定產出

每週必產出：跨競品主軸結論（conclusion）、4 張 KPI 卡、訊號記分板（signals）、兩組團隊行動建議（Marketing / PR、Product / R&D 提問）、2–3 張深讀卡（deepdives）、定位觀察（positioning_watch）、來源清單（sources）。沒有重大訊號的平靜週也照常出報告並如實註明。

**KPI 卡為固定模板**，4 張的標題與順序每週不變，只換內容：

1. **本周主軸**——本週所有訊號歸納出的跨競品主要趨勢／方向
2. **定價風向**——本週競品在 AI／訂閱定價模式上的共同動向或代表性個案
3. **最高優先訊號**——本週 `attention_index` 最高的一則訊號
4. **Ragic 應對焦點**——綜合以上，Ragic 這週最該優先思考或行動的方向

沒有對應內容時（例如本週無明顯定價動向）如實註明，不硬湊。

### 3.5 定位觀察（Positioning Watch）

每週列出所有有追蹤首頁的競品，狀態分三種：

- **changed**（有變動）——本週抓到的標語／CTA 跟上次不同，`note` 說明差異
- **unchanged**（無變動）——跟上次相同，`note` 寫「本週無變動」；**即使沒變動也照樣列出，不會省略**
- **unavailable**（暫無法查證）——來源網域封鎖存取，`note` 註明原因

這是純粹的「有沒有變」追蹤，不做優先級判讀；標語／CTA 變了不代表一定重大，但長期累積能看出各家定位敘事怎麼演變。

## 4. 如何修改指標、競品、邏輯

排程 agent 每次執行時都會**重新讀取** repo 內的設定與規則檔，所以修改以下檔案並 push 到 `main`，下週起自動生效，排程本身不用動：

| 想改什麼 | 改哪個檔案 |
|---|---|
| 新增/移除競品、調整來源 URL、觀測市場 | `config/monitor-config.json` 的 `competitors` |
| 新增/移除首頁定位追蹤 | 在該競品的 `sources` 加入或移除 `{"type": "homepage", "url": "..."}`；新增後首次執行會自動建立基準 |
| 每週訊號上限、回溯天數 | `config/monitor-config.json` 的 `report` |
| 關注指數公式 | `config/monitor-config.json` 的 `attention_index`（並同步更新本文件） |
| 判讀邏輯、產出格式、篩選標準 | `prompts/weekly-update.md` |
| 頁面版型與欄位 | `index.html`（新增資料欄位時需同步改 schema 與 render） |
| 執行時間、頻率 | Claude 排程 routine 設定（https://claude.ai/code/routines） |

改法：直接在 GitHub 上編輯，或在本機請 Claude Code 修改後 push。

## 5. 已知限制

- 來源頁改版或擋爬蟲時可能漏抓，agent 會以搜尋補漏並註明缺口，但無法保證 100% 涵蓋。
- 定位觀察只抓文字（標語、CTA），抓不到版面、配色、圖片這類視覺改版；部分官網網域會封鎖 WebFetch（例如已知 openai.com 系列網域會回 403），該情況會標記為 `unavailable` 並註明原因。
- 執行環境偶爾會整體封鎖 WebFetch（雲端網路政策問題），此時報告改以 WebSearch 交叉查證產生，查證嚴謹度略低於直接存取原始頁面；報告中若有此註記，重大決策前建議自行查證來源。
- read / impact 為 AI 分析師判讀，供內部討論起點，重大決策前請回到原始來源查證。
- 資料與判讀內容存於公開 repo，任何人可見；請勿在 config 或 prompts 中加入內部機密資訊。
