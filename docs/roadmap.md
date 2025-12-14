# 發展計劃

此專案最終目標為自動交易系統，依據使用者所訂定的策略，並希望對於目前所有的金融商品 (包括傳統交易系統)，可以透過此系統做個人設定，並進一步直接進行交易。

## 項目功能

TBD

## 硬體環境

- Server: Intel i9 (負責運算與 產品線的 Docker 運行，目前還是發展階段，故尚未使)
- Client: iMac M3 (負責開發與瀏覽器操作，開發環境執行，但目前也是發展階段的執行環境)
- 資料儲存: 本地磁碟機 Parquet 檔案。

## 技術堆疊 (Tech Stack)

- **Backend**: Python 3.12, Django Ninja, uv (套件管理), Pandas, Polars.
- **Frontend**: SvelteKit, TypeScript, Shadcn-Svelte, TailwindCSS.
- **Charts**: TradingView Lightweight Charts.
- **Infra**: Docker Compose, PostgreSQL (存 metadata), Redis.

## 專案目錄結構 (Monorepo)

- `data/`: 存放 parquet/raw 數據 (Docker volume 掛載至 /app/data)。
- `backend/`: Django 專案根目錄 (manage.py 在此)。
  - `core/`: 全域設置
  - `apps/`: 依領域分 (market_data, analysis, trading)。
  - `services/`: 純 Python 商業邏輯層。
- `frontend/`: SvelteKit 專案。

## Roadmap

依此發展順序，逐步完成系統，從加密貨幣量化分析開始，完成後，再加入傳統金融商品的交易。

### 加密貨幣量化交易系統

#### Phase 1: 數據基礎建設 (Data Infrastructure)

目標：建立可靠的本地數據池，並能在前端視覺化。重點在於 market_data 模組實作。

- [x] **系統基礎設定 (System Config)**:
  - [x] 建立 `SystemSetting` Model (包含 audit 欄位: updated_by)。
  - [x] 實作 CRUD API 統一回應格式 (`ApiResponse<T>`)。
  - [x] 前端基礎建設 (`config.ts`, `types.ts`)。
  - [ ] 前端設定頁面 (`/settings`) UI 整合與測試。
  - [ ] 完成 `ParquetConfigSchema` 驗證邏輯。
- [ ] 交易所適配器 (Exchange Adaptor): 完善 `backend/services/exchanges/`，實作 `Binance/OKX/Coinbase` 的下載歷史數據。
  - [x] 定義 `BaseExchangeClient` 介面。
  - [ ] 實作 `BinanceClient` 基礎連線與設定 (使用 CCXT)
  - [ ] 實作 `get_markets` 與 `fetch_ohlcv` 的整合測試
  - [ ] 實作 OKX/Coinbase 的適配器
- [ ] ETL 流程 (Data Pipeline): 建立定期任務 (可使用 Celery 或簡單的 Cron)，將原始數據清洗，並轉存為 Parquet。
  - 關鍵: 確保 Parquet 的 Partitioning 策略正確。按 Symbol/Year/Month 或 Symbol/Timeframe，這對 Polars 的讀取速度很重要。
- [ ] K 線視覺化: 前端整合 Lightweight Charts，透過 Django Ninja 讀取 Parquet 並回傳給 SvelteKit 渲染。

#### Phase 2: 策略回測引擎 (Backtesting Engine)

目標：驗證交易輯的獲利能力。此為 analysis 模組的核心。

- 指標庫 (Indicator Lib): 在 backend/services/indicator_calc.py 中封裝 pandas-ta 或自行用 polars 寫高性能指標計算。
- 向量化回測 (Vectorized Backtest): 利用 Polars 的極速特性，針對歷史數據進行全量回測 (不考量滑點與詳細搓合)，快速篩選策略。
- 回測報告: 在前端顯示權益曲線 (Equity Curve)、最大回撤 (MOD)、夏普比率 (Sharpe Ratio)。

#### Phase 3: 模擬交易與事件驅動 (Paper Trading & Event Loop)

目標：從「靜態分析」轉向「動態交易」，但不涉及實際交易。從此開始進入 trading 模組。

- 模擬交易所 (Paper Exchange): 寫一個假的交易所 Class，模擬訂單搓合、手續費與滑點。
- 事件驅動引擎 (Event-Driven Engine): 為了將來的自動交易，需要一個可接收 WebSocket 即時報價並觸發策略的迴圈。
  這與 Phase 2 的向量化回測不同，這是逐筆 (Tick-by-tick) 或逐根 K 線 (Bar-by-bar) 的處理。
- 即時監控儀表板 (Dashboard): SvelteKit 透過 SSE (Servver-Sent Events) 或 WebSocket 顯示即時報價與模擬單狀態。

#### Phase 4: 實盤自動交易 (Live Trading)

目標：正長串接交易所 API 進行下單。

- API Key 管理與安全: 確保 API Key 加密儲存。
- 執行路由器 (Execution Router): 根據信號發送真實訂單 (Market/Limit/Stop)。
- 風險控制模組 (Risk Management): 這是最重要部分。在發送訂單前強制檢查。
  - 單筆虧損不超過總資產 2%
  - 每日最大虧損停止交易
  - Kill Switch
- Line/Discord 通知: 交易發生時推播通知到手機。

#### Phase 5: 擴展與 AI 學習 (Expansion & AI Training)

目標：引入機器學習與傳統金融分析與交易。透過已建立好的可擴展架構，使系統也可以進行傳統金融 (股票) 分析與交易。

- 傳統金融適配: 擴充 Exchange 介面以支援 IBKR (Interactive Brokers) 或 Alpaca。需處理「休市時間」與「除權息」等加密貨幣沒有的問題。
- 機器學習: 利用已收集的 Parquet 數據訓練模型 (LSTM, Transformer)，作為策略的過濾器 (Filter) 或訊號為 (Signal)

## 架構與實作

### Coding 原則

- API 的 response 回傳，儘量用 Django-ninja 的格式。

### 架構原則

- 數據儲存 (Parquet vs Database)
  - 使用 Postgres 儲存 Metadata
  - 直接用 Polars 讀取 Parquet 檔案，經運算後轉 JSON 給前端。
  - Postgres 僅用來記錄「哪些幣種正在追蹤」、「回測紀錄摘要」、「使用者設定」等。
- 開發環境 (Tasks & Setting)
  - tasks.json: 要一個 Parquet 的同步，或是傳入雲端做為資料同步
- 後端結構 (Service Layer)
  - api.py 只負責解析 Request、驗證 Schema，呼叫 services/ 的函式，最後回傳 Respnose。
  - schemas.py 負責定義 api.py 所使用的 Schema
- 前端介面 (TradingView)
  - Lightweight Charts 在處理「即時更新」時，注意 Reactivity。
  - 儘量使用 Svelte 5 的新特性處理。
- 預留傳統金融的延展性 (TradFi)
  - 需要先加入 asset_type (Crypto/Stock/Forex) 和 market_hours 在 Schema。
  - 提供傳統金融非連續性的相關設置。
