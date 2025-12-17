# 發展計劃

此專案最終目標為自動交易系統，依據使用者所訂定的策略，並希望對於目前所有的金融商品 (包括傳統交易系統)，可以透過此系統做個人設定，並進一步直接進行交易。

> **技術架構說明**: 詳細的技術推疊、目錄結構與開發規範，請參閱內部文件「 [DEVELOPMENT.md](./DEVELOPMENT.md)」。

## 階段規劃 Milestones

依此發展順序，逐步完成系統，從加密貨幣量化分析開始，完成後，再加入傳統金融商品的交易。

### Phase 1: 數據基礎建設 (Data Infrastructure)

目標：建立可靠的本地數據池，並能在前端視覺化。重點在於 `market_data` 模組實作。

**[AI 提示 / AI Instructions]**
**此階段任務採「垂直切片」模式，需配合 `frontend/src/routers/data-manager` 介面同步開發。**

- [x] **文檔中心建設 (Documentation)**:
  - [x] 部署 MkDocs 至 GitHub Pages [Crypto-QA Documentation](https://your-username.github.io/crypto-qa-doc/)。
  - [x] **API 文件抽離**: 將 API 規格從 README 移至獨立 MkDocs 專案，以便於維護與公開查閱。
    - [x] 分離 User Manual 與 Developer Guide。

- [ ] **系統基礎設定 (System Config)**:
  - [x] 建立 `SystemSetting` Model (包含 audit 欄位: updated_by)。
  - [x] API 介面標準化 CRUD API 統一回應格式 (`ApiResponse<T>`)。
  - [ ] **Slice 1 (UI)**: 前端設定頁面 (`/settings`) UI 整合與測試。
  - [ ] **Slice 2 (Logic)**: 完成 `ParquetConfigSchema` 驗證邏輯。

- [ ] **交易所適配 (Exchange Adaptor)**:
  - [x] `BaseExchangeClient` 介面定義。
  - [x] `BinanceClient` 基礎連線與設定 (使用 CCXT)。
  - [x] 實作連線測試 API (/mkt/test-connection) 與市場列表 (/mkt/sources)。
  - [ ] 支援 OKX/Coinbase 的適配器。

- [ ] **歷史數據同步功能 (History Sync Feature)**:
  - [ ] **Slice 1 (UI)**: 建立「資料源管理」頁面，包含日期選擇與「保留原始檔 (Raw Data)」開關。
  - [ ] **Slice 2 (API)**: 實作 `POST /mkt/sync/history` 接收前端參數 (包含 `keep_raw` 設定)。
  - [ ] **Slice 3 (Logic)**: 實作 `BinanceClient` 下載邏輯，並整合 Hive Parquet 寫入邏輯。
  - [ ] **Slice 4 (Feedback)**: 實作檔案狀態檢查 API，讓前端顯示「已下載/已同步」。

- [ ] **K 線圖表檢視功能 (Candlestick Viewer Feature)**:
  - [ ] **Slice 1 (UI)**: 在 `data-manager/market/[symbol]` 頁面整合 Lightweight Charts。
  - [ ] **Slice 2 (API)**: 實作 `GET /mkt/candles`，支援 Timeframe 參數 Pagination。
  - [ ] **Slice 3 (Logic)**: 使用 Polars 讀取 Parquet (Hive Partition) 並進行高效 Resample (如 1m 轉 4h)。

- [ ] **ETL 與儲存優化 (Backend Optimization)**:
  - [ ] **Binance Vision**: 實作 Bulk Data 下載器，用於快速獲取歷史冷數據。
  - [ ] **Data Governance**: 實作 Raw Data 歸檔機制，確保原始數據的持久化保存。
  - [ ] **Performance**: 建立 `parquet_cold` Hive Partitioning 結構並驗證讀取效能。
  - [ ] **Automation**: 建立定期任務 (可使用 Celery 或簡單的 Cron)，將原始數據清洗，並轉存為 Parquet。

### Phase 1.2: 身份與權限管理 (Auth & RBAC)

目標：實作使用者登入與角色權限區分，確保 `MANUAL.md` 中定義的「管理員」與「交易員」權限得到系統強制執行。

**[AI 提示 / AI Instructions]**
**此階段任務採「整合」模式，開發時可先使用 Mock Provider，但介面需預留 OIDC 流程**

- [ ] **SSO 登入流程 (SSO Authentication)**:
  - [ ] **Slice 1: UI**: 建立登入頁面，實作「前往登入 (Login with SSO)」按鈕與 OAuth2 Redirect 邏輯。
  - [ ] **Slice 2: API**: 實作 `/auth/callback` 或 Token 驗證接口，接收外部 IAM 發出的 JWT (ID Token/Access Token)。
  - [ ] **Slice 3: Logic (Mock)**: 在外部 IAM 尚未就緒前，設計一個 `MockAuthProvider`，模擬回傳標準 JWT，確保開發不被阻塞。

- [ ] **使用者映射與權限 (User Provisioning & RBAC)**:
  - [ ] **Slice 1: Schema**: 設計本地 `UserExtension` 模型，用於儲存外部 User ID (UUID) 與本地角色 (Trader/Admin) 的對照關係。
  - [ ] **Slice 2: Logic**: 實作「自動開通 (Auto-provisioning)」邏輯——當新使用者首次 SSO 登入時，自動在本地建立帳號並給予預設權限。
  - [ ] **Slice 3: Guard**: 實作 Django Ninja `Auth` Dependency 與 SvelteKit `hooks`，解析 JWT 並讀取本地權限表進行路由保護。

### Phase 1.5: 前端礎建設 (Frontend Infrastructure)

- [ ] **App Shell 佈局實作**:
  - [x] 引入 Shadcn Sidebar (SidebarProvider, SidebarRoot, ...)。
  - [x] 實作響應式佈局: Sidebar (Rail) + Header + Status Bar。
  - [ ] 整合 Shadcn `Dark Mode`，適應長時間看盤需求。
  - [ ] 未來規劃: 於量化回測頁面 (`/backtest`) 引入 Resizable Panel 以支援自由調整面板大小。

### Phase 2: 策略回測引擎 (Backtesting Engine)

目標：驗證交易輯的獲利能力。此為 `analysis` 模組的核心。

- [ ] **指標庫 (Indicator Lib)**: 封裝 Polars/Pandas-TA 高效計算模組。
- [ ] **向量化回測 (Vectorized Backtest)**: 利用 Polars 的極速特性，針對歷史數據進行全量回測 (不考量滑點與詳細搓合)，快速篩選策略。
- [ ] **回測報告 (Report)**: 在前端顯示權益曲線 (Equity Curve)、最大回撤 (MOD)、夏普比率 (Sharpe Ratio)。

### Phase 3: 模擬交易與事件驅動 (Paper Trading & Event Loop)

目標：從「靜態分析」轉向「動態交易」，但不涉及實際交易。從此開始進入 `trading` 模組。

- [ ] **模擬交易所 (Paper Exchange)**: 模擬訂單搓合、滑點與手續費。
- [ ] **事件驅動引擎 (Event-Driven Engine)**: 處理 WebSocket 即時報價 (Tick-by-tick)。
  這與 Phase 2 的向量化回測不同，這是逐筆 (Tick-by-tick) 或逐根 K 線 (Bar-by-bar) 的處理。
- [ ] **即時監控儀表板 (Dashboard)**: SvelteKit 透過 SSE (Server-Sent Events) 或 WebSocket 顯示即時報價與模擬單狀態。

### Phase 4: 實盤自動交易 (Live Trading)

目標：串接交易所 API 進行真實下單。

- [ ] **API Key 管理與安全**: 確保 API Key 加密儲存。
- [ ] **執行路由器 (Execution Router)**: 根據信號發送真實訂單 (Market/Limit/Stop)。
- [ ] **風險控制模組 (Risk Management)**: 這是最重要部分。在發送訂單前強制檢查。
  - [ ] 單筆虧損不超過總資產 2%
  - [ ] 每日最大虧損停止交易
  - [ ] Kill Switch
- [ ] **Line/Discord 通知**: 交易發生時推播通知到手機。

### Phase 5: 擴展與 AI 學習 (Expansion & AI Training)

目標：引入機器學習與傳統金融分析與交易。透過已建立好的可擴展架構，使系統也可以進行傳統金融 (股票) 分析與交易。

- [ ] **傳統金融**: 擴充 Exchange 介面以支援 IBKR (Interactive Brokers) 或 Alpaca。需處理「休市時間」與「除權息」等加密貨幣沒有的問題。
- [ ] **AI 模組**: 機器學習: 利用已收集的 Parquet 數據訓練模型 (LSTM, Transformer)，作為策略的過濾器 (Filter) 或訊號為 (Signal)
