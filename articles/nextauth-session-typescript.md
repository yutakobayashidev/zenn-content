---
title: "Auth.js (Next.js)でセッション情報を追加した際のTypeScriptの型エラーについて"
emoji: "🐛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "nextauth", "typescript"]
published: true
---

Auth.js (旧 NextAuth.js)では、セッション情報として `name`,`image`,`email` をデフォルトで取得できますが、任意の情報を以下のように追加できます。

```ts:[...nextauth].ts
export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  callbacks: {
    session: ({ session, user }) => ({
      ...session,
      user: {
        ...session.user,
        id: user.id, // 追加
      },
    }),
  },
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
    }),
  ],
  secret: process.env.NEXTAUTH_SECRET,
}

export default NextAuth(authOptions)
```

しかしこうした場合、Next Auth の型が追加されていないため、型エラーが発生しました。

![Property 'id' does not exist on type '{ name?: string | null | undefined; email?: string | null | undefined; image?: string | null | undefined; }'.
](/images/nextauth-session-typescript/typescript-error.jpg)

```
Property 'id' does not exist on type '{ name?: string | null | undefined; email?: string | null | undefined; image?: string | null | undefined; }'.
```

そのため、[公式ドキュメント](https://next-auth.js.org/getting-started/typescript#module-augmentation)を参照した所、型を拡張する必要が分かりました。`types/next-auth.d.ts` に新たにファイルを作成し、以下のように指定します。

```ts:next-auth.d.ts
import NextAuth from "next-auth"

declare module "next-auth" {
  /**
   * Returned by `useSession`, `getSession` and received as a prop on the `SessionProvider` React Context
   */
  interface Session {
    user: {
      /** The user's Id. */
      id: string
    }
  }
}
```

これで型エラーが解消されました。

## 参考文献

https://github.com/nextauthjs/next-auth/discussions/2979
