# デプロイ手順(Cloudflare Workers)

`rucdev.com` を Cloudflare Workers に静的アセットとして配信する。
Cloudflare の Git 連携(Workers Builds)で `main` への push ごとに自動ビルド・デプロイする。

## 構成

| ファイル | 役割 |
|---|---|
| `astro.config.mjs` | `@astrojs/cloudflare` アダプタ。`imageService: 'compile'` でビルド時に画像を webp 化 |
| `wrangler.jsonc` | Worker 名(`rucdev-blog`)・互換日付・アセット設定 |
| `dist/client/` | ビルド成果物。`wrangler.json` と `_headers` はアダプタが自動生成 |
| `.wrangler/deploy/config.json` | `wrangler deploy` を `dist/client/wrangler.json` にリダイレクトする(自動生成、コミット不要) |

全ページ静的生成のため Worker 本体は生成されず、アセット配信のみ。
`SESSION` KV バインディングはアダプタが自動で付けるが未使用(初回デプロイ時に空の KV が作られるだけ)。

## ローカル確認

```sh
bun run build          # dist/client に出力
bun run preview        # ビルド後、wrangler dev で Workers ランタイム上で確認
bun run deploy         # ビルド後、手元から wrangler deploy(要 wrangler login)
```

## Cloudflare ダッシュボード側の設定(初回のみ)

1. Workers & Pages → Create → **Import a repository** → `Rucdev/blog_kit` を選択
2. ビルド設定
   - Build command: `bun run build`
   - Deploy command: `bunx wrangler deploy`
   - Root directory: `/`
   - Branch: `main`
3. 記事サブモジュール(`src/content/blog` → `Rucdev/tech-blog`)は公開リポジトリかつ HTTPS URL なので
   Workers Builds がそのまま clone する。private 化した場合は動かなくなるので注意
   - fetch は HTTPS だが、手元から push するには SSH が必要。clone し直した環境では一度だけ次を実行する:
     `git -C src/content/blog remote set-url --push origin git@github.com:Rucdev/tech-blog.git`
4. デプロイ後、Worker の Settings → Domains & Routes で `rucdev.com` をカスタムドメインとして追加
5. `www.rucdev.com` → `rucdev.com` のリダイレクトは Cloudflare の Bulk Redirects か
   Redirect Rules で設定(Worker 側では扱わない)

## 記事を公開するとき

記事リポジトリ側に push しただけでは公開されない。`blog_kit` 側でサブモジュールの参照を進めて push する。
必ず tech-blog を先に push すること。参照先コミットが GitHub に無いと Cloudflare 側の clone が失敗する。

```sh
git -C src/content/blog pull origin main
git add src/content/blog
git commit -m "update posts"
git push
```

## 今後の課題

- 404 ページ: `src/pages/404.astro` を作り、`wrangler.jsonc` の `assets.not_found_handling` を `"404-page"` にする
- `games.rucdev.com`(R2 + ヘッダ付与 Worker)は別 Worker として `docs/design.md` のフェーズ 5 で扱う
