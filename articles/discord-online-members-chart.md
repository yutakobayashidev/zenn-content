---
title: "discord.pyでサーバーのオンラインユーザー数の変化を分析してみた"
emoji: "📈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "discordpy", "matplotlib"]
published: true
---

最適なオンラインイベント開催時間を見つけるために、Discord サーバーのオンラインユーザー数の変化を収集する必要がありました。

そこで、今回は Discord の API ラッパーである `discord.py` を使用して、サーバーのオンラインユーザー数を収集し、`matplotlib` を使用して可視化する一連の手順を紹介します。

## BOT の作成と招待

![Discord Developers Portal](/images/discord-online-members-chart/discord-bot-create.jpg)

:::message
既に BOT をサーバーに招待済みであれば、次の[セクション](#discord.py-のインストールと-bot-の起動)に進んでください。
:::

BOT アカウントを作成し、サーバーに招待することで、Discord サーバーでの情報収集を開始できます。BOT アカウントの作成には Discord Developer Portal を利用します。

https://discord.com/developers/applications

1. Developer Portal にアクセスし、「New Application」をクリックします。
1. アプリケーション名を入力し、「Create」をクリックします。
1. 「Create a Bot」をクリックし、BOT を作成します。
1. 「Copy」をクリックすることで、BOT のトークンをコピーできます。このトークンは後で BOT を起動する際に使用するためメモしておいてください。

次に、BOT をサーバーに招待します。

1. 「Generate Link」をクリックし、招待リンクを生成します。
1. 生成されたリンクをクリックすると、BOT を招待するサーバーを選択する画面が表示されます。
1. 利用したいサーバーを選択し、「Authorize」をクリックします。

これで、BOT アカウントの作成とサーバーへの招待が完了しました。

## discord.py のインストールと BOT の起動

`discord.py` は Discord 用の API ラッパーであり、非同期処理にも対応しています。また、様々な Discord の操作を簡単に行えるため、初心者でも簡単に Discord BOT を作成できます。

https://discordpy.readthedocs.io/ja/latest/index.html

`discord.py` は pip でインストールできます。また、今回は `asyncio` と `pandas`[^1]、 `matplotlib` も使用するので一緒にインストールしておきます。

[^1]: ` asyncio` は、Python において非同期プログラミングを行うためのライブラリです。`pandas` はデータ解析を容易にする機能を提供するデータ解析ライブラリです。

```bash
pip install discord.py asyncio pandas matplotlib
```

次に、BOT を起動するためのスクリプトを作成します。

まず、`discord.py` を使用して Discord のクライアントを起動し、`on_ready` を定義します。これはクライアントが準備完了した際に呼び出されます。

BOT の起動には、トークンが必要です。先ほどメモした BOT のトークンを使用します。

```py
import discord

intents = discord.Intents.all()
client = discord.Client(intents=intents)

@client.event
async def on_ready():
    print("BOTを起動しました")

client.run("<BOTのトークン>")
```

`"BOTのトークン"` には先程 Developer Portal で取得したトークンを貼り付けます。必要に応じて環境変数から呼び出すなどしてください。

上記のスクリプトを実行すると、BOT が起動するはずです。成功していれば、ターミナルに `BOTを起動しました` と表示されます。

## オンラインメンバー数の取得と csv への書き込み

今回は csv に `online_count` と `members_count` を 30 分に 1 回書き込みます。

```py
import asyncio
import discord
import os
import time

intents = discord.Intents.all()
client = discord.Client(intents=intents)

# DiscordのBOTトークン
bot_token = "<BOTのトークン>"

# DiscordのサーバーID
server_id = <サーバーID>

# CSVファイルのパス
csv_path = f"{server_id}.csv"

# CSVファイルが存在しない場合は新しく作成し、ヘッダーを書き込む
if not os.path.exists(csv_path):
    with open(csv_path, "w") as f:
        f.write("time,online_count,members_count\n")

@client.event
async def on_ready():
    # ボットが起動したことを表示
    print("Bot is active")

    # 無限ループ
    while True:
        # 現在の時刻を取得
        now = time.strftime("%Y-%m-%d %H:%M")

        # サーバーを取得
        server = client.get_guild(server_id)

        # サーバーが取得できなかった場合は処理を中断
        if server is None:
            continue

        # サーバーのメンバーを取得
        members = server.members

        # BOTを除くオンラインのメンバー数をカウント
        online_count = sum(1 for member in members if member.status != discord.Status.offline and not member.bot)
        # BOTを除くメンバー数をカウント
        members_count = len([member for member in members if not member.bot])

        # サーバー名とオンライン数を出力
        print(f"{now}: {server.name}: {online_count} members online")

        # CSVファイルにデータを書き込む
        with open(csv_path, "a") as f:
            f.write(f"{now},{online_count},{members_count}\n")

        # 30分待つ
        await asyncio.sleep(1800)

# ボットを起動
client.run(bot_token)
```

`members_count` を含めている理由は Discord サーバーの合計メンバー自体が変動するためグラフ化する際にmembers_countに対するonline_countの割合を計算するためです。また、BOT はカウントから除外しています。

## matplotlib で折れ線グラフを作成

` matplotlib` は、グラフを描画するためのライブラリです。2D グラフや 3D グラフ、散布図、棒グラフ、円グラフなど、様々なグラフを作成できます。

先程の csv ファイルを読み込みオンラインメンバーの割合を計算しプロットします。

```py
import matplotlib.pyplot as plt
import pandas as pd

# CSVファイルを読み込んでデータフレームを作成
df = pd.read_csv("<サーバーID>.csv")

# timeカラムを取り出す
time = pd.to_datetime(df["time"])

# online_countとmembers_countのデータを取り出す
online_count = df["online_count"]
members_count = df["members_count"]

# online_countからmembers_countの割合を計算する
ratios = online_count / members_count

# ratiosを100で割る
ratios_percent = ratios.values * 100

# 折れ線グラフを作成する
plt.plot(time, ratios_percent)

# x軸にラベルを追加する
plt.xlabel("Time")

plt.xticks(rotation=45)

plt.grid()

# y軸にラベルを追加する
plt.ylabel("Ratio of online members to total members (percent)"")

# グラフを表示する
plt.show()
```

結果は以下のようになりました。今回は 27 日の 24 時間分のデータのみをプロットしましたが、平均を求めたり、特定の期間に絞ったプロットなどを行うなど応用もできると思います。

![matplotlibで作成した27日の24時間のグラフ](/images/discord-online-members-chart/discord-online-matplotlib-chart.png)

## まとめ

以上のように、Discord サーバーのオンラインユーザー数の変化を分析できました。

普段の BOT の開発では [discord.js](https://discord.js.org/#/) をよく使用していたのですが、最近はデータ周りをよく扱うようになり Python を書くことも増えてきました。

今後は、オンラインユーザー数だけでなく、他の情報も取得し、より詳細な分析をしたいと思っています。また、Slack 版やグラフを Discord のチャンネルに自動で投稿できるようにすることも考えています。
