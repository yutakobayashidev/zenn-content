---
title: "GPTs ActionsのOAuthで無茶苦茶ハマった"
emoji: "🔑"
type: "tech"
topics: ["openai", "chatgpt", "oauth"]
published: true
---

ChatGPT の新機能、GPTs では、OAuth 認証を実装でき、ユーザーごとに認証を行い独自のデータを取り出すことができます。しかし、まだドキュメントも整っておらずバグを踏みまくったのとかなりハック的な部分があるので、今後のためにまとめておきます。

## headers に Authentication 以外の独自情報を渡せない

設定画面から Authentication ヘッダーを追加することはできますが、それ以外の情報をウェブクライアント上で渡すことはできません。

今回のケースでは、Notion API を叩く予定でしたので、`Notion-Version`が必須です。

この問題に対処するため、Cloudflare Workers を使ってリバースプロキシを作成することにしました。

実際に以下の記事で手順を解説しています。

https://zenn.dev/yutakobayashi/articles/gpts-notion-api

## “Auth URL, Token URL and API hostname must share a root domain”

OAuth の構成を保存しようとした際にこのエラーが発生しました。

これは要するに、OpenAPI スキーマの servers の url と、Authorization URL、Token URL の各ルートドメインを揃える必要があるというわけです。

![<html id="__next_error__">](/images/gpts-oauth/3dfbc867fc26fc62f7d9554551eda87ff4a6299d.png)

正直とてもこれはハードですよね。

対策としては、リバースプロキシのコードを少し修正して、/redirect/{url}形式でアクセスされた場合に、元の完全な URL (パラメーター付き)に転送するようにしました。

```ts
if (url.pathname.startsWith("/redirect/")) {
  // Extract the target URL from the path
  let targetUrl = url.pathname.slice(10);

  // Preserve query parameters
  if (url.search) {
    targetUrl += url.search;
  }

  // Redirect to the specified URL
  return Response.redirect(targetUrl, 302);
}
```

---

最終的に設定する URL はこんな感じになります：

OpenAPI スキーマ：

```json
  "servers": [
    {
      "url": "https://reverse-proxy.yutakobayashi.workers.dev/https://api.notion.com"
    }
  ],
```

Client ID: 各自
Client Secret: 各自

**Authorization URL:** https://reverse-proxy.yutakobayashi.workers.dev/redirect/https://api.notion.com/v1/oauth/authorize

**Token URL**: https://reverse-proxy.yutakobayashi.workers.dev/https://api.notion.com/v1/oauth/token

**Token Exchange Method:** Basic authorization header

## GPTs を保存できない

今現在では修正された感じがしますが、GPTs がエラーを吐いて更新を保存できない時がありました。

これは Actions がある GPTs で発生しやすいようです。

どうしようもないので変更を加えたときは新規で新しく GPTs を作って元の GPTs をコピペするという絶望的な手順を踏んでいました。

## Couldn't log in with plugin.

ログイン後、callback url で OpenAI のウェブサイトに戻った際に「**Couldn't log in with plugin.」**エラーが頻繁に発生しました。

調べているとこれはどうやらこれは ChatGPT のバグのようで、言語設定を`en-US`にすると直りました。

https://community.openai.com/t/error-couldnt-log-in-with-plugin/386433

(僕はこれで一日を溶かした)

## 宣伝

ついでにこの苦労を元にできた Notion と連携できる GPTs を紹介します。

https://twitter.com/yutakobayashi__/status/1724481251783139805
