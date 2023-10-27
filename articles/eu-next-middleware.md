---
title: "Next.js + VercelのMiddlewareでEUからのアクセスをブロックする"
emoji: "🇪🇺"
type: "tech"
topics: ["next"]
published: true
---

Next.js で GDPR などの対応で EU 圏内からのアクセスをブロックしたい事例がありました。それを Middleware で実装したのでその備忘録です。

## 1. ブロックされた時に表示されるページを作る

`src/app/geoblocked/page.tsx`などに、ブロックされたときに表示するページを作成します。

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Access blocked",
};

export default function Page() {
  return (
    <div className="mx-auto flex h-screen w-screen max-w-2xl flex-col items-center justify-center space-y-5 px-4 text-center">
      <img width="50%" height="50%" src="/eu.jpg" alt="EU Flag" />
      <h1 className="text-3xl font-bold">Not available in your country</h1>
      <p className="text-slate-500">
        Access from within the EU is not permitted.
      </p>
    </div>
  );
}
```

## 2. Middleware を作る

`src/middleware.ts`を作成し、Middleware で EU の国コードのアクセスをブロックします。

国コードのリストは[このリポジトリ](https://github.com/vercel/examples/blob/main/edge-middleware/geolocation-script/app/constants.ts)にあります。

Vercel であれば、`req.geo?.country` で、Middleware から国コードを取得することができます。その国コードが、`EU_COUNTRY_CODES`配列に含まれていたら、`/geoblocked`の内容を表示します。また、EU 圏外のアクセスで`/geoblocked`にアクセスされた場合はルートに転送するようになっています。

```tsx
import { EU_COUNTRY_CODES } from "./lib/eu";

async function middleware(req) {

    const country = req.geo?.country;
    const isBLockPage = req.nextUrl.pathname.startsWith("/geoblocked");
    const isEuCountry = country && EU_COUNTRY_CODES.includes(country);

    if (!isBLockPage && isEuCountry) {
        return NextResponse.rewrite(new URL("/geoblocked", req.url));
    } else if (isBLockPage && !isEuCountry) {
        return NextResponse.redirect(new URL("/", req.url));
    }

    return NextResponse.next();
},

export const config = { matcher: "/((?!.*\\.|api\\/).*)" };
```

## Google アナリティクスだけブロックしたい場合

もし、Google アナリティクスだけブロックしたい場合は以下の Vercel 公式の記事が参考になります。

https://vercel.com/guides/geolocation-script
