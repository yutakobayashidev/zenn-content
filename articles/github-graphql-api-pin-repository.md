---
title: "Next.jsでGitHub GraphQL APIを使用してピンしたリポジトリを取得する"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "githubapi", "qraphqL"]
published: true
---

この記事では、Next.js で GitHub GraphQL API を使用してピン留めしたリポジトリを取得し、表示する方法を解説します。

![個人サイトのGitHubでピンしたリポジトリのスクリーンショット](/images/github-graphql-api-pin-repository/pin-repository.png)

## Octokit と GraphQL

Octokit は、GitHub API ライブラリです。API を直接呼び出す代わりに、Octokit を使用することでよりシンプルにアクセスできます。

まず、以下のコマンドでパッケージをインストールします。

```bash
## npm
npm install @octokit/graphql --save

## yarn
yarn add @octokit/graphql
```

GitHub API へのアクセスには、事前に[パーソナルアクセストークン](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)が必要です。`env` ファイルなどの環境変数ファイルにトークンを定義します。

```
GITHUB_TOKEN=<your-personal-access-token>
```

以下のように、トークンを含めた GraphQL リクエストを送信するためのセットアップを行います。

```ts
import { graphql } from "@octokit/graphql";

const octokit = graphql.defaults({
  headers: {
    authorization: `Token ${process.env.GITHUB_TOKEN}`,
  },
});
```

### GraphQL クエリ

次に、以下のような GraphQL クエリを使用して、ピン留めしたリポジトリを取得します。このクエリは、ユーザーの GitHub アカウントでピン留めした最大 6 つのリポジトリを取得します。login 部分は GitHub の ID に置き換えてください。

```ts
const { user } = await octokit(
  `
            {
                user(login: "your-github-id") {
                    pinnedItems(types: REPOSITORY, first: 6) {
                        nodes {
                            ... on Repository {
                                url
                                name
                                description
                                stargazerCount
                                languages(orderBy: {field: SIZE, direction: DESC}, first: 1) {
                                  nodes {
                                    name
                                    color
                                  }
                                }                      
                                repositoryTopics(first: 20) {
                                    nodes {
                                        topic {
                                            name
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        `
);
```

このクエリでは、user フィールドを使用して指定されたユーザーの情報を取得し、 pinnedItems フィールドを使用してピン留めされたリポジトリの情報を取得しています。 first 引数は、返されるリポジトリの最大数を設定します。

レスポンスは以下のようになると思います。

```json
{
  "user": {
    "pinnedItems": {
      "nodes": [
        {
          "url": "https://github.com/yutakobayashidev/dotfiles",
          "name": "dotfiles",
          "description": "💻 My dotfiles",
          "stargazerCount": 1,
          "languages": {
            "nodes": [
              {
                "name": "Shell",
                "color": "#89e051"
              }
            ]
          },
          "repositoryTopics": {
            "nodes": [
              {
                "topic": {
                  "name": "macos"
                }
              },
              {
                "topic": {
                  "name": "shell"
                }
              },
              {
                "topic": {
                  "name": "linux"
                }
              },
              {
                "topic": {
                  "name": "dotfiles"
                }
              }
            ]
          }
        }
        /// ... 省略
      ]
    }
  }
}
```

GitHub API のエクスプローラーを使用すると、必要な GraphQL クエリを簡単に作成できます。

https://docs.github.com/ja/graphql/overview/explorer

後はこのデータを加工して表示するだけです。私は Next.js の API Routes を使用することにしたので後は [SWR](https://swr.vercel.app/ja) などで作った API にアクセスして情報を表示するだけです。
