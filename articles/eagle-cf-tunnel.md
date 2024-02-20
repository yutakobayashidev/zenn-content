---
title: "別の端末からEagleに画像を送信して快適にライブラリを更新する (Cloudflare Tunnel) "
emoji: "🖼️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cloudflare", "eagle", "ショートカット"]
published: true
---

## Eagle が便利

Eagle は、スクリーンショットや画像、動画、音声、フォントなどをローカル環境で管理できる便利なアプリです。クラウドサービスとは異なり、自分で完全にコントロールできます。しかし、モバイルアプリがないため、別の端末から Eagle ライブラリを更新する方法がありません。

https://eagle.cool

私はよくモバイルアプリの UI をスクリーンショットを撮って保存しているので、効率よく Eagle に保存する方法がないか考えていました。

この記事では、Cloudflare Tunnel を使用して、外部の端末から Eagle に安全にファイルを送信する方法を紹介します。

### Eagle の API

Eagle のデスクトップアプリは `http://localhost:41595` で API を提供しており、この API を介してライブラリの中身を参照したり、ファイルをアップロードすることができますが、デフォルトではローカルネットワーク内でのみアクセス可能です。

https://api.eagle.cool

## Cloudflare Tunnel を利用した解決策

Cloudflare Tunnel を使用することで、Eagle の API サーバーを公開し、どこからでもアクセスできるようになります。これにより、iPhone のショートカットアプリなどを利用して、外出先から簡単にファイルを Eagle にアップロードできるようになります。

また、Cloudflare Workers と組み合わせることで、Basic 認証などを追加することもできます。

このようなケースでは ngrok を使うことが多いかと思いますが、最近は Cloudflare スタックで環境を統一しているのと、独自ドメインを無料で設定したかったので今回は使用しませんでした。

### 1. Cloudflare での設定

1. ネームサーバーの変更: 独自ドメインを使用する場合、ドメインのネームサーバーを Cloudflare に設定します。

2. Cloudflared のインストール: Cloudflare と通信するためのツール cloudflared をインストールします。

```bash
$ brew install cloudflared
```

3. トンネルの作成: Cloudflare にログインし、ドメインを選択してトンネルを作成します。

```bash
$ cloudflared tunnel login
$ cloudflared tunnel create eagle
```

### 2. DNS の設定

トンネルを作成した後、独自ドメインでトンネルにアクセスできるように DNS を設定します。

```bash
$ cloudflared tunnel route dns <uuid> eagle.example.com
```

### 3. 設定ファイルの作成

Eagle の API に接続するための設定ファイルを`~/.cloudflared/config.yaml`に作成します。

```yaml
tunnel: <uuid>
credentials-file: /Users/<ユーザー名>/.cloudflared/<uuid>.json

ingress:
  - hostname: eagle.example.com
    service: http://localhost:41595
  - service: http_status:404
```

### 4. 接続チェック

これで設定が完了しました。試しに起動してみましょう。

```bash
$ cloudflared tunnel run eagle
```

設定した URL に正しくアクセスできていれば成功です！

```
https://eagle.example.com/api/item/info
```

### 5. ベーシック認証をかける

このままでは、URL が分かればライブラリの中身を見たりファイルをあなたの端末の Eagle にアップロードできてしまいます。そこで Cloudflare Workers を使ってベーシック認証をかけます。

以下の記事が参考になります。Workers ルートはトンネルに設定するドメインにしてください。(今回の場合 eagle.example.com)

https://blog.tesoro-crea.biz/259/

### 6. Apple ショートカット

あとは好きなところからドキュメント通りに API を叩くだけです。

iOS や iPadOS、MacOS にはビルトインでショートカットと呼ばれるアプリが入っています。

操作などを自動化するためのアプリで、HTTP リクエストや base64 エンコードなどの高度なタスクも自動化できます。

![Eagle](/images/eagle-cf-tunnel/shortcuts.jpg)

これを使って Eagle に画像をアップロードすることができます。共有シートから呼びせばすぐに保存できます。便利ですね。

以下からショートカットをダウンロードできます ↓
https://www.icloud.com/shortcuts/e754acf863db4a678febcd680ba29298

---

以上のように、比較的簡単に Eagle の API を公開して画像を外部からアップロードする機構が完成しました。もしよければ試してみてください！
