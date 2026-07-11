# Ragic Competitor Monitor

Ragic 競品觀測站：每週自動更新的競品情報 dashboard。

- **公開頁面**：https://gozodeer0416.github.io/ragic-competitor-monitor/
- **更新頻率**：每週一 08:00（台北時間）由 Claude 排程雲端 agent 自動抓取、判讀、發布
- **方法論**：資料來源、抓取與判斷原則見 [docs/METHODOLOGY.md](docs/METHODOLOGY.md)

## 架構

```
index.html               dashboard 前端（GitHub Pages 發布）
data/
  index.json             週次清單（新→舊）
  YYYY-Wnn.json          每週一檔的觀測資料
config/monitor-config.json   競品清單、來源 URL、指標參數
prompts/weekly-update.md     排程 agent 的判讀規則與產出規格
docs/METHODOLOGY.md          方法論說明文件
```

每週一 00:00 UTC，Claude routine 會 clone 本 repo → 讀 config 與 prompts → 抓取各競品近 7 天更新 → 產出當週 JSON → push 到 `main` → GitHub Pages 自動重新部署。

## 修改觀測範圍或邏輯

改檔案 push 到 `main` 即可，下週自動生效（詳見 [METHODOLOGY.md 第 4 節](docs/METHODOLOGY.md#4-如何修改指標競品邏輯)）：

- 競品／來源／參數 → `config/monitor-config.json`
- 判讀邏輯與產出格式 → `prompts/weekly-update.md`
- 排程時間 → https://claude.ai/code/routines

## 本機預覽

```bash
python3 -m http.server 8000
# 開 http://localhost:8000
```

（需經 http 伺服器預覽；直接開 `index.html` 檔案會因 fetch 限制載不到資料。）
