---
title: "GPTsでNotion APIを叩くようにしてみたらやばかった"
emoji: "🔨"
type: "tech"
topics: ["openai", "chatgpt", "notion", "notionapi"]
published: true
---

OpenAI の DevDay で発表された、GPTs は、特定のタスクに特化したカスタムモデルを作成できる ChatGPT Plus で利用できる新しい機能です。作った GPTs は、自分だけで使うのはもちろん、友達にシェアしたり。ウェブ上で公開することもできます。

https://www.youtube.com/watch?v=U9mJuUkhUzk

この GPTs の機能である Actions を使うと、OpenAPI Schema を元に、外部 API を ChatGPT エージェントが実行するようになります。

この機能を使って Notion などの様々なサービスと GPTs を繋げてみたので、そのデモと GPTs のつくりかたを解説します。

## デモ

Notion は、[API](https://developers.notion.com/) と呼ばれる開発者が Notion のデータを操作し、外部アプリケーションやサービスと連携するための機能が公開されています。そこで、Notion API の OpenAPI スキーマを書いて検索、データベースクエリ、ブロックの読み取りエンドポイントを覚えさせ、実際に GPTs に叩かせてみました。

ユーザーのプロンプトを元にエンドポイントを使い分けてくれてるのが分かると思います。プロンプト次第では一回のクエリにとどまらず複数回のクエリで答えを探そうとし出します。

少し専門的なプロンプトですが、システムプロンプトでチューニングすればもう少し簡略化できると思います。

![japan-population](/images/gpts-notion-api/demo.png)

各サービスのデータを GPT を噛ませるだけでここまでの結果が返ってくるのでかなり可能性を感じました。Notion 以外にもデジタル庁の法令 API や Discord のメッセージ送信なども試しましたが無事動作しました 🎉

---

**追記：**

OAuth を使ってノーコードで使用できる GPTs を新しく作りました。

ChatGPT のウェブクライアントからログインするだけで Notion ワークスペースを検索することができます。

https://twitter.com/yutakobayashi__/status/1724481251783139805

:::message
ログイン後に「Couldn't log in with plugin.」というエラーが出た場合は、言語設定を `en-US` にすると直る事が多いです。

https://community.openai.com/t/error-couldnt-log-in-with-plugin/386433
:::

## Internal API を使う場合のやりかた

### 1. API キーの発行とコネクトの追加

[notion.so/my-integrations](https://www.notion.so/my-integrations)から、Notion の Internal API キーを発行します。

この API キーは控えておいてください。

作成したら、Notion のページやデータベースページの右上の...から先程作成したインテグレーションをコネクトに追加します。

### 2. リバースプロキシの設定

Notion API を叩く際には、headers に、`Notion-Version`をセットし、API バージョンを渡してあげる必要があります。

しかし、ChatGPT の Web UI 上では Authentication ヘッダーしか追加できません。

そこで、Cloudflare Workers を使ってリバースプロキシを作ることにしました。

```
https://reverse-proxy.yutakobayashi.workers.dev/<url>
```

ただ`api.notion.com`にアクセスされたときだけヘッダーを追加するだけのプロキシですが、不安であればリポジトリにコードがあるので自分でホストするのがいいと思います。

https://github.com/yutakobayashidev/workers/blob/main/workers/reverse-proxy/src/index.ts

### 3. GPTs の設定

GPTs の設定画面の Configure から、下から二番目の Actions を選択します。

OpenAPI のスキーマとして、例として今回は以下のような記載をしました。ちなみにこのスキーマ自体も大半 GPT に作ってもらいました。

(長いのでアコーディオンの中)

:::details OpenAPI スキーマ

````json
{
  "openapi": "3.1.0",
  "info": {
    "title": "Notion API",
    "version": "v1.0.0"
  },
  "servers": [
    {
      "url": "https://reverse-proxy.yutakobayashi.workers.dev/https://api.notion.com"
    }
  ],
  "paths": {
    "/v1/blocks/{block_id}/children": {
      "get": {
        "description": "Retrieve children of a block",
        "operationId": "getBlockChildren",
        "tags": [
          "Blocks"
        ],
        "parameters": [
          {
            "name": "block_id",
            "in": "path",
            "required": true,
            "description": "ID of the block or page ID to get the child process",
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "start_cursor",
            "in": "query",
            "description": "A cursor to paginate through the block's children",
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "page_size",
            "in": "query",
            "description": "The number of items from the block's children to return per page",
            "schema": {
              "type": "integer"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "object": {
                      "type": "string",
                      "enum": [
                        "list"
                      ]
                    },
                    "results": {
                      "type": "array",
                      "items": {
                        "type": "object"
                      }
                    },
                    "next_cursor": {
                      "type": "string"
                    },
                    "has_more": {
                      "type": "boolean"
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad request"
          },
          "401": {
            "description": "Unauthorized"
          },
          "404": {
            "description": "Not Found"
          },
          "429": {
            "description": "Too Many Requests"
          },
          "500": {
            "description": "Internal Server Error"
          }
        }
      }
    },
    "/v1/search": {
      "post": {
        "description": "Search across all pages and databases",
        "operationId": "search",
        "tags": [
          "Search"
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "query": {
                    "type": "string",
                    "description": "Text to search for"
                  },
                  "sort": {
                    "type": "object",
                    "properties": {
                      "direction": {
                        "type": "string",
                        "enum": [
                          "ascending",
                          "descending"
                        ]
                      },
                      "timestamp": {
                        "type": "string"
                      }
                    }
                  },
                  "filter": {
                    "type": "object",
                    "properties": {
                      "property": {
                        "type": "string"
                      },
                      "value": {
                        "type": "object"
                      }
                    }
                  },
                  "start_cursor": {
                    "type": "string"
                  },
                  "page_size": {
                    "type": "integer"
                  }
                },
                "required": [
                  "query"
                ]
              }
            }
          }
        },
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "results": {
                      "type": "array",
                      "items": {
                        "type": "object"
                      }
                    },
                    "next_cursor": {
                      "type": "string"
                    },
                    "has_more": {
                      "type": "boolean"
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad request"
          },
          "401": {
            "description": "Unauthorized"
          },
          "500": {
            "description": "Internal Server Error"
          }
        }
      }
    },
    "/v1/databases/{database_id}/query": {
      "post": {
        "description": "Query a database",
        "operationId": "queryDatabase",
        "tags": [
          "Databases"
        ],
        "parameters": [
          {
            "name": "database_id",
            "in": "path",
            "required": true,
            "description": "ID of the database to query",
            "schema": {
              "type": "string"
            }
          }
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "filter": {
                    "type": "object",
                    "description": "Filter conditions for the query"
                  },
                  "sorts": {
                    "type": "array",
                    "items": {
                      "type": "object"
                    }
                  },
                  "start_cursor": {
                    "type": "string"
                  },
                  "page_size": {
                    "type": "integer"
                  }
                }
              }
            }
          }
        },
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "object": {
                      "type": "string",
                      "enum": [
                        "list"
                      ]
                    },
                    "results": {
                      "type": "array",
                      "items": {
                        "type": "object"
                      }
                    },
                    "next_cursor": {
                      "type": "string"
                    },
                    "has_more": {
                      "type": "boolean"
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad request"
          },
          "401": {
            "description": "Unauthorized"
          },
          "404": {
            "description": "Not Found"
          },
          "429": {
            "description": "Too Many Requests"
          },
          "500": {
            "description": "Internal Server Error"
          }
        }
      }
    }
  },
  "components": {
    "schemas": {},
    "securitySchemes": {
      "apiKey": {
        "type": "apiKey",
        "in": "header",
        "name": "Authorization"
      }
    }
  }
}
```
:::

また、Authentication を選択し、API Key、Bearer として控えた API キーを登録します。

あとはシステムメッセージなどを雑に登録するだけです 🎉

Instructions の例:
```

この GPTs は、Notion データベースを自然言語クエリで検索するために設計されています。膨大な Notion ページから関連する情報を効率的に検索します。英語と日本語の両方のクエリを理解できるようにプログラムされており、多様なユーザがアクセスできるようになっています。

GPT は、Notion データベースから正確で文脈に沿った情報を提供することに重点を置いており、迅速で信頼性の高いデータを求める従業員や関係者にとって、非常に貴重なツールとなっています。GPT は、データのプライバシーとセキュリティの制約の中で運用され、機密情報が細心の注意を払って取り扱われることを保証します。また、簡潔で有益な回答を提供し、必要に応じてクエリを明確にして検索結果の精度を高めることができます。

検索 API とページのコンテンツ読み取りをうまく組み合わせてください。

また、特にユーザーの希望がない場合、以下のような形態でレスポンスしてください。

## ページタイトル

- 作成日
- 最終更新日
- 各プロパティ情報の要約
- Noiton の URL

{AI が考える概要}

````
