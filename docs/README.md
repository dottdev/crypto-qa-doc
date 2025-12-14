# crypto-qa

Crypto Quant Analysys 是一個量化交易分析平台，採用 Django Ninja 為後端，
SvelteKit + shadcn 為前端使用者介面，並使用 Parquet 構建本地數據池，
旨在於提供高效的加密貨幣數據清洗、回測與即時監控，並進一步達到可以自動交易功能。

## 📚 完整文檔

詳細的使用者手冊、開發指南與 API 說明，請參閱 [Crypto-QA Documentation](https://your-username.github.io/crypto-qa/)。

## 📖 Documentation

- **[ROADMAP.md](./ROADMAP.md)**: 專案發展路線圖與技術架構細節。
- **[MANUAL.md](./MANUAL.md)**: 使用者操作手冊，包含介面功能說明與故障排除。
- **[API.md](./API.md)**: 目前後端 REST API 介面定義與 Schema 規劃及進程。

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

詳細的 Endpoints 列表、Request/Response Schema 以及統一回應格式，請參閱 [API.md](./API.md)。

---

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
├── API.md 🚧               # 提供 REST API 目前發展狀態和規劃
├── MANUAL.md 🚧            # 使用者操作手冊
│
├── 📂 .github/wokflows         # GitHub Workflow
│   └── sync-docs.yml ✅        # 同步此專案文件到 crypto-qa-doc 專案中，用來方便使用者檢閱
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
