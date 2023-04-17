---
icon: "🚀"
title: "yusuke.sn Tech Stack"
excerpt: "このサイトの技術スタックについて紹介します。"
---

## 概要

このサイトの技術スタックについて紹介します。

## フロントエンド

### React

フロントエンドでは[React](https://react.dev)を使用し、フレームワークとして[Next.js](https://nextjs.org)を採用しています。基本的にはTypeScriptとCSS Modules + Sassで開発を行っています。

#### レンダリング

Next.jsのISR (Incremental Static Regeneration)は、ページのリクエスト時に背後でビルドを行い、以降のリクエストではエッジサーバーからキャッシュされたデータを返却します。

トップページのモジュールは動的なコンテンツを含みますが、この機能により静的ページとしてCDNにキャッシュすることが可能になるため、高パフォーマンスなサイトを実現できます。

### 関連ライブラリ

- [Recoil](https://recoiljs.org): グローバル状態管理ライブラリ
- [i18next](https://www.i18next.com): 多言語化フレームワーク
- [Remark](https://remark.js.org): MarkdownをHTMLへ変換
- [SVGR](https://react-svgr.com): SVGをReactコンポーネントへ変換
- [Framer Motion](https://www.framer.com/motion): モーションライブラリ  
  など

## バックエンド

### Hygraph

バックエンドではHeadless CMSの[Hygraph](https://hygraph.com)を使用しています。[GraphQL](https://graphql.org)を用いて型安全にフロントエンドと連携できることから採用しています。

### Cloudinary

画像のCDNとして[Cloudinary](https://cloudinary.com)を使用しています。ブログに使用する画像はCDNを介して最適化され、高速なレスポンスで表示されます。

### 関連ライブラリ

- [graphql-request](https://github.com/jasonkuhrt/graphql-request): GraphQLクライアント
- [graphql-codegen](https://the-guild.dev/graphql/codegen): GraphQLの型定義を生成  
  など

## ホスティング

### Vercel

フロントエンドのデプロイ先としてホスティングサービスの[Vercel](https://vercel.com)を使用しています。Next.jsとの相性が良く、HTTPS対応や静的ファイルのエッジキャッシュなどの要因から採用しています。

#### 動的OG画像

Vercel Edge Functionsを使用してブログのOG画像を動的に生成しています。コールドブートのない軽量なランタイムにより低レイテンシで動的コンテンツを配信できます。

### 関連ライブラリ

- [@vercel/og](https://vercel.com/docs/concepts/functions/edge-functions/og-image-generation): 動的なOG画像の生成
