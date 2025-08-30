**概要**
- Cloudflare D1 データベースに保存したポートフォリオを API 経由で取得し、現在価格で評価額を計算します。
- 価格取得は Alpha Vantage API または yfinance を使用できます（既定は yfinance）。

**ファイル構成**
- `report_db.html`: APIから取得したポートフォリオを表示するフロントエンド
- `portfolio_notify.py`: メインスクリプト
- `config.example.json`: 設定のサンプル
- `portfolio.example.csv`: ポートフォリオCSVのサンプル

**前提**
- Python 3.8+
- 依存ライブラリ:
  - `yfinance`（推奨・デフォルト）: `pip install yfinance`
  - `requests`（Alpha Vantageを使う場合のみ）

**セットアップ**
- 設定ファイルを作成/編集:
  - `config.json` を編集（価格プロバイダ等を設定）
- Cloudflare D1 にポートフォリオDBを作成:
  - `schema.sql` を用いて `holdings` テーブルを作成
  - `symbol,shares,currency` を登録
- 機密情報は環境変数でも上書き可能:
  - `PN_ALPHA_VANTAGE_KEY` … Alpha Vantage APIキー

**秘密情報の秘匿（.env推奨）**
- このリポジトリは `.env` を自動読み込みします（外部ライブラリ不要）。
- 手順:
  1) `.env.example` を `.env` にコピーし、必要な値を記入
  2) `config.json` からパスワード等の秘匿情報を空にする（`.env` が優先されます）
  3) `.env` は Git 追跡除外済み（`.gitignore`）
- 利用する主なキー:
  - （任意）`PN_ALPHA_VANTAGE_KEY`

**実行**
- 標準出力に結果表示:
  - `python3 portfolio_notify.py --config config.json --portfolio-url https://<worker>/api/portfolio`

**CSV → DB 反映（同期）**
- `portfolio.csv` の内容を D1（`/api/portfolio`）へ反映する補助スクリプトを追加しました。
  - 事前に Cloudflare Workers を `wrangler dev` でローカル起動するか、デプロイ済みのURLを指定してください。
  - 例（ローカル dev に反映、DBをCSVで置き換え）:
    - `python3 scripts/sync_portfolio_csv.py --csv portfolio.csv --api http://127.0.0.1:8787/api/portfolio --mode replace`
  - 例（本番Workerに upsert のみ）:
    - `python3 scripts/sync_portfolio_csv.py --csv portfolio.csv --api https://<your-worker>.workers.dev/api/portfolio --mode upsert`
  - 先に差分だけ見たい場合:
    - `python3 scripts/sync_portfolio_csv.py --csv portfolio.csv --api http://127.0.0.1:8787/api/portfolio --mode replace --dry-run`

  - Workerのデプロイもこのコマンドから実行可能（`--deploy`）:
    - `python3 scripts/sync_portfolio_csv.py \
        --deploy --worker tight-truth-243e \
        --csv portfolio.csv \
        --api https://tight-truth-243e.<your-account>.workers.dev/api/portfolio \
        --mode replace`
    - 内部で `wrangler deploy --name tight-truth-243e` を `proxy/` ディレクトリで実行します。

注意（Fundamentalsについて）
- Fundamentals（純利益・発行済株式数など）のAPI自動取得は廃止しました。
- 行の作成は `/admin/fundamentals-sync`、値の投入は手動で `/api/fundamentals` にPOSTしてください。
- 以前の自動バックフィル用エンドポイントは削除済みです。


**混在ポートフォリオ（米国株+日本株）**
- DB の `currency` 列で銘柄ごとに通貨を指定できます（`USD` または `JPY`）。
- レポートは以下の列順で表示します:
  - `SYMBOL`, `SHARES`, `USD_PRICE`, `USD_VALUE`, `JPY_PRICE`, `JPY_VALUE`
- 為替レートは Alpha Vantage の `CURRENCY_EXCHANGE_RATE` を使用（USD→JPYを1回取得）。
- 注意: 現状サポート通貨は USD/JPY のみです。


**価格プロバイダ**
- 既定は `yfinance`（APIキー不要）。`config.json` の `price_provider.type` を `"yfinance"` にするか、未指定なら自動で `yfinance` を使用します。
- Alpha Vantageを使う場合は `price_provider.type` を `"alpha_vantage"` にし、APIキーを設定してください（レート制限あり）。

**自動実行（cronの例）**
- 毎営業日 16:30 に実行して標準出力/HTML保存:
  - `crontab -e`
  - 例: `30 16 * * 1-5 PN_ALPHA_VANTAGE_KEY=... /usr/bin/python3 /path/to/portfolio_notify.py --config /path/to/config.json --portfolio-url https://<worker>/api/portfolio --save-html /path/to/report_db.html`

**制限・拡張**
- 現在は通貨換算を行わず、各銘柄のクォート通貨で集計します（USD等）。


## フロントエンド

`report_db.html` は Cloudflare D1 に保存されたポートフォリオを API 経由で取得して表示する単一ページアプリです。
`?api=` クエリまたは `localStorage.PF_API_BASE` に API のベース URL を指定して利用してください。
