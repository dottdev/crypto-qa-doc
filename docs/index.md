# Crypto-QA 使用者操作手冊 (User Manual)

本手冊分為兩大部分：

1. **交易員指南 (Trader's Guide)**：針對策略開發、回測與日常監控的操作。
2. **管理員指南 (Admin's Guide)**：針對系統設定、數據維護與環境部署的操作。

---

# Part 1: 交易員指南 (Trader's Guide)

此區塊說明如何使用平台進行市場分析與策略執行。

## 1. 儀表板 (Dashboard)

**路徑**: `/` (首頁)

- **總覽**: 顯示目前帳戶權益、活躍策略數量與今日損益。
- **操作**: (待開發)

## 2. 量化分析 (Analysis & Backtest)

**路徑**: `/backtest`

- **功能**: 選擇幣種、時間週期與策略參數進行回測。
- **操作流程**:
  1. 選擇 `Exchange` (交易所) 與 `Symbol` (交易對)。
  2. 設定 `Timeframe` (如 1h, 15m)。
  3. 調整策略參數 (Parameters)。
  4. 點擊 "Run Backtest" 查看權益曲線 (Equity Curve)。

## 3. 即時監控 (Live Monitor)

**路徑**: `/monitor` (規劃中)

- **功能**: 查看實盤運行的策略狀態與信號。

---

# Part 2: 管理員指南 (Admin's Guide)

此區塊說明系統參數配置、數據源管理與故障排除。

## 1. 系統參數設定 (System Settings)

**路徑**: `/settings`
**權限**: 僅限管理員

此頁面控制系統的全域行為，修改後通常立即生效。

### 核心設定項目

| Key | 範例值 | 說明 |
| :--- | :--- | :--- |
| `site.name` | "Crypto-QA" | 網站顯示名稱 |
| `market.data.retention_days` | `90` | Parquet 熱數據保留天數 |
| `trade.risk.max_drawdown` | `0.05` | (5%) 每日最大虧損限制 (Kill Switch) |
| `system.maintenance_mode` | `false` | 若設為 true，將停止所有自動下單 |

### 操作說明

1. **編輯**: 點擊列表右側 "編輯"，修改 Value 欄位。
2. **JSON 格式**: 若欄位為複雜設定 (如 `{"a":1}`)，請嚴格遵守 JSON 雙引號格式。
3. **稽核**: 系統會自動記錄最後修改者 (`Updated by`) 與時間。

## 2. 資料源管理 (Data Manager)

**路徑**: `/data-manager`

### 交易所串接 (Exchange Integration)

- **新增交易所**: 在此輸入 API Key / Secret (資料將加密儲存)。
- **連線測試**: 點擊 "Test Connection" 確認連線狀態。

### 數據維護 (Data Maintenance)

- **手動同步**: 當自動任務失敗時，可在此手動觸發 `Sync` 任務補齊 K 線資料。
- **Parquet 檢視**: 檢查本地數據湖的檔案完整性。

## 3. 故障排除 (Troubleshooting)

### API 連線失敗

- **症狀**: 前端顯示 "Network Error"。
- **檢查**: 
  1. Docker 容器 `qa_backend` 是否健康 (`docker compose ps`)。
  2. 瀏覽器 Console 是否有 CORS 錯誤。
  3. 確認 `.env` 中的 `PUBLIC_API_URL` 設定正確。

### 資料庫遷移 (Migration)

若更新代碼後發現功能異常，可能需要執行資料庫遷移：

```bash
docker compose exec backend uv run python manage.py migrate
```

---

# Part 3: 發展人員指南 (Developer Guide)

## 新增交易所
- 必須繼承 BaseExchangeClient，並實作 fetch_ohlcv 與 download_history。
- 務必使用 async with 呼叫以避免連線洩漏。
