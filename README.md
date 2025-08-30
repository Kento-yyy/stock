# stock

ポートフォリオをブラウザで表示するシンプルな Web アプリです。Cloudflare D1 に保存した保有銘柄を Cloudflare Workers 経由で取得し、`report.html` で最新株価とともに表示します。

## 主要ファイル
- `schema.sql` — D1 で使用する `holdings` テーブル定義
- `proxy/worker.js` — `/api/portfolio` と株価取得用の Worker
- `report.html` — PWA 対応のポートフォリオレポート
- `portfolio.html` — 保有一覧を確認する簡易ページ
- `service-worker.js`, `manifest.webmanifest` — PWA 用ファイル

## セットアップ
1. **D1 データベース作成**  
   `wrangler d1 create stock-db` 等でデータベースを作成し、`schema.sql` を適用します。
2. **Worker デプロイ**  
   `proxy/worker.js` を D1 バインド付きでデプロイします。`/api/portfolio` が保有銘柄を JSON で返し、`/quote` が株価を取得します。
3. **HTML を配置**  
   `report.html` や `portfolio.html` を静的ホスティングに配置します。

## 使い方
ブラウザで `report.html` を開くと、`/api/portfolio` から保有情報を取得し、現在値を表示します。HTTPS で配信すると PWA としてオフラインでも利用できます。CORS 回避が必要な場合は Worker を経由して `/quote` を利用してください。
