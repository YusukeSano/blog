---
icon: "🍱"
title: "Introducing Type-Safe CMS with Next.js and GraphQL"
excerpt: "Learn how to handle CMS data safely and flexibly."
---

**🌐 This article was translated by [DeepL API](https://www.deepl.com/pro-api).**

## Introduction

A content management system (CMS) is an essential tool for developing websites and applications.

By using a CMS, developers can easily create and update content. However, when introducing a CMS, data type safety becomes extremely important.

In this article, I will explain how to introduce a CMS with type safety using [Next.js](https://nextjs.org) and [GraphQL](https://graphql.org).

Next.js is a framework based on [React](https://react.dev), and GraphQL is a query language for passing data between client and server.

This combination allows for flexible handling of data between client and server, making it easy and efficient to introduce a CMS.

## Selecting Headless CMS

There are now many CMS services available.

Using a CMS service that supports GraphQL is very easy to introduce. It is also possible to integrate with CMS services by creating a GraphQL API server.

In this article, I will use [Hygraph](https://hygraph.com) (formerly GraphCMS), a CMS service specialized in GraphQL API, for explanation.

Hygraph is a CMS service that is also used on this site, and features such as schema design and internationalization are available free of charge.

## Setting Up Next.js Project

### Software Versions

I am developing using the following versions.  
Information may differ depending on version.

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

### Automatic Setup

Create a Next.js application. If it already exists, please skip this step.

Run the following command to create a TypeScript project.

```shell
npx create-next-app@latest --typescript
```

After installation is complete, move to the created project and start the development server.

```shell
npm run dev
```

Visit `http://localhost:3000` to view the application.

## Environment Variables

Enter the endpoint for obtaining the GraphQL schema, authentication token, entry ID, etc. in `.env.local` to protect them.

If the values differ between production and development environments, set them in `.env.development.local` and `.env.production.local`, respectively.

```sh
GRAPHQL_SCHEMA_API=https://xxx
GRAPHQL_AUTH_TOKEN=xxx
GRAPHQL_ENTRY_ID=xxx
```

Since the default type of environment variables will be `string | undefined`, you can safely eliminate `undefined` by creating an assertion function.

If necessary, create a type definition file such as `@types/env.d.ts` to override the type of environment variables.

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

## Installing GraphQL Client

Use [graphql-request](https://github.com/jasonkuhrt/graphql-request), the simplest and lightest GraphQL client.

```shell
npm install graphql-request graphql
```

## Defining GraphQL Queries

Enter a GraphQL query in a file such as `src/gql/getPost.graphql`.

If you are using environment variables like an entry ID, you can write them as placeholders in template literals.

Defining fields in fragments is useful when passing properties because type definitions are generated for each fragment.

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

## Automatic Code Generation

Use [graphql-codegen](https://the-guild.dev/graphql/codegen) to generate code automatically.

```shell
npm install -D @graphql-codegen/cli @graphql-codegen/client-preset
```

Add a plugin to generate TypeScript type definitions.

```shell
npm install -D @graphql-codegen/typescript @graphql-codegen/typescript-operations
```

Add a plugin for the graphql-request client.

If you want to remove dependencies from the client, you can use [TypedDocumentNode](https://the-guild.dev/blog/typed-document-node) to unify interfaces.

```shell
npm install -D @graphql-codegen/typescript-graphql-request
```

Create a `codegen.ts` file to configure graphql-codegen.

Specify the generated files in `src/gql/index.ts` to simplify importing them into other files.

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

Add the script for graphql-codegen to `package.json`.

Specify an env file in `dotenv_config_path` to switch environment variables.

```json
"scripts": {
  "codegen:prod": "graphql-codegen --require dotenv/config --config codegen.ts dotenv_config_path=.env.production.local",
  "codegen:dev": "graphql-codegen --require dotenv/config --config codegen.ts dotenv_config_path=.env.development.local"
}
```

Start GraphQL Code Generator.

```shell
npm run codegen:dev
```

## Generated File

The automatically generated GraphQL query type definitions in `src/gql/index.ts`.

The types for query arguments, return values, and fragments are all generated.

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

SDK for graphql-request automatically generated in the same file.

This SDK assigns automatically generated types to requests and responses.

Here, the GraphQL query is made into a template literals, so the placeholders described in the query will work.

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

## Displaying Contents

Create a file such as `src/libs/sdk.ts` to get the SDK from GraphQLClient instance.

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

Contents can be displayed by importing the SDK from other files.

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

## Conclusion

That's it, you have successfully introduced a type-safe CMS using Next.js and GraphQL.

I encourage you to use this article as a reference and try your hand at developing web applications.
