# Crypto-QA 使用者操作手冊 (User Manual)

本手冊分為兩大部分：

1. [交易員指南 (Trader's Guide)](#part-1-交易員指南-traders-guide)：針對策略開發、回測與日常監控的操作。
2. [管理員指南 (Admin's Guide)](#part-2-管理員指南-admins-guide)：針對系統設定、數據維護與環境部署的操作。

---

# Part 1: 交易員指南 (Trader's Guide)

此區塊說明如何使用平台進行市場分析、回測與策略執行。

## 1. 儀表板 (Dashboard)

**路徑**: `/` (首頁)

* **總覽**: 顯示目前帳戶權益、活躍策略數量與損益狀態。
* **導航**: 點擊左上角圖示可收合側邊欄 (Sidebar)，提供更大的圖表檢視空間。

## 2. 量化分析 (Analysis & Backtest)

**路徑**: `/backtest`

* **功能**: 透過歷史數據驗證交易策略的歷史表現。
* **操作步驟**:

1. **選擇標的**: 設定交易所 (Exchange) 與交易對 (Symbol)。
2. **設定週期**: 選擇 Timeframe (如 1h, 15m)。
3. **策略參數**: 調整右側面板的策略參數 (Parameters)。
4. **執行回測**: 點擊 "Run Backtest" 系統將計算權益曲線 (Equity Curve)、最大回撤 (MOD) 與夏普比率。

## 3. 即時監控 (Live Monitor)

**路徑**: `/monitor` (規劃中)

* **功能**: 查看實盤運行的策略狀態、即時信號觸發與訂單執行狀況。

---

# Part 2: 管理員指南 (Admin's Guide)

此區塊說明系統參數配置、數據源管理與故障排除，僅限管理員權限存取。

## 1. 系統參數設定 (System Settings)

**路徑**: `/settings`
**權限**: 僅限管理員

此頁面控制系統的全域行為。修改後的設定通常會立即生效，並記錄修改者與時間。

| Key | 說明 | 範例值 |
| :--- | :--- | :--- |
| `site.name` | 網站顯示名稱 | "Crypto-QA" |
| `market.data.retention_days` | Parquet 熱數據保留天數 | `90` |
| `trade.risk.max_drawdown` | 每日最大虧損限制 (Kill Switch) | `0.05` (5%) |
| `system.maintenance_mode` | 若設為 true，將停止所有自動下單 | `false` |

## 2. 資料源管理 (Data Manager)

**路徑**: `/data-manager`

* **交易所串接** (Exchange Integration)
  * **新增**: 在此輸入 API Key / Secret (系統採加密儲存)。
  * **測試**: 點擊 "Test Connection" 確認連線狀態與權限。

* **數據維護** (Data Maintenance)
  * **手動同步**: 當自動排程 (Cron) 失敗時，可在此手動觸發 `Sync` 任務補 K 線資料。
  * **完整性檢查**: 檢視本地數據湖 (Parquet) 是否有缺漏。

## 3. 故障排除 (Troubleshooting)

* **API 連線失敗** ("Network Error")
  1. 確認後端服務是否運行正常。
  2. 檢查瀏覽器 Console 是否有 CORS 相關錯誤。
  3. 確認環境變數在 `.env` 中的 `PUBLIC_API_URL` 設定正確。

* **顯示異常**: 嘗試清除瀏覽器快取或重新整理頁面。
