# cloudflare-deploy-test

Cloudflare デプロイ動作確認用。**Cloudflare Workers (Static Assets)** で静的サイトを配信し、
GitHub の `main` への push で自動デプロイされる構成。

## 構成

```
public/            ← 公開されるディレクトリ（ここの中身がそのまま配信される）
  index.html
  404.html
  styles.css
wrangler.jsonc     ← Cloudflare の設定（assets.directory = ./public）
package.json       ← wrangler を devDependency で固定
pnpm-workspace.yaml← wrangler のネイティブバイナリのビルド許可（ローカル実行用）
```

Worker スクリプト（`main`）は無し。静的ファイルのみの配信なので不要。

## 最初のデプロイが失敗した原因

```
Could not detect a directory containing static files
```

リポジトリに `README.md` しか無く、**配信する静的ファイルのディレクトリも `wrangler.jsonc` も存在しなかった**ため、
Cloudflare 側の自動検出が失敗していた。`public/` と `wrangler.jsonc` を置いて解消。

## ローカルで動かす

```sh
pnpm install
pnpm dev          # http://127.0.0.1:8787
```

確認ポイント
- `/` が 200 で返る
- `/styles.css` が `text/css` で返る
- 存在しないパスが **404 ステータス + 404.html** で返る

## デプロイ

`main` に push すると Cloudflare Workers Builds が自動でデプロイする。手動デプロイは不要。

手元から直接デプロイする場合のみ（Cloudflare へのログインが必要）:

```sh
pnpm wrangler deploy
```

## 設定のポイント

| 項目 | 値 | 理由 |
|---|---|---|
| `assets.directory` | `./public` | ここを指定しないと静的ファイルが検出されない |
| `assets.not_found_handling` | `404-page` | 未知のパスで `404.html` を404ステータスで返す |
| `assets.html_handling` | `auto-trailing-slash` | `/foo.html` と `/foo/` を自然に扱う既定値 |
| `compatibility_date` | 固定日付 | Workers の挙動を固定し、将来の変更で壊れないようにする |
| `wrangler` | devDependency で固定 | CI とローカルでバージョンを揃える |

SPA（React / Vue など）にする場合は `not_found_handling` を `single-page-application` に変える。
ビルドが必要なフレームワークを使う場合は Cloudflare 側の **Build command** に `pnpm build`、
`assets.directory` をビルド出力（`./dist` など）に変更する。

## 注意

- 無料プラン（Workers Free）で運用する。静的アセットのリクエストは無料。
- APIトークンや認証情報はこのリポジトリに置かない（`.gitignore` 済み: `.dev.vars` / `.env`）。
