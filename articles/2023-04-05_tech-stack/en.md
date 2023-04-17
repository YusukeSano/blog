---
icon: "🚀"
title: "yusuke.sn Tech Stack"
excerpt: "Introducing the technology stack used in this website."
---

## Overview

This article introduces the technology stack used in this website.

## Frontend

### React

The frontend uses [React](https://react.dev) and [Next.js](https://nextjs.org) as a framework. Development is mainly done with TypeScript and CSS Modules + Sass.

#### Rendering

Next.js' ISR (Incremental Static Regeneration) builds the page in the background on the first request and returns cached data from the edge server for subsequent requests.

The top page module contains dynamic content, but this feature allows it to be cached as a static page on the CDN, resulting in a high-performance site.

### Related Libraries

- [Recoil](https://recoiljs.org): Global state management library
- [i18next](https://www.i18next.com): Internationalization framework
- [Remark](https://remark.js.org): Converts Markdown to HTML
- [SVGR](https://react-svgr.com): Converts SVG to React components
- [Framer Motion](https://www.framer.com/motion): Motion library  
  etc.

## Backend

### Hygraph

The backend uses [Hygraph](https://hygraph.com) from Headless CMS. It is adopted because it can be integrated with the frontend in a type-safe manner using [GraphQL](https://graphql.org).

### Cloudinary

I use [Cloudinary](https://cloudinary.com) as the CDN for images. The images used in the blog are optimized via CDN and displayed with fast response time.

### Related Libraries

- [graphql-request](https://github.com/jasonkuhrt/graphql-request): GraphQL client
- [graphql-codegen](https://the-guild.dev/graphql/codegen): Generates GraphQL type definitions  
  etc.

## Hosting

### Vercel

I use the hosting service [Vercel](https://vercel.com) as the deployment destination for the frontend. It is adopted due to its compatibility with Next.js and factors such as HTTPS support and edge caching of static files.

#### Dynamic OG Images

I use Vercel Edge Functions to dynamically generate OG images for the blog. Low-latency dynamic content can be delivered with a lightweight runtime without cold boot.

### Related Libraries

- [@vercel/og](https://vercel.com/docs/concepts/functions/edge-functions/og-image-generation): Dynamic OG image generation
