---
title: "Notion APIの活用事例"
emoji: "💡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["notion", "notionapi", "api"]
published: true
published_at: 2022-12-12 00:00
---

:::message
これは「[Notion Community Advent Calendar 2022](https://adventar.org/calendars/8074)」の 12 日目の記事です。昨日は[りえこ専務＠心を運ぶヒューマニティ経営](https://twitter.com/rieko0510)さんの「[Notion × 　ストレングスファインダー](https://note.com/rieko0510/n/n15a1f908bad8)」でした。
:::

## Notion API について

![Notion APIのウェブサイト](/images/notion-api-advent-calendar-22/notion-api.jpg)_developers.notion.com_

Notion API は Notion にプログラムからアクセスしたり操作するためのインターフェースです。Notion API を使うことで、Notion 上のデータを外部のアプリケーションやサービスと連携させたり、様々な作業を自動化したりできます。

本記事では私が Notion API を使って行った事例を紹介させて頂きます。

## Discord BOT

![/notion query: サイト改修](/images/notion-api-advent-calendar-22/discord-notion-search.png)

[小中学生が運営するコミュニティ](https://hurikura.com)のスタッフをしているのですが、ドキュメントやタスク管理に Notion を使用しています。しかし、チーム内でページを共有する際にいちいちリンクをコピーして共有するのが少し億劫になっていました。そこでコミュニケーションは主に Discord で行っているので Discord BOT に Notion のページを検索できるスラッシュコマンドを追加しました。タスクやプロジェクト管理のデータベースに限ってはステータスも表示されるようになっています。

大変だったのはタイトルプロパティ名が固定でないことです。以下のようなアロー関数でページタイトルを取得しています。

```js
const titleProperty = Object.entries(page.properties)
  .map(([, value]) => value)
  .find(({ type }) => {
    return type === "title";
  });
```

この BOT のソースコードは他の機能も含め Github で公開しています。

https://github.com/hurikura/discord-bot

## フォロワー数推移の自動記録

![/notion query: サイト改修](/images/notion-api-advent-calendar-22/follower-data.png)

こちらも先程のコミュニティ用に作成したものです。GAS のトリガーで毎日 1 回 API を叩いて各 SNS のフォロワー数などの情報を取得し、結果を Notion のデータベースに保存しています。

元々はスプレッドシートに書き込んでいたのですがオールインワンで管理したかったので Notion に統合しました。

こちらの実装方法などは機会があれば記事にしたいと思います。

Notion のデータベースからグラフを作成できるサービスなどもあるようなので色々と組み合わせると面白いかもしれません。

https://note.com/35d/n/n99ccabe87b12

## Apple Shortcuts

iOS や macOS にプリインストールされているショートカットアプリを使えば GET や POST など、ウェブのリクエストを行うこともできます。それを利用して Notion API を叩き様々なフローを作成しています。

### Google Books API と ISBN を利用した本の登録

https://twitter.com/yutakobayashi__/status/1540178870472970240

本の名前、または ISBN で本を検索し、Notion のページを作成するショートカットです。

### Shazam した曲の保存

![Shazamを保存しているNotionのデータベース](/images/notion-api-advent-calendar-22/shazam-notion.png)

Shazam は音楽を聴いているときに、その曲の名前やアーティストを知りたいときに使うアプリケーションです。Shazam は Apple が保有しているのでショートカットから呼び出すことができ、その結果を Notion に保存しています。これはかなり気に入っています。

---

これ以外にも日記ページに書き込んだり、TMDb API を使って映画情報を取得して登録したり、服用記録を取ったり、様々なフローをショートカットは実現できます。レシピは今後公開予定です。

## スプラトゥーンの戦歴自動保存

![Splatoon Notion](/images/notion-api-advent-calendar-22/splatoon2-notion.png)

これはかなりマニアックですが、スプラトゥーン 2 の戦歴を自動的に Notion に保存する仕組みを作りました。

このページは公開しているので誰でも閲覧でき、[ソースコード](https://github.com/yutakobayashidev/splatoon-notion)も公開しています。

https://yutakobayashi.notion.site/Splatoon2-d66a5ae5905f4fc8b14636e138c4cc87

3 バージョンも作成してある程度は仕上がっているのですがまだまだゲームの機能が出揃っておらず API の仕様変更が激しいのと、内部 API を利用している問題などもあり、スクリプトの公開はしばらく先になりそうです...🙏

https://twitter.com/yutakobayashi__/status/1568563415324696576

## 最後に

他にも Notion API を使えばブログを作ったり、普段から利用している様々なツールとの連携が可能です。興味のある方は是非以下のページから試してみてください！

https://developers.notion.com/

因みに Notion API や Shazam の説明は ChatGPT に書いてもらいました。Notion にも[Notion AI](https://www.notion.so/ja-jp/product/ai)というアシスタントが追加されるようなので記事執筆も楽になりそうです。

明日の Advent Calendar は[みきち@Notion× 採用 × インスタの人](https://www.instagram.com/mikity_s_notion/)さんの「[社内に Notion をごり押し導入していった話](https://paint-study-b02.notion.site/Notion-8321f2c5e5f349df9ae76e22153e0c83)」です。お楽しみに。
