---
title: "Next.js 13でGoogleアナリティクスを導入した際のheadの表示問題と解決方法 (useSearchParams)"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "googleanalytics", "react"]
published: true
---

Next.js 13.2 で Google アナリティクスを導入した際に、Metadata が正しく動作しない問題が発生しました。具体的には、一瞬 `<html id="__next_error__">` が表示され、ボットが OGP を正しく読み取れない状態になっていました。この記事では、この問題の原因と解決方法を紹介します。

## 問題の再現

Vercel にデプロイした Google アナリティクスを導入した Next.js アプリで問題を確認しました。開発環境では問題が発生せず、`yarn start` で同様の問題が確認されました。

![<html id="__next_error__">](/images/metadata-google-analytics/error.gif)

以下のような方法で Google アナリティクスを実装していました。

```tsx:GoogleAnalytics.tsx
"use client";

import { usePathname, useSearchParams } from "next/navigation";
import Script from "next/script";
import { useEffect } from "react";
import { GA_TRACKING_ID, pageview } from "@src/lib/gtag";

export const GoogleAnalytics = () => {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  useEffect(() => {
    if (!searchParams) return;
    const url = pathname + searchParams.toString();
    pageview(url);
  }, [pathname, searchParams]);

  return (
    <>
      <Script
        strategy="afterInteractive"
        src={`https://www.googletagmanager.com/gtag/js?id=${GA_TRACKING_ID}`}
      />
      <Script
        id="gtag-init"
        strategy="afterInteractive"
        dangerouslySetInnerHTML={{
          __html: `
        window.dataLayer = window.dataLayer || [];
        function gtag(){dataLayer.push(arguments);}
        gtag('js', new Date());
        gtag('config', '${GA_TRACKING_ID}', {
          page_path: window.location.pathname,
        });
      `,
        }}
      />
    </>
  );
};
```

```tsx:layout.tsx
import "@src/styles/globals.css";
import Footer from "@src/components/Footer";
import Header from "@src/components/Header";
import type { Metadata } from "next";
import { GoogleAnalytics } from "@src/components/GoogleAnalytics";

export const metadata: Metadata = {
  title: {
    default: "テスト",
    template: "%s | テスト",
  },
  openGraph: {
    title: "テスト",
    description: "テストサイト",
    url: "https://yutakobayashi.dev",
    siteName: "テスト",
    images: [
      {
        url: "https://example.com/og.jpg",
        width: 400,
        height: 400,
      },
    ],
    locale: "ja-JP",
    type: "website",
  },
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ja">
      <head>
        <GoogleAnalytics />
      </head>
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  );
}
```

## 問題の原因

Next.js の App Dir では、ルートが静的にレンダリングされる場合、`useSearchParams()` を呼び出すと、最も近い [Suspense 境界](https://beta.nextjs.org/docs/data-fetching/streaming-and-suspense#example-using-suspense-boundaries)までのツリーがクライアントサイドレンダリングされます。

useSearchParams を使用するコンポーネントを Suspense 境界で囲むことで、クライアントサイドでレンダリングされるルートの部分を減らすことができます。

ということで、Suspense でラップするように変更することで問題なく表示されるようになりました。

```tsx
"use client";

import { usePathname, useSearchParams } from "next/navigation";
import Script from "next/script";
import { Suspense, useEffect } from "react";
import { GA_TRACKING_ID, pageview } from "@src/lib/gtag";

const GoogleAnalytics = () => {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  useEffect(() => {
    if (!searchParams) return;
    const url = pathname + searchParams.toString();
    pageview(url);
  }, [pathname, searchParams]);

  return (
    <>
      <Script
        strategy="afterInteractive"
        src={`https://www.googletagmanager.com/gtag/js?id=${GA_TRACKING_ID}`}
      />
      <Script
        id="gtag-init"
        strategy="afterInteractive"
        dangerouslySetInnerHTML={{
          __html: `
        window.dataLayer = window.dataLayer || [];
        function gtag(){dataLayer.push(arguments);}
        gtag('js', new Date());
        gtag('config', '${GA_TRACKING_ID}', {
          page_path: window.location.pathname,
        });
      `,
        }}
      />
    </>
  );
};

export default function Analytics(): JSX.Element {
  return (
    <>
      {GA_TRACKING_ID && (
        <Suspense>
          <GoogleAnalytics />
        </Suspense>
      )}
    </>
  );
}
```

## 参考

https://github.com/vercel/next.js/issues/44868#issuecomment-1436638682

https://beta.nextjs.org/docs/rendering/fundamentals#static-rendering

https://beta.nextjs.org/docs/api-reference/use-search-params#static-rendering
