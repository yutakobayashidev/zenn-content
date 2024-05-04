---
title: "Drizzle StudioでローカルのCloudflare D1データベースに繋ぐときのポイント"
emoji: "🌩️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cloudflare", "drizzleorm", "d1"]
published: true
publication_name: "hanabi_rest"
---

Cloudflare D1 は、ローカル開発時用の SQLite DB と、本番環境の DB が用意される形式になっています。

基本的には、以下の記事の通りなのですが、Drizzle Studio からローカルの DB を使用するには Wrangler の SQLite パスを手動で指定する必要などがありました。

https://zenn.dev/king/articles/3d5610429811eb

この開発を少しよくできましたのでご紹介します。

## 依存関係のインストール

kentcdodds さんが作られたコマンドラインから環境変数を指定できるライブラリをインストールします。

:::message
cross-env はメンテナンスモードにあり、深刻なバグのみ修正されます。問題は現在感じられないため使用を続けていますが、[env-cmd](https://github.com/toddbluhm/env-cmd)などの代替ライブラリの検討もお勧めします。
:::

```bash
pnpm i -D cross-env
```

インストールできたら、以下のように npm スクリプトを登録します 👇

```json
"scripts": {
    "dev": "wrangler dev src/index.ts --port 8989",
    "deploy": "wrangler deploy --minify src/index.ts",
    "generate": "drizzle-kit generate:sqlite",
    "migrate": "pnpm generate && wrangler d1 migrations apply hanabi-d1 --local",
    "db:studio": "cross-env LOCAL_DB_PATH=$(find .wrangler/state/v3/d1/miniflare-D1DatabaseObject -type f -name '*.sqlite' -print -quit) drizzle-kit studio",
    "db:studio:prod": "drizzle-kit studio"
},
```

`db:studio`コマンドでは、cross-env コマンドを使って wrangler フォルダの SQLite データベースファイル (拡張子が`.sqlite`のファイル）を find で検索し、`LOCAL_DB_PATH`環境変数に登録しているイメージです。

`db:studio:prod`コマンドでは、通常通りリモートの D1 DB に接続されます。

## drizzle の config を変える

`drizzle.config.ts`ファイルを編集します。

cross-env 経由の`LOCAL_DB_PATH`が存在する場合は`better-sqlite3` Driver を使って Drizzle Studio から接続できるようにします。

そうでない場合は、通常通り d1 Driver を使用します。

```bash
pnpm i better-sqlite3
```

```ts
import path from "path";
import "dotenv/config";
import type { Config } from "drizzle-kit";

const wranglerConfigPath = path.resolve(__dirname, "wrangler.toml");

export default process.env.LOCAL_DB_PATH
  ? ({
      schema: "./src/database.ts",
      driver: "better-sqlite",
      dbCredentials: {
        // biome-ignore lint/style/noNonNullAssertion: <explanation>
        url: process.env.LOCAL_DB_PATH!,
      },
    } satisfies Config)
  : ({
      schema: "./src/database.ts",
      out: "./drizzle",
      driver: "d1",
      dbCredentials: {
        wranglerConfigPath,
        dbName: "dev-d1",
      },
      verbose: true,
      strict: true,
    } satisfies Config);
```

これで、npm スクリプトによって接続先を切り替えられるようになりました。
