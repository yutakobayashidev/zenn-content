---
title: "Notion APIの活用事例"
emoji: "💡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["notion","notionapi","api"]
published: true
published_at: 2022-12-12 00:00
---

:::message
これは「[Notion Community Advent Calendar 2022](https://adventar.org/calendars/8074)」の12日目の記事です。昨日は[りえこ専務＠心を運ぶヒューマニティ経営](https://twitter.com/rieko0510)さんの「[Notion ×　ストレングスファインダー](https://note.com/rieko0510/n/n15a1f908bad8)」でした。
:::

## Notion APIについて

![Notion APIのウェブサイト](/images/notion-api-advent-calendar-22/notion-api.jpg)*developers.notion.com*

Notion APIはNotionにプログラムからアクセスしたり操作するためのインターフェースです。Notion APIを使うことで、Notion上のデータを外部のアプリケーションやサービスと連携させたり、様々な作業を自動化したりすることができます。


本記事では私がNotion APIを使って行った事例を紹介させて頂きます。

## Discord BOT

![/notion query: サイト改修](/images/notion-api-advent-calendar-22/discord-notion-search.png)

[小中学生が運営するコミュニティ](https://hurikura.com)のスタッフをしているのですが、ドキュメントやタスク管理にNotionを使用しています。しかし、チーム内でページを共有する際にいちいちリンクをコピーして共有するのが少し億劫になっていました。そこでコミュニケーションはDiscordで行っているのでDiscord BOTにNotionのページを検索できるスラッシュコマンドを作成しました。タスクやプロジェクト管理のデータベースに限ってはステータスも表示されるようになっています。

大変だったのはタイトルプロパティ名が固定でないことです。以下のようなアロー関数でページタイトルを取得しています。

```js
const titleProperty = Object.entries(page.properties).map(([, value]) => value).find(({ type }) => {
    return type === 'title';
})
```

このBOTのソースコードは他の機能も含めGithubで公開しています。

https://github.com/hurikura/discord-bot

## フォロワー数推移の自動記録

![/notion query: サイト改修](/images/notion-api-advent-calendar-22/follower-data.png)


こちらも先程のコミュニティ用に作成したものです。GASのトリガーで毎日1回APIを叩いて各SNSのフォロワー数などの情報を取得し、結果をNotionのデータベースに保存しています。

元々はスプレッドシートに書き込んでいたのですがオールインワンにしたかったのでNotionに統合しました。

こちらの実装方法などは機会があれば記事にしたいと思います。

Notionのデータベースからグラフを作成できるサービスなどもあるようなので色々と組み合わせると面白いかもしれません。

https://note.com/35d/n/n99ccabe87b12


## Apple Shortcuts

iPhoneやMacにプリインストールされているショートカットアプリを使えばGETやPOSTなど、ウェブのリクエストを行うこともできます。それを利用してNotion APIを叩き様々なフローを作成しています。

### Google Books APIとISBNを利用した本の登録

本の名前、またはISBNで本を検索し、Notionのページを作成するショートカットです。これ以外にも日記ページに書き込んだり、TMDb APIを使って映画情報を取得して登録したり、服用記録を取ったり、様々な場面でショートカットが使えます。レシピは今後公開予定です。

https://twitter.com/yutakobayashi__/status/1540178870472970240

## スプラトゥーンの戦歴自動保存

![Splatoon Notion](/images/notion-api-advent-calendar-22/splatoon2-notion.png)

これはかなりマニアックですが、スプラトゥーン2の戦歴を自動的にNotionに保存する仕組みを作りました。

このページは公開しているので誰でも閲覧でき、[ソースコード](https://github.com/yutakobayashidev/splatoon-notion)も公開しています。

https://yutakobayashi.notion.site/Splatoon2-d66a5ae5905f4fc8b14636e138c4cc87


3バージョンも作成してある程度は仕上がっているのですがまだまだゲームの機能が出揃っておらずAPIの仕様変更が激しいのと、内部APIを利用している問題などもあり、スクリプトの公開はしばらく先になりそうです...🙏

https://twitter.com/yutakobayashi__/status/1568563415324696576

## 最後に

他にもNotion APIを使えばブログを作ったり、普段から利用している様々なツールとの連携が可能です。興味のある方は是非以下のページから試してみてください！

https://developers.notion.com/


明日のAdvent Calendarは**みきち@Notion×採用×インスタの人**
さんです。お楽しみに。