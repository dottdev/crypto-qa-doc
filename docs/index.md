# crypto-qa

Crypto Quant Analysys 是一個量化交易分析平台，採用 Django Ninja 為後端，
SvelteKit + shadcn 為前端使用者介面，並使用 Parquet 構建本地數據池，
旨在於提供高效的加密貨幣數據清洗、回測與即時監控，並進一步達到可以自動交易功能。

## 🛠 Tech Stack

- **Backend:** Python 3.12, Django, Django Ninja, Uv
- **Frontend:** Svelte 5, SvelteKit, TypeScript, TailwindCSS, Shadcn/ui
- **Data:** Parquet, Pandas/Polars, Postgres, Redis

---

## 🚀 Development envirionment

確保系統已安裝 [uv](https://github.com/astral-sh/uv) 與 [pnpm](https://pnpm.io/)。

### 1. Initialization

```bash
# Clone repository
git clone <YOUR_REPO_URL>
cd crypto-qa

# Backup Setup (using uv)
cd backend
uv sync

# Frontend Setup
cd ../frontend
pnpm install
```

### 2. Run Services (Local)

建議開啟三個終端機視窗分別執行：

#### Terminal 1: Docker DB Service

```bash
docker compose up -d db redis
```

#### Terminal 2: Backend

```bash
cd backend
# Run migration and start server
uv run python manage.py migrate
uv run python manage.py runserver 0.0.0.0:8000
```

#### Terminal 3: Frontend

```bash
cd frontend
pnpm run dev
```

### 3. Run with Docker (Optional)

#### 情境1: 全端跑在 Docker (產品模式/預覽)

```bash
docker compose --profile all up -d
```

#### 情境2: 混合開發 (Hybrid Dev) 只想用 Docker 跑 DB 和 Redis，程式碼在 Local 跑

```bash
docker compose --profile
```

## 🔌 API design

所有 API 均基於 Django Ninja，預設前綴為 `/api`

### 統一回應格式 (Envelope Pattern)

為了前後端串接的一致性，所有 API (除特定二進位流或非標準介面) 皆採用以下 JSON 結構：

```json
{
    "code": 0,              // 0: 成功, -1: 錯誤, >0: 其他錯誤代碼
    "message": "success",   // 人類可讀訊息
    "data": { ... }         // 實際資料 payload
}
```

### 1. System Config (系統設定)

定義於 `backend/apps/system/api.py`。
負責本系統相關設置

| Method | Endpoint | 功能說明 | Body / Query | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/sys/health` | 系統健康檢查。 | docker Healthcheck 用 | ✅ 完成 |
| **GET** | `/sys/info` | 系統資源狀態。 | CPU, Memory, Disk | ⏳ 未開始 |
| **GET** | `/sys/settings` | 列出系統所有設定。 | - | ✅ 完成 |
| **GET** | `/sys/settings/{key}` | 獲取單一設定詳情。 | - | ✅ 完成 |
| **PUT** | `/sys/settings/{key}` | 更新系統設定。 | `SystemSettingSchema` | ✅ 完成 |

### 2. Market Data (市場數據):

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

### 3. Analysis (量化分析與回測)

定義於 `backend/apps/analysis/api.py`。
負責技術指標計算與策略回測。

| Method | Endpoint | 功能說明 | Body / Query | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/ana/indicators` | 獲取系統支援的技術指標清單 (如 MA, RSI) | - | ⏳ 未開始 |
| **POST** | `/ana/preview` | 快速計算指標並回傳 (不存 DB，前端預覽用) | `{ "symbol": "BTC/USDT", "indicators": [...] }` | ⏳ 未開始 |
| **POST** | `/ana/backtest/run` | **【核心】** 提交回測任務 | `BacktestConfigSchema` | ⏳ 未開始 |
| **GET** | `/ana/backtest/{id}` | 查詢回測狀態與簡易結果 | - | ⏳ 未開始 |
| **GET** | `/ana/backtest/{id}/result`| 獲取詳細回測報告 (權益曲線、交易明細) | - | ⏳ 未開始 |

### 4. Trading (實盤/模擬交易)

定義於 `backend/apps/trading/api.py`。負責對接交易所 API 進行下單與帳戶管理。

| Method | Endpoint | 功能說明 | Body / Query | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/trade/accounts` | 獲取交易所帳戶餘額與權益 | `?exchange=binance` | ⏳ 未開始 |
| **GET** | `/trade/positions` | 獲取當前持倉 (Positions) | - | ⏳ 未開始 |
| **POST** | `/trade/order` | 發送下單請求 (市價/限價) | `OrderCreateSchema` | ⏳ 未開始 |
| **DELETE**| `/trade/order/{id}` | 取消訂單 | - | ⏳ 未開始 |
| **GET** | `/trade/history` | 獲取歷史成交紀錄 | `?limit=50` | ⏳ 未開始 |

---

## 🏗 Data Schemas (Pydantic Models)

以下定義核心的 Request/Response Schema

### Core Schema

```python
class ApiResponse(Schema, Generic[T]):
    code: int = 0               # 0: Success, -1: Error, >0: Warning/Specific
    message: str = "success"
    data: Optional[T] = None
```

### System Schemas

```python
class SystemSettingOut(Schema):
    key: str
    value: Any
    description: str
    updated_at: datetime
    updated_by_name:

class SystemSettingResponse(ApiSchema[SystemSettingOut]):
    pass
```

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

## Frondend Design

| 名稱 | API 整合 | 功能說明 | 狀態 |
| :--- | :--- | :--- | :--- |
| TradingView | - | Lightweight Charts 整合 | ⏳ 未開始 |

## 📂 file structure

### 主程式檔案結購

```Plaintext
crypto-qa/
├── .git/
├── .gitignore ✅           # 全局忽略 (忽略 data/, venv/, node_modules/)
├── .env ✅                 # 全局環境變數 (DB 密碼, API Keys)
├── docker-compose.yml ✅   # 編排服務 (Postgres, Redis, Backend, Frontend)
├── README.md 🚧            # 也提供 AI 可以了解目前執行狀態和規劃
├── ROADMAP.md 🚧           # 提供 AI 可以了解目前執行規劃的進行方式
│
├── 📂 .vscode/             # 發展環境設定
│   ├── launch.json ✅
│   ├── settings.json ✅
│   └── tasks.json ✅
│
├── 📂 data/                    # 【本地資料湖】 (Data Lake)
│   ├── .gitkeep ⏳
│   ├── raw/                    # 原始數據 (API 下載的原始 CSV/JSON)
│   ├── 📂 parquet_hot/ ⏳      # 清洗後的 Parquet 檔案 (近期: 暫定半年內)
│   │   └── 📂 binance/
│   │       └── 📂 spot/
│   │           └── 📂 BTCUSDC/
│   │               └── 📂 1h/
│   │
│   └── 📂 parquet_cold/        # 清洗後的 Parquet 檔案 (歷史: 暫定半年以上)
│       ├── 📂 binance/         # 來源/交易所1
│       │   └── 📂 spot/
│       │       └── 📂 BTCUSDC/
│       │           └── 📂 1d/
│       └── 📂 okx/             # 來源/交易所2
│           └── 📂 spot/
│               └── 📂 BTCUSDC/
│                   └── 📂 1d/
│
├── 📂 backend/            # 【後端】 Django + Ninja (Python Environment)
│   ├── pyproject.toml     # uv 依賴管理
│   ├── uv.lock
│   ├── manage.py
│   ├── Dockerfile         # Backend container
│   ├── conf.py            # 避免程式中有寫死的設定。例如：內部根目錄
│   ├── test_conn.py       # 手動測試交易所的連線與讀取資料功能
│   │ 
│   ├── 📂 core/           # Django 核心設定 (settings, wsgi, asgi)
│   │   ├── api.py ✅      # 將各 apps 的 API 整合於此處
│   │   ├── asgi.py
│   │   ├── schemas.py 🚧  # 全域使用的 Schema: ApiResponse
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   │
│   ├── 📂 apps/           # Django Apps (依據「領域」劃分，而非功能)
│   │   ├── 📂 system/        # 負責：系統相關 API
│   │   │   ├──api.py 🚧         # 定義 API
│   │   │   ├──models.py 🚧      # 定義資料庫資料型式 (Settings)
│   │   │   └──schemas.py 🚧     # 定義 API 所使用的 schema
│   │   │
│   │   ├── 📂 market_data/   # 負責：下載數據、讀寫 Parquet、提供 K 線 API
│   │   │   ├──api.py 🚧         # 定義 API
│   │   │   ├──models.py 🚧      # 定義資料庫資料型式
│   │   │   ├──schemas.py 🚧     # 定義 API 所使用的 schema
│   │   │   └──services.py ⏳    # 定義相關服務函數
│   │   │
│   │   ├── 📂 analysis/ ⏳   # 負責：技術指標計算、回測引擎
│   │   ├── 📂 trading/ ⏳    # 負責：交易所對接、下單、帳戶餘額
│   │   └── 📂 dashboard/ ⏳  # 負責：前端需要的非業務 API (如 User settings)
│   │
│   └── 📂 services/        # 【Service Layer】 純 Python 邏輯 (與 Django 解耦)
│       ├── 📂 exchanges/           # 負責：交易所相關服務函式
│       │   ├──base.py ✅           # 負責：基本定義
│       │   ├──binance.py 🚧        # 負責：BinanceClient
│       │   └──factory.py 🚧        # 負責：定義產生交易所 Client 的類別
│       ├── indicator_calc.py
│       ├── market_data.py          # 處理 apps/market_data 的邏輯
│       └── parquet_manager.py 🚧
│
└── 📂 frontend/           # 【前端】 SvelteKit + Shadcn (Node Environment)
    ├── .env               # frontend 環境變數
    ├── .gitignore
    ├── package.json
    ├── pnpm-lock.json
    ├── svelte.config.js
    ├── tailwind.config.js
    ├── components.json    # shadcn 設定檔
    ├── Dockerfile         # 產品 container 設定檔
    │
    └── 📂 src/
        ├── app.html
        ├── 📂 lib/        # Svelte 核心邏輯庫
        │   ├── 📂 api/    # 封裝 fetch 請求 (對接 Django Ninja)
        │   │   ├── types.ts           # 呼叫後端 API 的參數設定
        │   │   ├── config.ts          # svelte 呼叫的 API 設定
        │   │   └── market.ts          # 處理 market_data/api.py 的請求
        │   │
        │   ├── 📂 components/
        │   │   ├── 📂 data-manager/   # Data Manager 專屬組件，避免汗染全域元件
        │   │   │   ├── exchange-card.svelte        # 單個交易所的狀態卡片
        │   │   │   ├── add-exchange-dialog.svelte  # 搜並新增交易所的彈窗
        │   │   │   └── connection-status.svelte    # 顯示連線測試結果
        │   │   │
        │   │   └── 📂 charts/ # 封裝 Lightweight Charts
        │   │
        │   └── 📂 stores/ # Svelte Stores (全局狀態)
        │
        └── 📂 routes/     # 頁面路由 (File-based routing)
            ├── +layout.svelte
            ├── +page.svelte            # 首頁 (Dashboard)
            ├── 📂 backtest/            # 回測頁
            │
            └── 📂 data-manager/
                ├── +layout.svelte
                ├── +page.svelte        # 資料管理首頁 (Data Manager)
                │
                ├── 📂 sources/         # 【第一階段目標】交易所管理
                │   └── +page.svelte
                │
                ├── 📂 market/          # 【下一階段】交易對 (Symbol) 管理與下載
                │   ├── +page.svelte    # 列表與選擇
                │   └── 📂 [symbol]/    # 單一幣種詳細數據檢視
                │       └── +page.svelte
                │
                └── 📂 tasks/           # 【未來規劃】背景任務監控 (Celery/Redis Queue)
                    └── +page.svelte
```

### Parquet 檔案結購

```plaintext
data/
└── parquet/
    ├── binance/
    │   └── BTCUSDT/
    │       └── 1h/
    │           └── data.parquet
    └── bybit/
        └── BTCUSDT/
            └── ...
```
