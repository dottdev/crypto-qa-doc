# crypto-qa

Crypto Quant Analysys 是一個量化交易分析平台，採用 Django Ninja 為後端，
SvelteKit + shadcn 為前端使用者介面，並使用 Parquet 構建本地數據池，
旨在於提供高效的加密貨幣數據清洗、回測與即時監控，並進一步達到可以自動交易功能。

## 📚 文檔索引

詳細的使用者手冊、開發指南與 API 說明，請參閱 [Crypto-QA Documentation](https://your-username.github.io/crypto-qa/)。

## 📖 Documentation

- **[MANUAL.md](./MANUAL.md)**: **使用者操作手冊** (公開)。適合交易員與系統管理員。
- **[DEVELOPER.md](./DEVELOPMENT.md)**: **開發者指南** (內部)。包含架構設計、目錄結構 Coding Style。
- **[ROADMAP.md](./ROADMAP.md)**: **發展路線圖**，專案發展路線圖與技術架構細節。
- **[API.md](./API.md)**: **API 規格書**。目前後端 REST API 介面定義與 Schema 規劃及進程。

## 🚀 快速啟動

詳細環境建置與指令請參閱 `DEVELOPMENT.md`。

1. **初始化**: 第一次安裝系統

    ```bash
    # Clone repository
    git clone <YOUR_REPO_URL>
    cd backend && uv sync
    cd ../frontend && pnpm install
    ```

2. **啟動服務**: 發展環境中，要啟用系統服務

    建議開啟三個終端機視窗分別執行：

   - Terminal 1: Docker DB Service

    ```bash
    docker compose up -d db redis
    ```

   - Terminal 2: Backend

    ```bash
    cd backend
    # Run migration and start server
    uv run python manage.py migrate
    uv run python manage.py runserver 0.0.0.0:8000
    ```

    - Terminal 3: Frontend

    ```bash
    cd frontend
    pnpm run dev
    ```

3. **初始化設定**: Database Seeding

    系統包含自動資料播種機制。當後端啟動時，會自動執行 `python manage.py seed_settings`。
    這確保了 `SystemSetting` 表格中始終包含系統運作所需的預設參數 (如 `site.name`, `risk.max_drawdown`)。
    若需新增全域預設設定，請修改 `backend/apps/system/management/commands/seed_settings.py`。

4. **產品線啟用**: Run with Docker (Optional)

    * **情境 1**: 全端跑在 Docker (產品模式/預覽)

    ```bash
    docker compose --profile all up -d
    ```

    * **情境 2**: 混合開發 (Hybrid Dev) 只想用 Docker 跑 DB 和 Redis，程式碼在 Local 跑

    ```bash
    docker compose --profile
    ```
