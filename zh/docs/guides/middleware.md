# 中间件

中间件在处理程序之后/之前工作。我们可以在分派之前获取 `Request` 或在分派之后操作 `Response`。

## 中间件的定义

- 处理程序 - 应返回 `Response` 对象。只有一个处理程序会被调用。
- 中间件 - 应不返回任何内容，将通过 `await next()` 继续执行下一个中间件。

用户可以使用 `app.use` 或 `app.HTTP_METHOD` 以及处理程序来注册中间件。此功能可以轻松指定路径和方法。

```ts
// 匹配任何方法，所有路由
app.use(logger())

// 指定路径
app.use('/posts/*', cors())

// 指定方法和路径
app.post('/posts/*', basicAuth())
```

如果处理程序返回 `Response`，它将用于最终用户，并停止处理。

```ts
app.post('/posts', (c) => c.text('已创建！', 201))
```

在这种情况下，四个中间件在分派之前按以下顺序处理：

```ts
logger() -> cors() -> basicAuth() -> *处理程序*
```

## 执行顺序

中间件的执行顺序由其注册顺序决定。
第一个注册的中间件的 `next` 之前的过程最先执行，
而 `next` 之后的过程最后执行。
请看下面。

```ts
app.use(async (_, next) => {
  console.log('中间件 1 开始')
  await next()
  console.log('中间件 1 结束')
})
app.use(async (_, next) => {
  console.log('中间件 2 开始')
  await next()
  console.log('中间件 2 结束')
})
app.use(async (_, next) => {
  console.log('中间件 3 开始')
  await next()
  console.log('中间件 3 结束')
})

app.get('/', (c) => {
  console.log('处理程序')
  return c.text('你好！')
})
```

结果如下。

```
中间件 1 开始
  中间件 2 开始
    中间件 3 开始
      处理程序
    中间件 3 结束
  中间件 2 结束
中间件 1 结束
```

请注意，如果处理程序或任何中间件抛出异常，hono 将捕获它，并将其传递给[您的 app.onError() 回调](/docs/api/hono#error-handling)或自动将其转换为 500 响应，然后再将其返回到中间件链。这意味着 next() 永远不会抛出异常，因此无需将其包装在 try/catch/finally 中。

## 内置中间件

Hono 有内置中间件。

```ts
import { Hono } from 'hono'
import { poweredBy } from 'hono/powered-by'
import { logger } from 'hono/logger'
import { basicAuth } from 'hono/basic-auth'

const app = new Hono()

app.use(poweredBy())
app.use(logger())

app.use(
  '/auth/*',
  basicAuth({
    username: 'hono',
    password: 'acoolproject',
  })
)
```

::: warning
在 Deno 中，可以使用与 Hono 版本不同的中间件版本，但这可能会导致错误。
例如，此代码不起作用，因为版本不同。

```ts
import { Hono } from 'jsr:@hono/hono@4.4.0'
import { upgradeWebSocket } from 'jsr:@hono/hono@4.4.5/deno'

const app = new Hono()

app.get(
  '/ws',
  upgradeWebSocket(() => ({
    // ...
  }))
)
```

:::

## 自定义中间件

您可以直接在 `app.use()` 中编写自己的中间件：

```ts
// 自定义记录器
app.use(async (c, next) => {
  console.log(`[${c.req.method}] ${c.req.url}`)
  await next()
})

// 添加自定义标头
app.use('/message/*', async (c, next) => {
  await next()
  c.header('x-message', '这是中间件！')
})

app.get('/message/hello', (c) => c.text('你好中间件！'))
```

但是，直接在 `app.use()` 中嵌入中间件会限制其可重用性。因此，我们可以将我们的
中间件分离到不同的文件中。

为确保在分离中间件时我们不会丢失 `context` 和 `next` 的类型定义，我们可以使用
Hono 工厂中的 [`createMiddleware()`](/docs/helpers/factory#createmiddleware)。这也使我们能够以类型安全的方式从下游处理程序[访问我们在 `Context` 中 `set` 的数据](https://hono.dev/docs/api/context#set-get)。

```ts
import { createMiddleware } from 'hono/factory'

const logger = createMiddleware(async (c, next) => {
  console.log(`[${c.req.method}] ${c.req.url}`)
  await next()
})
```

:::info
类型泛型可与 `createMiddleware` 一起使用：

```ts
createMiddleware<{Bindings: Bindings}>(async (c, next) =>
```

:::

### 在 Next 之后修改响应

此外，中间件还可以设计为在必要时修改响应：

```ts
const stripRes = createMiddleware(async (c, next) => {
  await next()
  c.res = undefined
  c.res = new Response('新响应')
})
```

## 中间件参数中的上下文访问

要在中间件参数中访问上下文，请直接使用 `app.use` 提供的上下文参数。有关说明，请参见下面的示例。

```ts
import { cors } from 'hono/cors'

app.use('*', async (c, next) => {
  const middleware = cors({
    origin: c.env.CORS_ORIGIN,
  })
  return middleware(c, next)
})
```

### 在中间件中扩展上下文

要在中间件中扩展上下文，请使用 `c.set`。您可以通过将 `{ Variables: { yourVariable: YourVariableType } }` 泛型参数传递给 `createMiddleware` 函数来使其类型安全。

```ts
import { createMiddleware } from 'hono/factory'

const echoMiddleware = createMiddleware<{
  Variables: {
    echo: (str: string) => string
  }
}>(async (c, next) => {
  c.set('echo', (str) => str)
  await next()
})

app.get('/echo', echoMiddleware, (c) => {
  return c.text(c.var.echo('你好！'))
})
```

## 第三方中间件

内置中间件不依赖于外部模块，但第三方中间件可以依赖于第三方库。
因此，我们可以使用它们来制作更复杂的应用程序。

我们可以探索各种[第三方中间件](https://hono.dev/docs/middleware/third-party)。
例如，我们有 GraphQL 服务器中间件、Sentry 中间件、Firebase Auth 中间件等。
