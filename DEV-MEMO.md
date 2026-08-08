# DEV-MEMO.md

Cloudflare OS（`cloudflare/cloudflare-os`）を自分のデバイス（macOS / QNAP NAS）で
動かすための技術解説付きセットアップメモ。同時に、GitHub Pages 上に
LP兼チュートリアル（Quarto）を公開する手順もまとめる。

## 目的

- **TypeA（macOS）**: ローカルで Cloudflare OS を動かし、動作・機能を検証する。
- **TypeB（QNAP NAS）**: 常時稼働するセルフホスト環境として運用する可能性まで探る。
- 公式クラウド版（`os.cloudflare.app`）ではなく、自前で運用した場合の
  **メリット・デメリットを両面から考察**できる材料を残す。

## 環境

| ツール | バージョン | 備考 |
|---|---|---|
| Node.js | 20.19+ / 22.12+ | Vite 7・Wrangler 4 の要求。22 LTS 推奨 |
| pnpm | 11.17.0 | リポジトリの `packageManager` 指定に合わせる（corepack） |
| git | 2.x | Cloudflare OS の clone / Pages 公開用 |
| Quarto | >= 1.9.20 | LP／チュートリアル（本リポジトリ）の生成 |
| uv | >= 0.5 | Quarto サイトの GH Actions 用 Python 環境 |
| workerd | — | Wrangler 内包（`pnpm run-local` が内部利用） |

### Node.js / pnpm のセットアップ

```bash
# corepack 経由で pnpm@11.17.0 を揃える（packageManager 記載と一致させる）
corepack enable
corepack prepare pnpm@11.17.0 --activate
pnpm --version  # => 11.17.0
```

## 全体像（技術解説）

Cloudflare OS は Cloudflare Workers 上に構築された「AI 生産性 OS」。以下の用語が出てくる:

| 用語 | 本体 | 説明 |
|---|---|---|
| **Kernel** | `packages/workshop-backend` | ユーザー・Gadget・Gatekeeper を結び付け、サンドボックス化とアクセス制御を実装 |
| **Shell** | `packages/workshop-frontend` | エンドユーザー向け Web UI（エージェントチャット＋Gadget 一覧） |
| **Process** | gadgets | 「AI が作ってくれた小さな個人アプリ」= 独立サンドボックスで動く |
| **Executable** | blueprints | アプリのコード群を再利用可能なテンプレート化したもの |
| **Driver** | `packages/gatekeeper-*` | 外部サービス（GitHub / Google / Notion 等）への接続を司る独立 Worker |
| **ACL** | 共有パーミッション | Gadget の共有・共同編集 |

技術コンテナである理由:

- **Durable Objects**: 各 Gadget／ワークスペースは DO に裏打ちされ、リアルタイム複数人同時編集を実現。
- **Dynamic Workers / Facets**: 各 Gadget を動的に立ち上がる Worker Facet としてサンドボックス実行。
- **Cap'n Web RPC**: クライアント〜サーバ間の低 boilerplate 通信プロトコル。Gadget の API がそのままエージェントから呼べる。

この設計により「エージェントが書いたコード」を**インターネットに接続できないサンドボックス**で
安全に実行できるのが売りである。

## TypeA: macOS ローカルで動かす

### 手順

```bash
git clone https://github.com/cloudflare/cloudflare-os.git
cd cloudflare-os
pnpm install                 # モノレポの依存を一括導入
pnpm run-local               # 全部の Worker をローカル起動
```

`http://localhost:8787` を開くと UI が表示される。

※ 初回は `wrangler login` は不要（`run-local` は Cloudflare アカウント無しで動く）。

### `pnpm run-local` は何をしているか（重要）

`pnpm run-local` = `node scripts/run-local.mjs` 実行。具体的には:

1. `wrangler.dev.jsonc`（gitignored）たちを自動生成
   - 全 `packages/gatekeeper-*` を探索し、service binding を `wrangler.dev.jsonc` に注入
   - `workshop-backend` にも GatekeeperVendor 経由で全 gatekeeper を bind
   - `ADMINS=["admin"]` を自動追加（= admin アカウント）
2. `--serve-frontend-assets` を有効化 → backend が `workshop-frontend/dist` を静的配信
3. `wrangler dev` を**マルチコンフィグ**（`-c` を複数指定）で同時起動

つまり「Web サーバー 1 個」ではなく、**開発用 Router + バックエンド + 各 Gatekeeper が
1つの workerd ランタイム内で同時に動く**。

### データの保存場所

- `.wrangler/` ディレクトリが生成され、Durable Objects のローカル永続化がここに置かれる。
- 消すとワークスペース・Gadget がリセットされる（バックアップ対象）。

### `.dev.vars`（注意）

- gitignored の秘密情報ファイル。外部サービスの OAuth や token を書くと
  `run-local` が読み込み、各 Worker に注入する。
- 例: `GITHUB_CLIENT_ID=...` / `GITHUB_CLIENT_SECRET=...`

### 開発モード（構成を分ける場合）

```bash
pnpm dev-server   # backend を wrangler dev で起動（API 側）
pnpm dev-client   # frontend を Vite で起動（http://localhost:3000）
```

- `run-local`: ビルド済みフロントをバックエンドが配信（本番に近い動作）
- `dev-*`: Vite がフロントを配信（ホットリロードで開発しやすい）

### `run-local` の追加オプション

- **`--use-workers-ai-binding`**: Workers AI を backend にバインドして使う場合。
  - ⚠️ **このオプションを使う場合のみ `wrangler login`（Cloudflare アカウント）が必要**。
    通常の起動はアカウント不要（本文の「login 不要」はこの通常ケースを指す）。
- **`VITE_BACKEND_HOST`**: 開発時のバックエンドホスト指定。
  - 例: `VITE_BACKEND_HOST=localhost:9000`（ポート 9000 を使う場合）。
    このとき`--port`も wrangler へ渡される。

## TypeB: QNAP NAS でセルフホスト

### 最初に知っておくべき制約（重要）

- **本番の `workerd` デプロイは「COMING SOON」** のまま。公式は
  「workerd で本番デプロイする手順は未整備」。
- GitHub Pages などの単純静的配信は**無理**（DO / Workers ランタイムが必要）。

よって QNAP 運用は「**開発用の `pnpm run-local` を常時起動して持ち上げる**」のが現実的。

### ルート1: Container Station で Node コンテナを常時起動（推奨）

QNAP の Container Station 上に Node 22（or 24）コンテナを作成する。

```bash
# コンテナ内
apt-get update && apt-get install -y git
corepack enable
git clone https://github.com/cloudflare/cloudflare-os.git
cd cloudflare-os && pnpm install --frozen-lockfile
pnpm run-local &
```

- x86_64（Intel / AMD QNAP）前提。**ARM（armv8 / arm64）の QNAP は未検証**。
- ポート 8787 を LAN に公開（コンテナのポートマッピング）。
- Container Station は「再起動時に自動開始」にすれば常駐化できる。

### ルート2: Entware / opkg 経由で Node.js 直接

```bash
# QNAP 上
opkg install nodejs npm corepack
```

→ パッケージ管理の競合が生じるため、**おすすめはルート1（Docker）**。

### LAN / 外部公開

- **LAN 内**: `http://<NAS-IP>:8787` で使える。
- **外部公開** したい場合は**必ず HTTPS**（OAuth クライアントは http を拒否する場合が多い）。
   - QNAP reverse proxy や Caddy / nginx で TLS 終端。
   - **Cloudflare Tunnel（`cloudflared`）** が楽（クレデンシャル不要で公開）。

### 認証まわり

- 既定 `ADMINS=["admin"]` のまま外部公開すると誰でもログインできる（開発用構成）。
- 外部公開時は admin を外す／強いパスワードを設定する。

## Gatekeeper の設定（外部サービス連携）

外部サービス連携の際は各 `packages/gatekeeper-*` の README に従う。

同梱の Gatekeeper（README 準拠）:
GitHub / Google / Cloudflare / Supabase / Notion / Confluence / Email Workers /
Home Assistant / Slack / Spotify / ZoomInfo。

- 例: GitHub 連携 → OAuth アプリを作成し、取得した Client ID / Secret を `.dev.vars` に設定。
- OAuth コールバック URL は https でないと受け付けないサービスが多い。

## 既知の制約（重要）

| 制約 | 影響 |
|---|---|
| workerd 本番デプロイは COMING SOON | セルフホストは「開発実行の常駐」としてしか運用できない（現時点） |
| ARM 端末（arm64） | 依存ビルドで苦労する可能性（x86_64 前提） |
| `admin` 自動アカウント | 放置すると外部に露出危険 |
| OAuth コールバック | 外部公開時は TLS（https）必須 |

## GitHub Pages へ公開（Quarto）

Cloudflare OS 本体には以下のデプロイ選択肢がある（本リポジトリは Playground として利用）:

- **公式のクラウドデプロイ**: https://os.cloudflare.app/deploy
  （自分の Cloudflare アカウントへ最短デプロイ）
- **上級デプロイ用スターター**: https://github.com/cloudflare/cloudflare-os-starter
  （Gatekeeper 込み・コード変更を伴うデプロイ）
- **本リポジトリの用途**: 開発用 `run-local` をローカル（macOS）／NAS（TypeB）で常駐させて検証。

本リポジトリ自身は Quarto でサイト化し、GH Pages にデプロイする。

```bash
quarto render      # _site/ を生成
quarto preview     # ローカルプレビュー
```

デプロイは `.github/workflows/gh-pages.yml`（`setup-uv` + `setup-quarto` + `uv sync --frozen` +
`quarto render` → `_site` を `upload-pages-artifact`）で行う。

### 公開 URL

- サイト: **https://watanabe3tipapa.github.io/cfos-sandbox/**
- リポジトリ: **https://github.com/watanabe3tipapa/cfos-sandbox**

## 実装チェックリスト（2026-08-08 時点）

- [x] Quarto サイト（LP＋チュートリアル4本）
- [x] `_quarto.yml` / styles / favicon
- [x] `quarto render` ビルド確認（5ページ HTTP 200）
- [x] 独立リポジトリ化 → `gh repo create` → push
- [x] Pages 設定（build_type: workflow）＋デプロイ成功
- [ ] TypeA: macOS で `pnpm run-local` の動作確認
- [ ] TypeB: QNAP 上の Node コンテナで常駐起動を試験
- [ ] Gatekeeper（GitHub）設定試験

## 更新履歴

- 2026-08-08: 初版作成。
- 2026-08-08: Quarto サイト（LP＋ガイド4本）実装、ビルド検証。
- 2026-08-08: 独立リポジトリ化＆Pages デプロイ完了。
- 2026-08-08: PLAN.md を rm（本メモは復活）。
- 2026-08-08: **再検証**（内容の事実確認と更新）。
  - Gatekeeper 一覧を公式 README 準拠へ（email / spotify 追加）
  - `run-local` のオプション（`--use-workers-ai-binding` / `VITE_BACKEND_HOST`）を補足
  - デプロイ選択肢（os.cloudflare.app/deploy / cloudflare-os-starter）を追記
  - GH Actions を最新バージョンへ更新（checkout v5 / setup-uv v9 / configure-pages v6 / upload-pages-artifact v5 / deploy-pages v5）