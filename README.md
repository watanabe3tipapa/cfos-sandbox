# cfos-sandbox

Cloudflare OS（`cloudflare/cloudflare-os`）を自分のデバイス（macOS / QNAP NAS）で動かすための技術解説付きチュートリアルサイトです。

- 公式リポジトリ: https://github.com/cloudflare/cloudflare-os
- 公開サイト: https://watanabe3tipapa.github.io/cfos-sandbox/

## 概要

このリポジトリは Quarto（type: website）で作成したドキュメントサイトのソースです。Cloudflare OS を個人運用環境で動作させる手順や、QNAP などでの常時稼働構成、Gatekeeper 連携やトラブルシュートなどの解説を収録しています。

## 主な内容

- LP: `index.qmd`
- チュートリアル: `guides/*.qmd`
  - `01-macos.qmd` — macOS で Cloudflare OS を動かす
  - `02-qnap.qmd` — QNAP NAS で常時稼働するセルフホスト運用を構築する
  - `03-gatekeeper.qmd` — アクセス制御の Gatekeeper を連携する
  - `04-troubleshoot.qmd` — トラブルシューティング

その他に、サイト設定（`_quarto.yml`）、カスタムスタイル（`styles/`）、画像（`images/`）などが含まれます。

## ローカルでの編集とビルド

編集・ビルドには Quarto のみで動作します。pnpm 等の追加パッケージマネージャは不要です。

ローカルプレビュー:

```bash
quarto preview   # ローカルプレビュー（例: http://localhost:8080）
```

サイト生成:

```bash
quarto render    # _site/ を生成
```

CI と同じ環境を Python で再現するための一例として、このリポジトリには `uv` を使う手順も含まれます（任意）:

```bash
uv sync --frozen
uv run quarto render
```

## GitHub Pages での公開

`.github/workflows/gh-pages.yml` により、`uv` と `quarto render` を実行して `_site/` を GitHub Pages にデプロイするワークフローが設定されています。

公開サイト: https://watanabe3tipapa.github.io/cfos-sandbox/

Pages の公開方式は「GitHub Actions」を使用しています。

## 環境情報

- Quarto（type: website）で構築されています。
- このリポジトリの `pyproject.toml` には Python の要件 `requires-python = ">=3.13"` が記載されており、依存関係は空（dependencies = []）です。

## 開発・保守状態

- リポジトリはアーカイブされていません。最終更新はリポジトリの更新日時をご確認ください。
- サイトは GitHub Pages を用いて公開されています。

（詳細なメンテナンス方針や貢献手順はリポジトリ内のファイルを参照してください。）

## ライセンス

- 本文: CC BY 4.0
- コード部分: Apache-2.0（Cloudflare OS 由来の知見を含む）

---

公式リポジトリ: https://github.com/cloudflare/cloudflare-os

公開サイト: https://watanabe3tipapa.github.io/cfos-sandbox/
