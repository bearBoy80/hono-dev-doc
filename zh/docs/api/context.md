# 上下文

`Context` 对象在每个请求中实例化，并一直保留到返回响应为止。您可以在其中放置值、设置要返回的标头和状态码，以及访问 HonoRequest 和 Response 对象。

## req

`req` 是 HonoRequest 的一个实例。有关更多详细信息，请参阅 [HonoRequest](/docs/api/request)。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/hello', (c) => {
  const userAgent = c.req.header('User-Agent')
  // ...
  // ---cut-start---
  return c.text(`你好，${userAgent}`)
  // ---cut-end---
})
```

## status()

您可以使用 `c.status()` 设置 HTTP 状态码。默认为 `200`。如果状态码是 `200`，则无需使用 `c.status()`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/posts', (c) => {
  // 设置 HTTP 状态码
  c.status(201)
  return c.text('您的帖子已创建！')
})
```

## header()

您可以为响应设置 HTTP 标头。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  // 设置标头
  c.header('X-Message', '我的自定义消息')
  return c.text('你好！')
})
```

## body()

返回 HTTP 响应。

::: info
**注意**：返回文本或 HTML 时，建议使用 `c.text()` 或 `c.html()`。
:::

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/welcome', (c) => {
  c.header('Content-Type', 'text/plain')
  // 返回响应正文
  return c.body('感谢光临')
})
```

您也可以这样写。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/welcome', (c) => {
  return c.body('感谢光临', 201, {
    'X-Message': '你好！',
    'Content-Type': 'text/plain',
  })
})
```

该响应与下面的 `Response` 对象相同。

```ts twoslash
new Response('感谢光临', {
  status: 201,
  headers: {
    'X-Message': '你好！',
    'Content-Type': 'text/plain',
  },
})
```

## text()

将文本呈现为 `Content-Type:text/plain`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/say', (c) => {
  return c.text('你好！')
})
```

## json()

将 JSON 呈现为 `Content-Type:application/json`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/api', (c) => {
  return c.json({ message: '你好！' })
})
```

## html()

将 HTML 呈现为 `Content-Type:text/html`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  return c.html('<h1>你好！Hono！</h1>')
})
```

## notFound()

返回一个 `Not Found` 响应。您可以使用 [`app.notFound()`](/docs/api/hono#not-found) 对其进行自定义。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/notfound', (c) => {
  return c.notFound()
})
```

## redirect()

重定向，默认状态码为 `302`。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/redirect', (c) => {
  return c.redirect('/')
})
app.get('/redirect-permanently', (c) => {
  return c.redirect('/', 301)
})
```

## res

您可以访问将要返回的 Response 对象。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Response 对象
app.use('/', async (c, next) => {
  await next()
  c.res.headers.append('X-Debug', '调试消息')
})
```

## set() / get()

获取和设置任意键值对，其生命周期为当前请求。这允许在中间件之间或从中间件到路由处理程序传递特定值。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono<{ Variables: { message: string } }>()
// ---cut---
app.use(async (c, next) => {
  c.set('message', 'Hono 很酷！！')
  await next()
})

app.get('/', (c) => {
  const message = c.get('message')
  return c.text(`消息是 "${message}"`)
})
```

将 `Variables` 作为泛型传递给 `Hono` 的构造函数以使其类型安全。

```ts twoslash
import { Hono } from 'hono'
// ---cut---
type Variables = {
  message: string
}

const app = new Hono<{ Variables: Variables }>()
```

`c.set` / `c.get` 的值仅在同一请求内保留。它们不能在不同请求之间共享或持久化。

## var

您还可以使用 `c.var` 访问变量的值。

```ts twoslash
import type { Context } from 'hono'
declare const c: Context
// ---cut---
const result = c.var.client.oneMethod()
```

如果您想创建提供自定义方法的中间件，
请像下面这样编写：

```ts twoslash
import { Hono } from 'hono'
import { createMiddleware } from 'hono/factory'
// ---cut---
type Env = {
  Variables: {
    echo: (str: string) => string
  }
}

const app = new Hono()

const echoMiddleware = createMiddleware<Env>(async (c, next) => {
  c.set('echo', (str) => str)
  await next()
})

app.get('/echo', echoMiddleware, (c) => {
  return c.text(c.var.echo('你好！'))
})
```

如果您想在多个处理程序中使用中间件，可以使用 `app.use()`。
然后，您必须将 `Env` 作为泛型传递给 `Hono` 的构造函数以使其类型安全。

```ts twoslash
import { Hono } from 'hono'
import type { MiddlewareHandler } from 'hono/types'
declare const echoMiddleware: MiddlewareHandler
type Env = {
  Variables: {
    echo: (str: string) => string
  }
}
// ---cut---
const app = new Hono<Env>()

app.use(echoMiddleware)

app.get('/echo', (c) => {
  return c.text(c.var.echo('你好！'))
})
```

## render() / setRenderer()

您可以在自定义中间件中使用 `c.setRenderer()` 设置布局。

```tsx twoslash
/** @jsx jsx */
/** @jsxImportSource hono/jsx */
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.use(async (c, next) => {
  c.setRenderer((content) => {
    return c.html(
      <html>
        <body>
          <p>{content}</p>
        </body>
      </html>
    )
  })
  await next()
})
```

然后，您可以使用 `c.render()` 在此布局中创建响应。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  return c.render('你好！')
})
```

其输出将是：

```html
<html>
  <body>
    <p>你好！</p>
  </body>
</html>
```

此外，此功能还提供了自定义参数的灵活性。
为确保类型安全，可以按如下方式定义类型：

```ts
declare module 'hono' {
  interface ContextRenderer {
    (
      content: string | Promise<string>,
      head: { title: string }
    ): Response | Promise<Response>
  }
}
```

以下是如何使用它的示例：

```ts
app.use('/pages/*', async (c, next) => {
  c.setRenderer((content, head) => {
    return c.html(
      <html>
        <head>
          <title>{head.title}</title>
        </head>
        <body>
          <header>{head.title}</header>
          <p>{content}</p>
        </body>
      </html>
    )
  })
  await next()
})

app.get('/pages/my-favorite', (c) => {
  return c.render(<p>拉面和寿司</p>, {
    title: '我的最爱',
  })
})

app.get('/pages/my-hobbies', (c) => {
  return c.render(<p>看棒球</p>, {
    title: '我的爱好',
  })
})
```

## executionCtx

您可以访问 Cloudflare Workers 特定的 [ExecutionContext](https://developers.cloudflare.com/workers/runtime-apis/context/)。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono<{
  Bindings: {
    KV: any
  }
}>()
declare const key: string
declare const data: string
// ---cut---
// ExecutionContext 对象
app.get('/foo', async (c) => {
  c.executionCtx.waitUntil(c.env.KV.put(key, data))
  // ...
})
```

## event

您可以访问 Cloudflare Workers 特定的 `FetchEvent`。这在“Service Worker”语法中使用。但现在不推荐使用。

```ts twoslash
import { Hono } from 'hono'
declare const key: string
declare const data: string
type KVNamespace = any
// ---cut---
// 类型定义以进行类型推断
type Bindings = {
  MY_KV: KVNamespace
}

const app = new Hono<{ Bindings: Bindings }>()

// FetchEvent 对象（仅在使用 Service Worker 语法时设置）
app.get('/foo', async (c) => {
  c.event.waitUntil(c.env.MY_KV.put(key, data))
  // ...
})
```

## env

在 Cloudflare Workers 中，绑定到 worker 的环境变量、机密、KV 命名空间、D1 数据库、R2 存储桶等称为绑定。
无论类型如何，绑定始终作为全局变量可用，并可通过上下文 `c.env.BINDING_KEY` 访问。

```ts twoslash
import { Hono } from 'hono'
type KVNamespace = any
// ---cut---
// 类型定义以进行类型推断
type Bindings = {
  MY_KV: KVNamespace
}

const app = new Hono<{ Bindings: Bindings }>()

// Cloudflare Workers 的环境对象
app.get('/', async (c) => {
  c.env.MY_KV.get('my-key')
  // ...
})
```

## error

如果处理程序抛出错误，则错误对象将放置在 `c.error` 中。
您可以在中间件中访问它。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.use(async (c, next) => {
  await next()
  if (c.error) {
    // 执行某些操作...
  }
})
```

## ContextVariableMap

例如，如果您希望在使用特定中间件时向变量添加类型定义，可以扩展 `ContextVariableMap`。例如：

```ts
declare module 'hono' {
  interface ContextVariableMap {
    result: string
  }
}
```

然后您可以在中间件中使用它：

```ts twoslash
import { createMiddleware } from 'hono/factory'
// ---cut---
const mw = createMiddleware(async (c, next) => {
  c.set('result', '一些值') // result 是一个字符串
  await next()
})
```

在处理程序中，变量被推断为正确的类型：

```ts twoslash
import { Hono } from 'hono'
const app = new Hono<{ Variables: { result: string } }>()
// ---cut---
app.get('/', (c) => {
  const val = c.get('result') // val 是一个字符串
  // ...
  return c.json({ result: val })
})
```
