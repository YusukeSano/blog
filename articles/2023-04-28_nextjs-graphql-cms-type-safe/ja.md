---
icon: "🍱"
title: "Next.jsとGraphQLで型安全にCMSを導入する"
excerpt: "CMSのデータを安全かつ柔軟に取り扱う方法を紹介します。"
---

## はじめに

ウェブサイトやアプリケーションを開発する上で、コンテンツ管理システム（CMS）は必要不可欠なツールです。

CMSを利用することで、開発者はコンテンツの作成や更新を非常に容易に行うことができます。しかし、CMSを導入する際には、データの型安全性が非常に重要になってきます。

そこで本記事では、[Next.js](https://nextjs.org)と[GraphQL](https://graphql.org)を使用して、型安全にCMSを導入する方法を解説します。

Next.jsは、[React](https://react.dev)ベースのフレームワークであり、GraphQLはクライアントとサーバーの間のデータの受け渡しを行うためのクエリ言語です。

この組み合わせは、クライアントとサーバー間のデータの取り扱いを柔軟にできるため、簡単かつ効率的にCMSを導入することができます。

## Headless CMSの選定

最近では、多数のCMSサービスが展開されています。

GraphQLをサポートしたCMSサービスを使用すると、非常に簡単に導入を行うことができます。また、GraphQLのAPIサーバを作成することで、CMSサービスと連携することも可能です。

本記事では、GraphQL APIに特化したCMSサービスである[Hygraph](https://hygraph.com)（旧GraphCMS）を利用して解説を行います。

Hygraphは、本サイトでも利用しているCMSサービスであり、スキーマ設計機能や多言語化といった機能を無料で使用できます。

## Next.jsプロジェクトをセットアップする

### ソフトウェアバージョン

下記のバージョンで開発を行なっています。  
バージョンの差異により、情報が異なる可能性があります。

- next: 13.3.1
- react: 18.2.0
- react-dom: 18.2.0
- typescript: 5.0.4
- graphql: 16.6.0
- graphql-request: 5.1.0
- @graphql-codegen/cli: 3.3.1
- @graphql-codegen/client-preset: 3.0.1
- @graphql-codegen/typescript: 3.0.4
- @graphql-codegen/typescript-operations: 3.0.4
- @graphql-codegen/typescript-graphql-request: 4.5.9

### 自動セットアップ

Next.jsのアプリケーションを作成します。既に存在する場合はスキップしてください。

次のコマンドを実行して、TypeScriptのプロジェクトを作成します。

```shell
npx create-next-app@latest --typescript
```

インストールが完了したら、作成したプロジェクトに移動して開発サーバーを起動します。

```shell
npm run dev
```

`http://localhost:3000`にアクセスして、アプリケーションが表示されることを確認します。

## 環境変数

GraphQLスキーマを取得するためのエンドポイントや、認証トークン、エントリーIDなどを`.env.local`に入力して保護します。

本番環境と開発環境で値が異なる場合は`.env.development.local`と`.env.production.local`にそれぞれ設定します。

```sh
GRAPHQL_SCHEMA_API=https://xxx
GRAPHQL_AUTH_TOKEN=xxx
GRAPHQL_ENTRY_ID=xxx
```

環境変数のデフォルトの型は`string | undefined`になるので、アサーション関数を作成しておくと、安全に`undefined`を排除できます。

必要であれば`@types/env.d.ts`等の型定義ファイルを作成して、環境変数の型の上書きを行います。

```ts
/// <reference types="node" />

declare namespace NodeJS {
  interface ProcessEnv {
    readonly GRAPHQL_SCHEMA_API: string;
    readonly GRAPHQL_AUTH_TOKEN: string;
    readonly GRAPHQL_ENTRY_ID: string;
  }
}
```

## GraphQLクライアントのインストール

GraphQLクライアントには、最もシンプルで軽量な[graphql-request](https://github.com/jasonkuhrt/graphql-request)を使用します。

```shell
npm install graphql-request graphql
```

## GraphQLのクエリを定義

GraphQLのクエリを`src/gql/getPost.graphql`等に入力します。

エントリーIDなどの環境変数を用いる場合は、テンプレートリテラルのプレースホルダーのように記述します。

フィールドをフラグメントで定義しておくと、型定義がフラグメントごとに生成されるので、プロパティを受け渡しする際に便利です。

```graphql
query getPost {
  post(stage: PUBLISHED, where: { id: "${process.env['GRAPHQL_ENTRY_ID']}" }) {
    ...Post
  }
}

fragment Post on Post {
  id
  slug
  title
}
```

## コードの自動生成

[graphql-codegen](https://the-guild.dev/graphql/codegen)を使用して、コードの自動生成を行います。

```shell
npm install -D @graphql-codegen/cli @graphql-codegen/client-preset
```

TypeScriptの型定義を生成するプラグインを追加します。

```shell
npm install -D @graphql-codegen/typescript @graphql-codegen/typescript-operations
```

GraphQLクライアントのgraphql-request用プラグインを追加します。

クライアントへの依存を無くしたい場合は、[TypedDocumentNode](https://the-guild.dev/blog/typed-document-node)を使用するとインターフェースが統一されます。

```shell
npm install -D @graphql-codegen/typescript-graphql-request
```

`codegen.ts`ファイルを作成しgraphql-codegenの設定を行います。

生成後のファイルを`src/gql/index.ts`に指定することで、他ファイルへのインポートを簡潔に記述できます。

```ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  overwrite: true,
  schema: [
    {
      [process.env["GRAPHQL_SCHEMA_API"]]: {
        headers: {
          Authorization: `Bearer ${process.env["GRAPHQL_AUTH_TOKEN"]}`,
        },
      },
    },
  ],
  documents: "src/gql/**/*.graphql",
  generates: {
    "src/gql/index.ts": {
      plugins: [
        "typescript",
        "typescript-operations",
        "typescript-graphql-request",
      ],
    },
  },
};

export default config;
```

`package.json`にgraphql-codegenのスクリプトを追加します。

`dotenv_config_path`にenvファイルを指定することで環境変数を切り替えます。

```json
"scripts": {
  "codegen:prod": "graphql-codegen --require dotenv/config --config codegen.ts dotenv_config_path=.env.production.local",
  "codegen:dev": "graphql-codegen --require dotenv/config --config codegen.ts dotenv_config_path=.env.development.local"
}
```

GraphQL Code Generatorを起動します。

```shell
npm run codegen:dev
```

## 生成ファイル

`src/gql/index.ts`に自動生成されたGraphQLクエリの型定義です。

クエリの引数・戻り値の型と、フラグメントの型がそれぞれ生成されています。

```ts
export type GetPostQueryVariables = Exact<{ [key: string]: never }>;

export type GetPostQuery = {
  __typename?: "Query";
  post?: {
    __typename?: "Post";
    id: string;
    slug: string;
    title: string;
  } | null;
};

export type PostFragment = {
  __typename?: "Post";
  id: string;
  slug: string;
  title: string;
};
```

同ファイル内に自動生成されたgraphql-request用のSDKです。

このSDKにより、リクエストやレスポンスに対して自動生成された型が割り当てられます。

ここで、GraphQLクエリがテンプレートリテラル化されるため、クエリ内に記述したプレースホルダーが機能します。

```ts
import { GraphQLClient } from "graphql-request";
import * as Dom from "graphql-request/dist/types.dom";
import { gql } from "graphql-tag";

export const PostFragmentDoc = gql`
  fragment Post on Post {
    id
    slug
    title
  }
`;
export const GetPostDocument = gql`
  query getPost {
    post(stage: PUBLISHED, where: {id: "${process.env["GRAPHQL_ENTRY_ID"]}"}) {
      ...Post
    }
  }
  ${PostFragmentDoc}
`;

export type SdkFunctionWrapper = <T>(
  action: (requestHeaders?: Record<string, string>) => Promise<T>,
  operationName: string,
  operationType?: string,
) => Promise<T>;

const defaultWrapper: SdkFunctionWrapper = (
  action,
  _operationName,
  _operationType,
) => action();

export function getSdk(
  client: GraphQLClient,
  withWrapper: SdkFunctionWrapper = defaultWrapper,
) {
  return {
    getPost(
      variables?: GetPostQueryVariables,
      requestHeaders?: Dom.RequestInit["headers"],
    ): Promise<GetPostQuery> {
      return withWrapper(
        (wrappedRequestHeaders) =>
          client.request<GetPostQuery>(GetPostDocument, variables, {
            ...requestHeaders,
            ...wrappedRequestHeaders,
          }),
        "getPost",
        "query",
      );
    },
  };
}
export type Sdk = ReturnType<typeof getSdk>;
```

## コンテンツの表示

`src/libs/sdk.ts`等を作成して、GraphQLClientのインスタンスからSDKを取得します。

```ts
import { GraphQLClient } from "graphql-request";
import { getSdk } from "../gql";

const client = new GraphQLClient(process.env["GRAPHQL_SCHEMA_API"], {
  headers: {
    Authorization: `Bearer ${process.env["GRAPHQL_AUTH_TOKEN"]}`,
  },
});

export const sdk = getSdk(client);
```

SDKを他のファイルからインポートすることにより、コンテンツを表示できます。

```tsx
import { sdk } from "../libs/sdk";
import type { InferGetStaticPropsType, GetStaticProps } from "next/types";
import type { PostFragment } from "../graphql";

export default function Home({
  post,
}: InferGetStaticPropsType<typeof getStaticProps>) {
  return (
    <main>
      <h1>{post.title}</h1>
      <span>{post.slug}</span>
    </main>
  );
}

export const getStaticProps: GetStaticProps<{
  post: PostFragment;
}> = async () => {
  const response = await sdk.getPost();

  if (!response.post) {
    return {
      notFound: true,
    };
  }

  return {
    props: {
      post: response.post,
    },
  };
};
```

## おわりに

以上で、Next.jsとGraphQLを使用して、型安全にCMSを導入することができました。

是非、この記事を参考にして、Webアプリケーションの開発に取り組んでみてください。
