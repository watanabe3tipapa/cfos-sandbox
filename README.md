# cfos-sandbox

Cloudflare OS（[`cloudflare/cloudflare-os`](https://github.com/cloudflare/cloudflare-os)）を
自分のデバイス（macOS / QNAP NAS）で動かすための技術解説付きチュートリアルサイトです。

## 構成

- **LP + チュートリアル**: Quarto（`type: website`）製。`index.qmd` が LP、
  `guides/*.qmd` がチュートリアル4本（macOS / QNAP / Gatekeeper / トラブル）。

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
`_site/` を Pages にデプロイします。

- サイト: https://watanabe3tipapa.github.io/cfos-sandbox/
- Pages 設定はソース「GitHub Actions」方式。

## ライセンス

本文は CC BY 4.0（参考: ir-qubit に倣う）。
コード部分は Apache-2.0（Cloudflare OS 由来の知見を含む）。