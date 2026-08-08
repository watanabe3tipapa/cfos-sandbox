# cfos-sandbox

Cloudflare OS（[`cloudflare/cloudflare-os`](https://github.com/cloudflare/cloudflare-os)）を
自分のデバイス（macOS / QNAP NAS）で動かすための技術解説付きチュートリアルサイトです。

- 公式リポジトリ: https://github.com/cloudflare/cloudflare-os
- 公開サイト: https://watanabe3tipapa.github.io/cfos-sandbox/

## 構成

Quarto（`type: website`）製のサイトです。

- **LP**: `index.qmd`
- **チュートリアル**: `guides/*.qmd`
  - `01-macos.qmd` — macOS で Cloudflare OS を動かす
  - `02-qnap.qmd` — QNAP NAS で常時稼働するセルフホスト運用を構築する
  - `03-gatekeeper.qmd` — アクセス制御の Gatekeeper を連携する
  - `04-troubleshoot.qmd` — トラブルシューティング

## ローカルで動かす

編集／ビルドには **quarto のみ**で OK（pnpm は不要。依存関係はゼロ）。

```bash
quarto preview   # ローカルプレビュー（http://localhost:8080 など）
quarto render    # _site/ を生成
```

CI（GitHub Pages）と同じ環境を Python で再現する場合は uv を使用します（ローカルでは任意）。

```bash
uv sync --frozen
uv run quarto render
```

## GitHub Pages 公開

`.github/workflows/gh-pages.yml` が、`uv` ＋ `quarto render` を走らせて
`_site/` を GitHub Pages にデプロイします。

- サイト: https://watanabe3tipapa.github.io/cfos-sandbox/
- Pages 設定はソース「GitHub Actions」方式。

## ライセンス

- 本文: CC BY 4.0
- コード部分: Apache-2.0（Cloudflare OS 由来の知見を含む）