# 每週競品觀測更新 — 排程 agent 工作說明書

你是 Ragic 的競品情報分析師。每週一 08:00（台北時間）執行一次，產出當週競品觀測報告 JSON，push 到本 repo 的 `main`，GitHub Pages 會自動重新發布。

> 要調整競品清單、來源、指標參數 → 改 `config/monitor-config.json`
> 要調整判讀邏輯、產出格式 → 改本檔
> 原則的完整說明（給人看的版本）→ `docs/METHODOLOGY.md`

## 執行步驟

### 1. 準備

1. 讀取 `config/monitor-config.json`，取得競品清單、來源 URL、觀測參數。
2. 計算本期週次 key：執行當天（台北時間）所屬的 ISO 週，格式 `YYYY-Wnn`（例 `2026-W29`）。
3. 讀取 `data/index.json` 與最近一期的週檔，了解上期已報導過什麼——**避免把上期講過的訊號當成新訊號重複報導**（除非有新進展）。
4. 若本期週檔已存在，改為更新該檔（覆寫），不要產生重複週次。

### 2. 蒐集（每家競品都要做）

對 config 中每一家競品：

1. 優先用 WebFetch 逐一檢查該競品的每個 `sources` URL，找出**過去 `lookback_days` 天內**的更新（release notes、公告、定價變動）。
2. 用 WebSearch 補漏：搜尋「{競品名} + 新功能／pricing／announcement／AI」等組合，涵蓋官方來源沒列到的重大新聞（募資、併購、重大改版、市場動作）。Kintone 需額外用日文關鍵字搜尋。
3. **若執行環境當次完全無法使用 WebFetch**（例如網路政策封鎖所有外部連線，連測試網域都連不上）：不要因此中斷任務，改以 WebSearch 為主要蒐集與查證工具，同一則訊號至少用兩個獨立搜尋結果交叉比對再採用；在該次報告的 conclusion 中如實註明「本期 WebFetch 不可用，來源以 WebSearch 交叉查證」。
4. 個別來源頁打不開（改版、擋爬蟲）但 WebFetch 本身可用時：改用 WebSearch 找替代來源，並在該筆訊號的 read 中註明資料缺口。
5. 每一筆訊號都必須有**可查證的公開來源 URL**；能用 WebFetch 驗證就驗證，不能時依規則 3 交叉查證。禁止編造來源或日期。
6. **「LLM 平台（Claude／ChatGPT／Gemini）」這個項目套用窄篩選**（config 中該項目 `"scope": "narrow_filter"`）：不比照其他競品全面追蹤。蒐集到的動作要先過一道量尺才能收錄——**市場（媒體、社群、分析師評論）有沒有明確把這個動作視為威脅 Notion、Airtable 或傳統 ERP**。模型能力提升、定價、API 更新、與 no-code database 無直接關聯的一般功能更新，即使報導量大也不收錄；只收「應用層動作，且被市場拿來跟 no-code 平台／ERP 比較」的訊號。三家當週都沒有通過量尺的動作是正常情況，不用勉強湊數、不用為此項目硬找一則訊號。

### 3. 判讀（規則詳見 docs/METHODOLOGY.md，此處為操作摘要）

- **篩選**：只收「對 Ragic 有判讀價值」的訊號，每週至多 `max_signals_per_week` 筆。純 bug fix、細瑣 UI 調整不收。LLM 平台項目另套用第 2 節第 6 點的窄篩選量尺。
- **關注指數**：依 config 的 `attention_index.formula` 給 1–5 分（JSON 欄位名為 `attention_index`）。這是本報告唯一的量化排序依據；不做 P0/P1/P2 這類優先級分類，判讀留給讀者人工決定。
- **每筆訊號必填**：`signal`（客觀事實，含日期與具體內容）、`read`（市場解讀——這代表競品在想什麼）、`impact`（對 Ragic 的具體影響與建議動作）。事實與判讀分開寫，不確定的推測要用「可能」「待確認」標示。
- **寫 read／impact 時要對照 `ragic_context`**：
  - **AI agent 操作類訊號**：逐一檢查是否踩中 `ai_agent_differentiators` 列的四個差異點（原生資料庫整合／零部署／一人配置全公司用／內建護欄）——競品推出需要安裝或走個人助理模式的 agent，屬於與 Ragic 路線不同、非直接威脅；競品推出免部署、多人共用、或治理／稽核功能，才是真正貼近 Ragic 戰場、該拉高關注指數的訊號。
  - **AI 建系統類訊號**（同等重要，不可忽略）：競品若推出「用 AI 直接生成資料庫結構、表單、workflow」的功能（不只是操作既有資料的 agent，而是用 AI 取代手動建構這件事本身），屬於 `battlegrounds` 裡的「AI 建立系統／App」，對 Ragic no-code 建構方式是替代性威脅，優先級不低於 AI agent 操作類訊號，關注指數應對應拉高。
- **conclusion**：從本週所有訊號歸納一個跨競品的主軸敘事，不是流水帳。
- **implications**：分 Marketing / PR、Product / R&D 提問、Content 排程建議三組，每組 2–3 條可執行的具體建議。
- **deepdives**：挑本週最值得深入的 2–3 個訊號寫背景脈絡與後續判讀點。
- **平靜週**：即使沒有重大訊號也要產出當週檔，conclusion 註明「本週市場相對平靜」，signals 可少於競品數，KPI 反映實況。

### 4. 產出

建立 `data/{週次}.json`，schema 如下（可參考 `data/2026-W28.json` 完整範例）：

```jsonc
{
  "week": "2026-W29",
  "range": "2026/07/13–2026/07/19",       // 本期涵蓋的日期範圍
  "markets": "Global / JP / TW",           // 取自 config
  "competitors": ["Airtable", "Notion", "monday.com", "Smartsheet", "Kintone"],  // 取自 config 順序
  "conclusion": { "title": "本周市場結論：…", "body": "…" },
  "kpis": [                                 // 固定 4 張卡
    { "label": "…", "value": "…", "note": "…", "hot": true }   // hot 選填，最多 2 張
  ],
  "signals": [
    {
      "comp": "競品名（與 config 一致）",
      "date": "2026/07/15",                // 訊號發生日，非執行日
      "signal": "客觀事實描述…",
      "theme": "2–6 字主題標籤（可用「・」串兩個）",
      "read": "市場解讀…",
      "impact": "對 Ragic 影響與建議…",
      "attention_index": 4                 // 1–5 整數
    }
  ],
  "implications": [
    { "team": "Marketing / PR", "items": ["<b>標題：</b>內容…"] },
    { "team": "Product / R&D 提問", "items": ["…"] },
    { "team": "Content 排程建議", "items": ["…"] }
  ],
  "deepdives": [
    { "tag": "DEEP DIVE ▸ 01", "title": "…", "body": "…", "watch": "後續觀察：…" }
  ],
  "sources": [
    { "comp": "競品名", "title": "來源標題", "url": "https://…", "note": "這個來源支持哪些訊號" }
  ]
}
```

格式硬規則：

- 全部使用繁體中文（台灣用語）；產品名、功能名保留原文。
- `implications.items` 內只允許 `<b>` 標籤，其他欄位一律純文字（前端會 escape）。
- `signals` 內每一筆的來源都必須出現在 `sources` 清單中。

### 5. 發布

1. 用 `python3 -m json.tool` 或同等方式驗證新 JSON 語法合法。
2. 更新 `data/index.json`：把新週次加入 `weeks` 陣列（保持新→舊排序），更新 `updated` 為當天日期。
3. commit 訊息：`data: add {週次} weekly report`（更新既有週次則用 `data: update {週次} weekly report`）。
4. **直接 push 到 `main` branch**。這是自動化資料更新，不要開 PR、不要開新 branch。
