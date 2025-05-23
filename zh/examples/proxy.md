---
title: 代理
description: 使用 Hono 代理请求。
---

# 代理

::: tip
**更新：** 我们已经引入了新的代理辅助函数，使代理功能更加简便。查看 [代理辅助函数文档](https://hono.dev/docs/helpers/proxy) 了解更多详情。
:::

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/posts/:filename{.+.png$}', (c) => {
  const referer = c.req.header('Referer')
  if (referer && !/^https:\/\/example.com/.test(referer)) {
    return c.text('Forbidden', 403)
  }
  return fetch(c.req.url)
})

app.get('*', (c) => {
  return fetch(c.req.url)
})

export default app
```

::: tip
如果你在使用类似代码时看到 `Can't modify immutable headers.`（无法修改不可变头部）错误，你需要克隆响应对象。

```ts
app.get('/', async (_c) => {
  const response = await fetch('https://example.com')
  // 克隆响应对象以返回一个可修改头部的响应
  const newResponse = new Response(response.body, response)
  return newResponse
})
```

由 `fetch` 返回的 `Response` 的头部是不可变的。因此，如果你尝试修改它，就会发生错误。
:::
