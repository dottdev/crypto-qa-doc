# 🔌 API design

所有 API 均基於 Django Ninja，預設前綴為 `/api`

## 統一回應格式 (Envelope Pattern)

為了前後端串接的一致性，所有 API (除特定二進位流或非標準介面) 皆採用以下 JSON 結構：

```json
{
    "code": 0,              // 0: 成功, -1: 錯誤, >0: 其他錯誤代碼
    "message": "success",   // 人類可讀訊息
    "data": { ... }         // 實際資料 payload
}
```

以下定義核心的 Request/Response Schema

### Core Schema

```python
class ApiResponse(Schema, Generic[T]):
    code: int = 0               # 0: 成功, -1: 錯誤, >0: 特定錯誤代碼
    message: str = "success"
    data: Optional[T] = None
```

## 1. System Config (系統設定)

定義於 `backend/apps/system/api.py`。
負責本系統相關設置

| Method | Endpoint | 功能說明 | Body / Query | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/sys/health` | 系統健康檢查。 | docker Healthcheck 用 | ✅ 完成 |
| **GET** | `/sys/info` | 系統資源狀態 (CPU, Memory, Disk) | - | ⏳ 未開始 |
| **GET** | `/sys/settings` | 列出系統所有設定。 | - | ✅ 完成 |
| **GET** | `/sys/settings/{key}` | 獲取單一設定詳情。 | - | ✅ 完成 |
| **PUT** | `/sys/settings/{key}` | 更新系統設定。 | `SystemSettingUpdateSchema` | ✅ 完成 |

### System Schemas

```python
# Update Request (Input)
class SystemSettingUpdateSchema(Schema):
    value: Any                  # 允許不同型態 (int, str, bool, dict)，後端需做驗證
    description: Optional[str] = None

# Response Object (Output)
class SystemSettingOut(Schema):
    key: str
    value: Any                          # 不同 key 的結構可能不同
    description: str
    updated_at: datetime
    updated_by: Optional[str] = None    # 回傳 Username 或 User ID

class SystemSettingResponse(ApiSchema[SystemSettingOut]):
    pass

class SystemSettingListResponse(ApiSchema[list[SystemSettingOuto]]):
    pass
```

## 2. Market Data (市場數據)

定義於 `backend/apps/market_data/api.py`。
負責數據清洗、儲存與讀取 (Parquet)。

| Method | Endpoint | 功能說明 | Body / Query | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/mkt/sources` | 獲取支援的交易所列表 | - | 🚧 進行中 |
| **GET** | `/mkt/sources/discover` | 列出CCXT可用的交易所 | - | ⏳ 未開始 |
| **POST** | `/mkt/sources/{exchange_id}/enable` | 啟用交易所資料 | - | ⏳ 未開始 |
| **GET** | `/mkt/test-connection` | 測試連線 | - | ⏳ 未開始 |
| **GET** | `/mkt/symbols` | 搜尋/列出交易所的交易對 | `?exchange=binance&q=BTC` | 🚧 進行中 |
| **POST** | `/mkt/track` | 將交易對加入本地追蹤清單 (存入 DB) | `{ "symbol": "BTC/USDT", "source": "binance" }` | ⏳ 未開始 |
| **GET** | `/mkt/tracked` | 獲取目前已追蹤的交易對狀態 (數據完整性) | - | ⏳ 未開始 |
| **GET** | `/mkt/candles` | **【核心】** 讀取 K 線數據 (供前端繪圖) | `?symbol=BTC/USDT&tf=1h&start=...&end=...` | ⏳ 未開始 |
| **GET** | `/mkt/sync-test` | 觸發下載並存檔 | - | ⏳ 未開始 |
| **POST** | `/mkt/sync` | 觸發手動數據補全任務 (下載歷史數據) | `SyncRequest` | ⏳ 未開始 |

### Market Data Schemas

```python
# 讀取 K 線回傳格式 (針對 TradingView Lightweight Charts 優化)
class CandleResponse(Schema):
    time: int   # Timestamp (Unix epoch)
    open: float
    high: float
    low: float
    close: float
    volume: float

class ExchangeResponse(Schema):
    id: str                             # "binance"
    name: str                           # "Binance 幣安"
    supported_market_types: list[str]   # ["spot", "usdt_futures"]
    is_active: bool

# 數據同步請求
class SyncRequest(Schema):
    source: str = "binance"
    symbol: str = "BTC/USDC"
    market_type: str = "spot"
    timeframe: str = "1d"
    days: int = 30  # 預設補 30 天

# 數據內容
class SymbolResponse(Schema):
    id: int             # 對應 Symbol 的自動 id
    name: str           # 對應 Symbol.name
    exchange_id: str    # Django Ninja 可自動對應到 s.exchange_id
    market_type: str    # 原本的 market_type
```

## 3. Analysis (量化分析與回測)

定義於 `backend/apps/analysis/api.py`。
負責技術指標計算與策略回測。

| Method | Endpoint | 功能說明 | Body / Query | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/ana/indicators` | 獲取系統支援的技術指標清單 (如 MA, RSI) | - | ⏳ 未開始 |
| **POST** | `/ana/preview` | 快速計算指標並回傳 (不存 DB，前端預覽用) | `{ "symbol": "BTC/USDT", "indicators": [...] }` | ⏳ 未開始 |
| **POST** | `/ana/backtest/run` | **【核心】** 提交回測任務 | `BacktestConfigSchema` | ⏳ 未開始 |
| **GET** | `/ana/backtest/{id}` | 查詢回測狀態與簡易結果 | - | ⏳ 未開始 |
| **GET** | `/ana/backtest/{id}/result`| 獲取詳細回測報告 (權益曲線、交易明細) | - | ⏳ 未開始 |

### Analysis Schemas

```python
# 回測設定
class BacktestConfigSchema(Schema):
    strategy_name: str       # 例如 "RsiStrategy"
    symbol: str              # "ETH/USDT"
    timeframe: str           # "15m"
    start_time: datetime
    end_time: datetime
    initial_capital: float   # 初始資金
    params: Dict[str, Any]   # 策略參數 e.g. {"rsi_period": 14, "overbought": 70}

# 回測結果摘要
class BacktestResultSchema(Schema):
    id: str
    status: str              # "pending", "running", "completed", "failed"
    total_return: float      # 總報酬率 %
    max_drawdown: float      # 最大回撤 %
    win_rate: float          # 勝率
    total_trades: int        # 總交易次數
```

## 4. Trading (實盤/模擬交易)

定義於 `backend/apps/trading/api.py`。負責對接交易所 API 進行下單與帳戶管理。

| Method | Endpoint | 功能說明 | Body / Query | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/trade/accounts` | 獲取交易所帳戶餘額與權益 | `?exchange=binance` | ⏳ 未開始 |
| **GET** | `/trade/positions` | 獲取當前持倉 (Positions) | - | ⏳ 未開始 |
| **POST** | `/trade/order` | 發送下單請求 (市價/限價) | `OrderCreateSchema` | ⏳ 未開始 |
| **DELETE**| `/trade/order/{id}` | 取消訂單 | - | ⏳ 未開始 |
| **GET** | `/trade/history` | 獲取歷史成交紀錄 | `?limit=50` | ⏳ 未開始 |

### Trading Schemas

```python
# 下單請求
class OrderCreateSchema(Schema):
    symbol: str
    side: str                # "buy" or "sell"
    type: str                # "limit" or "market"
    quantity: float
    price: Optional[float]   # 市價單可為 None
    leverage: int = 1        # 槓桿倍數
```
