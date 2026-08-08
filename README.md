# cfos-sandbox

Cloudflare OS（[`cloudflare/cloudflare-os`](https://github.com/cloudflare/cloudflare-os)）を
自分のデバイス（macOS / QNAP NAS）で動かすための技術解説付きチュートリアルサイトです。

## 構成

- **LP + チュートリアル**: Quarto（`type: website`）製。`index.qmd` が LP、
  `guides/*.qmd` がチュートリアル4本（macOS / QNAP / Gatekeeper / トラブル）。
- **DEV-MEMO.md**: 技術解説付きのセットアップメモ（主成果物）。

## ローカル手順

> uv はオプション（編集／ビルドには `quarto` のみあれば OK）。

```bash
pnpm exec quarto render      # ⇔ 依存ゼロなので quarto 単体
quarto render                # _site/ を生成
quarto preview               # ローカルプレビュー
```

```bash
uv sync --frozen
uv run quarto render
```

## GitHub Pages 公開

`.github/workflows/gh-pages.yml` が、`uv`（流用版）＋`quarto render` を走らせて
`_site/` を Pages にデプロイします。リポジトリ設定でソースを「GitHub Actions」にしてください。

※ このリポジトリは独立リポジトリ化していません（親リポジトリ管理のディレクトリ）。
  公開する前に `git init`（または独立リポジトリ作成）が必要です。詳細は
  [DEV-MEMO.md](DEV-MEMO.md) を参照。

## ライセンス

本文は CC BY 4.0（参考: ir-qubit に倣う）。
コード部分は Apache-2.0（Cloudflare OS 由来の知見を含む）。