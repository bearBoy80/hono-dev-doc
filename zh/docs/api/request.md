# HonoRequest

`HonoRequest` 是一个可以从 `c.req` 中获取的对象，它包装了一个 [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) 对象。

## param()

获取路径参数的值。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// 捕获的参数
app.get('/entry/:id', async (c) => {
  const id = c.req.param('id')
  //    ^?
  // ...
})

// 一次性获取所有参数
app.get('/entry/:id/comment/:commentId', async (c) => {
  const { id, commentId } = c.req.param()
  //      ^?
})
```

## query()

获取查询字符串参数。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// 查询参数
app.get('/search', async (c) => {
  const query = c.req.query('q')
  //     ^?
})

// 一次性获取所有参数
app.get('/search', async (c) => {
  const { q, limit, offset } = c.req.query()
  //      ^?
})
```

## queries()

获取多个查询字符串参数值，例如 `/search?tags=A&tags=B`

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/search', async (c) => {
  // tags 将是 string[]
  const tags = c.req.queries('tags')
  //     ^?
  // ...
})
```

## header()

获取请求标头值。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  const userAgent = c.req.header('User-Agent')
  //      ^?
  return c.text(`您的用户代理是 ${userAgent}`)
})
```

::: warning
当不带参数调用 `c.req.header()` 时，返回的记录中的所有键都是**小写**的。

如果要获取名称为大写的标头的值，
请使用 `c.req.header(“X-Foo”)`。

```ts
// ❌ 不起作用
const headerRecord = c.req.header()
const foo = headerRecord['X-Foo']

// ✅ 起作用
const foo = c.req.header('X-Foo')
```

:::

## parseBody()

解析类型为 `multipart/form-data` 或 `application/x-www-form-urlencoded` 的请求正文

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.parseBody()
  // ...
})
```

`parseBody()` 支持以下行为。

**单个文件**

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody()
const data = body['foo']
//    ^?
```

`body['foo']` 是 `(string | File)`。

如果上传了多个文件，将使用最后一个。

### 多个文件

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody()
body['foo[]']
```

`body['foo[]']` 始终是 `(string | File)[]`。

需要 `[]` 后缀。

### 具有相同名称的多个文件或字段

如果您有一个允许多个 `<input type="file" multiple />` 的输入字段或具有相同名称的多个复选框 `<input type="checkbox" name="favorites" value="Hono"/>`。

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody({ all: true })
body['foo']
```

`all` 选项默认禁用。

- 如果 `body['foo']` 是多个文件，它将被解析为 `(string | File)[]`。
- 如果 `body['foo']` 是单个文件，它将被解析为 `(string | File)`。

### 点表示法

如果将 `dot` 选项设置为 `true`，则返回值将根据点表示法进行结构化。

假设收到以下数据：

```ts twoslash
const data = new FormData()
data.append('obj.key1', 'value1')
data.append('obj.key2', 'value2')
```

您可以通过将 `dot` 选项设置为 `true` 来获取结构化值：

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody({ dot: true })
// body 是 `{ obj: { key1: 'value1', key2: 'value2' } }`
```

## json()

解析类型为 `application/json` 的请求正文

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.json()
  // ...
})
```

## text()

解析类型为 `text/plain` 的请求正文

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.text()
  // ...
})
```

## arrayBuffer()

将请求正文解析为 `ArrayBuffer`

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.arrayBuffer()
  // ...
})
```

## blob()

将请求正文解析为 `Blob`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.blob()
  // ...
})
```

## formData()

将请求正文解析为 `FormData`。

```ts
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/entry', async (c) => {
  const body = await c.req.formData()
  // ...
})
```

## valid()

获取已验证的数据。

```ts
app.post('/posts', async (c) => {
  const { title, body } = c.req.valid('form')
  // ...
})
```

可用目标如下。

- `form`
- `json`
- `query`
- `header`
- `cookie`
- `param`

有关用法示例，请参阅[验证部分](/docs/guides/validation)。

## routePath

::: warning
**在 v4.8.0 中已弃用**：此属性已弃用。请改用[路由助手](/docs/helpers/route)中的 `routePath()`。
:::

您可以在处理程序中像这样检索已注册的路径：

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/posts/:id', (c) => {
  return c.json({ path: c.req.routePath })
})
```

如果您访问 `/posts/123`，它将返回 `/posts/:id`：

```json
{ "path": "/posts/:id" }
```

## matchedRoutes

::: warning
**在 v4.8.0 中已弃用**：此属性已弃用。请改用[路由助手](/docs/helpers/route)中的 `matchedRoutes()`。
:::

它返回处理程序中匹配的路由，这对于调试很有用。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.use(async function logger(c, next) {
  await next()
  c.req.matchedRoutes.forEach(({ handler, method, path }, i) => {
    const name =
      handler.name ||
      (handler.length < 2 ? '[handler]' : '[middleware]')
    console.log(
      method,
      ' ',
      path,
      ' '.repeat(Math.max(10 - path.length, 0)),
      name,
      i === c.req.routeIndex ? '<- 从此处响应' : ''
    )
  })
})
```

## path

请求路径名。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/about/me', async (c) => {
  const pathname = c.req.path // `/about/me`
  // ...
})
```

## url

请求 URL 字符串。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/about/me', async (c) => {
  const url = c.req.url // `http://localhost:8787/about/me`
  // ...
})
```

## method

请求的方法名称。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/about/me', async (c) => {
  const method = c.req.method // `GET`
  // ...
})
```

## raw

原始的 [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) 对象。

```ts
// 对于 Cloudflare Workers
app.post('/', async (c) => {
  const metadata = c.req.raw.cf?.hostMetadata?
  // ...
})
```

## cloneRawRequest()

从 HonoRequest 克隆原始 Request 对象。即使请求正文已被验证器或 HonoRequest 方法使用后也能正常工作。

```ts
import { Hono } from 'hono'
const app = new Hono()

import { cloneRawRequest } from 'hono/request'
import { validator } from 'hono/validator'

app.post(
  '/forward',
  validator('json', (data) => data),
  async (c) => {
    const clonedReq = await cloneRawRequest(c.req)
    await clonedReq.json()
    // ...
  }
)
```
