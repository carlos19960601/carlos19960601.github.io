---
title: 'Nextjs学习'
date: 2024-09-12T09:27:24+08:00
draft: false
categories:
  - 前端
---


### 官方教程

* [Learn Next.js](https://nextjs.org/learn/dashboard-app)

### 服务端组件示例

创建一个名为 app/api/posts/route.ts 的文件，用于定义路由处理程序：

```ts
import { db } from '../../lib/db';

export async function GET(request: Request) {
  const posts = await db.post.findMany();
  return new Response(JSON.stringify(posts), { status: 200 });
}
```

在页面组件中使用 `fetch` 和 `getServerSideProps` 获取数据并展示：

```ts
// pages/posts/index.tsx
import { useEffect, useState } from 'react';

export async function getServerSideProps() {
  const response = await fetch('/api/posts');
  const data = await response.json();
  return {
    props: {
      posts: data,
    },
  };
}

export default function PostsPage({ posts }: { posts: any[] }) {
  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </div>
      ))}
    </div>
  );
}
```


### Should I use SWR or Server Actions?

No exact answer - but here is what I do:

1. For pure mutations, I prefer Server Actions.
2. For pulling data from the server-side as a result of user interactions, I use SWR.

Ultimately, you should use what you feel more comfortable with as both approaches are valid.

参考： 

* https://makerkit.dev/how-to/next-supabase/api-routes-vs-server-actions
* https://stackoverflow.com/questions/77748391/nextjs-14-server-actions-vs-route-handlers

> 个人理解：nextjs 让前端有能力连接数据库，成为全干工程师。
> 
> 当在技术选型的时候，如果项目比较小，并且只是对数据库的CRUD等操作，使用nextjs + prisma就可以实现
> 
> 如果项目较为复杂，还是只使用nextjs的前端能力，后端的事情还是单独部署
>
>   对于单纯使用nextjs的前端能力，如果需要初始化websockets，可以封装成hook 或者 在 lib 或 utils 目录下创建websockets客户端