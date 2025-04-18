---
title: Web API
description: 在Cloudflare Workers 和其他运行时环境中创建 Web API 的示例。
---
# Web API

这是一个在 Cloudflare Workers 和其他运行时环境中创建 Web API 的示例。

```ts
import { Hono } from 'hono'
import { cors } from 'hono/cors'
import { basicAuth } from 'hono/basic-auth'
import { prettyJSON } from 'hono/pretty-json'
import { getPosts, getPost, createPost, Post } from './model'

const app = new Hono()
app.get('/', (c) => c.text('Pretty Blog API'))
app.use(prettyJSON())
app.notFound((c) => c.json({ message: 'Not Found', ok: false }, 404))

type Bindings = {
  USERNAME: string
  PASSWORD: string
}

const api = new Hono<{ Bindings: Bindings }>()
api.use('/posts/*', cors())

api.get('/posts', (c) => {
  const { limit, offset } = c.req.query()
  const posts = getPosts({ limit, offset })
  return c.json({ posts })
})

api.get('/posts/:id', (c) => {
  const id = c.req.param('id')
  const post = getPost({ id })
  return c.json({ post })
})

api.post(
  '/posts',
  async (c, next) => {
    const auth = basicAuth({
      username: c.env.USERNAME,
      password: c.env.PASSWORD,
    })
    return auth(c, next)
  },
  async (c) => {
    const post = await c.req.json<Post>()
    const ok = createPost({ post })
    return c.json({ ok })
  }
)

app.route('/api', api)

export default app
```

## 另请参阅

- [Hono 示例 - basic](https://github.com/honojs/examples/tree/main/basic)
