# RPC

RPC 功能允许在服务器和客户端之间共享 API 规范。

首先，从您的服务器代码中导出您的 Hono 应用的 `typeof`（通常称为 `AppType`）——或者只是您希望客户端可用的路由。

通过接受 `AppType` 作为泛型参数，Hono 客户端可以推断出验证器指定的输入类型和返回 `c.json()` 的处理程序发出的输出类型。

> [!NOTE]
> 目前，从中间件返回的响应[客户端无法推断](https://github.com/honojs/hono/issues/2719)。

> [!NOTE]
> 为了使 RPC 类型在 monorepo 中正常工作，在客户端和服务器的 tsconfig.json 文件中，在 `compilerOptions` 中设置 `"strict": true`。[阅读更多](https://github.com/honojs/hono/issues/2270#issuecomment-2143745118)。

## 服务器

在服务器端，您需要做的就是编写一个验证器并创建一个变量 `route`。以下示例使用 [Zod 验证器](https://github.com/honojs/middleware/tree/main/packages/zod-validator)。

```ts{1}
const route = app.post(
  '/posts',
  zValidator(
    'form',
    z.object({
      title: z.string(),
      body: z.string(),
    })
  ),
  (c) => {
    // ...
    return c.json(
      {
        ok: true,
        message: '已创建！',
      },
      201
    )
  }
)
```

然后，导出类型以与客户端共享 API 规范。

```ts
export type AppType = typeof route
```

## 客户端

在客户端，首先导入 `hc` 和 `AppType`。

```ts
import type { AppType } from '.'
import { hc } from 'hono/client'
```

`hc` 是一个创建客户端的函数。将 `AppType` 作为泛型传递，并指定服务器 URL 作为参数。

```ts
const client = hc<AppType>('http://localhost:8787/')
```

调用 `client.{path}.{method}` 并将您希望发送到服务器的数据作为参数传递。

```ts
const res = await client.posts.$post({
  form: {
    title: '你好',
    body: 'Hono 是一个很酷的项目',
  },
})
```

`res` 与“fetch”响应兼容。您可以使用 `res.json()` 从服务器检索数据。

```ts
if (res.ok) {
  const data = await res.json()
  console.log(data.message)
}
```

### Cookie

要使客户端在每个请求中都发送 cookie，请在创建客户端时在选项中添加 `{ 'init': { 'credentials": 'include' } }`。

```ts
// client.ts
const client = hc<AppType>('http://localhost:8787/', {
  init: {
    credentials: 'include',
  },
})

// 此请求现在将包含您可能已设置的任何 cookie
const res = await client.posts.$get({
  query: {
    id: '123',
  },
})
```

## 状态码

如果您在 `c.json()` 中明确指定状态码，例如 `200` 或 `404`。它将作为类型添加以传递给客户端。

```ts
// server.ts
const app = new Hono().get(
  '/posts',
  zValidator(
    'query',
    z.object({
      id: z.string(),
    })
  ),
  async (c) => {
    const { id } = c.req.valid('query')
    const post: Post | undefined = await getPost(id)

    if (post === undefined) {
      return c.json({ error: '未找到' }, 404) // 指定 404
    }

    return c.json({ post }, 200) // 指定 200
  }
)

export type AppType = typeof app
```

您可以通过状态码获取数据。

```ts
// client.ts
const client = hc<AppType>('http://localhost:8787/')

const res = await client.posts.$get({
  query: {
    id: '123',
  },
})

if (res.status === 404) {
  const data: { error: string } = await res.json()
  console.log(data.error)
}

if (res.ok) {
  const data: { post: Post } = await res.json()
  console.log(data.post)
}

// { post: Post } | { error: string }
type ResponseType = InferResponseType<typeof client.posts.$get>

// { post: Post }
type ResponseType200 = InferResponseType<
  typeof client.posts.$get,
  200
>
```

## 未找到

如果您想使用客户端，则不应使用 `c.notFound()` 作为“未找到”响应。客户端从服务器获取的数据无法正确推断。

```ts
// server.ts
export const routes = new Hono().get(
  '/posts',
  zValidator(
    'query',
    z.object({
      id: z.string(),
    })
  ),
  async (c) => {
    const { id } = c.req.valid('query')
    const post: Post | undefined = await getPost(id)

    if (post === undefined) {
      return c.notFound() // ❌️
    }

    return c.json({ post })
  }
)

// client.ts
import { hc } from 'hono/client'

const client = hc<typeof routes>('/')

const res = await client.posts[':id'].$get({
  param: {
    id: '123',
  },
})

const data = await res.json() // 🙁 data is unknown
```

请使用 `c.json()` 并为“未找到”响应指定状态码。

```ts
export const routes = new Hono().get(
  '/posts',
  zValidator(
    'query',
    z.object({
      id: z.string(),
    })
  ),
  async (c) => {
    const { id } = c.req.valid('query')
    const post: Post | undefined = await getPost(id)

    if (post === undefined) {
      return c.json({ error: '未找到' }, 404) // 指定 404
    }

    return c.json({ post }, 200) // 指定 200
  }
)
```

## 路径参数

您还可以处理包含路径参数的路由。

```ts
const route = app.get(
  '/posts/:id',
  zValidator(
    'query',
    z.object({
      page: z.string().optional(),
    })
  ),
  (c) => {
    // ...
    return c.json({
      title: '夜晚',
      body: '该睡觉了',
    })
  }
)
```

使用 `param` 指定要包含在路径中的字符串。

```ts
const res = await client.posts[':id'].$get({
  param: {
    id: '123',
  },
  query: {},
})
```

### 包含斜杠

`hc` 函数不会对 `param` 的值进行 URL 编码。要包含斜杠，请使用[正则表达式](/docs/api/routing#regexp)。

```ts
// client.ts

// 请求 /posts/123/456
const res = await client.posts[':id'].$get({
  param: {
    id: '123/456',
  },
})

// server.ts
const route = app.get(
  '/posts/:id{.+}',
  zValidator(
    'param',
    z.object({
      id: z.string(),
    })
  ),
  (c) => {
    // id: 123/456
    const { id } = c.req.valid('param')
    // ...
  }
)
```

> [!NOTE]
> 不带正则表达式的基本路径参数不匹配斜杠。如果您使用 hc 函数传递包含斜杠的 `param`，则服务器可能无法按预期进行路由。建议使用 `encodeURIComponent` 对参数进行编码以确保正确的路由。

## 标头

您可以将标头附加到请求中。

```ts
const res = await client.search.$get(
  {
    //...
  },
  {
    headers: {
      'X-Custom-Header': '这是 Hono 客户端',
      'X-User-Agent': 'hc',
    },
  }
)
```

要向所有请求添加通用标头，请将其指定为 `hc` 函数的参数。

```ts
const client = hc<AppType>('/api', {
  headers: {
    Authorization: 'Bearer TOKEN',
  },
})
```

## `init` 选项

您可以将 fetch 的 `RequestInit` 对象作为 `init` 选项传递给请求。以下是中止请求的示例。

```ts
import { hc } from 'hono/client'

const client = hc<AppType>('http://localhost:8787/')

const abortController = new AbortController()
const res = await client.api.posts.$post(
  {
    json: {
      // 请求正文
    },
  },
  {
    // RequestInit 对象
    init: {
      signal: abortController.signal,
    },
  }
)

// ...

abortController.abort()
```

::: info
由 `init` 定义的 `RequestInit` 对象具有最高优先级。它可用于覆盖其他选项（如 `body | method | headers`）设置的内容。
:::

## `$url()`

您可以使用 `$url()` 获取用于访问端点的 `URL` 对象。

::: warning
您必须传入一个绝对 URL 才能使其正常工作。传入相对 URL `/` 将导致以下错误。

`Uncaught TypeError: Failed to construct 'URL': Invalid URL`

```ts
// ❌ 将抛出错误
const client = hc<AppType>('/')
client.api.post.$url()

// ✅ 将按预期工作
const client = hc<AppType>('http://localhost:8787/')
client.api.post.$url()
```

:::

```ts
const route = app
  .get('/api/posts', (c) => c.json({ posts }))
  .get('/api/posts/:id', (c) => c.json({ post }))

const client = hc<typeof route>('http://localhost:8787/')

let url = client.api.posts.$url()
console.log(url.pathname) // `/api/posts`

url = client.api.posts[':id'].$url({
  param: {
    id: '123',
  },
})
console.log(url.pathname) // `/api/posts/123`
```

## 文件上传

您可以使用表单正文上传文件：

```ts
// 客户端
const res = await client.user.picture.$put({
  form: {
    file: new File([fileToUpload], filename, {
      type: fileToUpload.type,
    }),
  },
})
```

```ts
// 服务器
const route = app.put(
  '/user/picture',
  zValidator(
    'form',
    z.object({
      file: z.instanceof(File),
    })
  )
  // ...
)
```

## 自定义 `fetch` 方法

您可以设置自定义 `fetch` 方法。

在下面的 Cloudflare Worker 示例脚本中，使用了服务绑定的 `fetch` 方法，而不是默认的 `fetch`。

```toml
# wrangler.toml
services = [
  { binding = "AUTH", service = "auth-service" },
]
```

```ts
// src/client.ts
const client = hc<CreateProfileType>('http://localhost', {
  fetch: c.env.AUTH.fetch.bind(c.env.AUTH),
})
```

## 推断

使用 `InferRequestType` 和 `InferResponseType` 来了解要请求的对象的类型和要返回的对象的类型。

```ts
import type { InferRequestType, InferResponseType } from 'hono/client'

// InferRequestType
const $post = client.todo.$post
type ReqType = InferRequestType<typeof $post>['form']

// InferResponseType
type ResType = InferResponseType<typeof $post>
```

## 使用类型安全助手解析响应

您可以使用 `parseResponse()` 助手轻松地从 `hc` 解析响应并确保类型安全。

```ts
import { parseResponse, DetailedError } from 'hono/client'

// result 包含已解析的响应正文（根据 Content-Type 自动解析）
const result = await parseResponse(client.hello.$get()).catch(
  (e: DetailedError) => {
    console.error(e)
  }
)
// 如果响应不正常，parseResponse 会自动抛出错误
```

## 使用 SWR

您还可以使用 [SWR](https://swr.vercel.app) 等 React Hook 库。

```tsx
import useSWR from 'swr'
import { hc } from 'hono/client'
import type { InferRequestType } from 'hono/client'
import type { AppType } from '../functions/api/[[route]]'

const App = () => {
  const client = hc<AppType>('/api')
  const $get = client.hello.$get

  const fetcher =
    (arg: InferRequestType<typeof $get>) => async () => {
      const res = await $get(arg)
      return await res.json()
    }

  const { data, error, isLoading } = useSWR(
    'api-hello',
    fetcher({
      query: {
        name: 'SWR',
      },
    })
  )

  if (error) return <div>加载失败</div>
  if (isLoading) return <div>加载中...</div>

  return <h1>{data?.message}</h1>
}

export default App
```

## 在大型应用程序中使用 RPC

在大型应用程序的情况下，例如[构建大型应用程序](/docs/guides/best-practices#building-a-larger-application)中提到的示例，您需要注意类型推断。
一种简单的方法是链接处理程序，以便始终可以推断类型。

```ts
// authors.ts
import { Hono } from 'hono'

const app = new Hono()
  .get('/', (c) => c.json('作者列表'))
  .post('/', (c) => c.json('创建作者', 201))
  .get('/:id', (c) => c.json(`获取 ${c.req.param('id')}`))

export default app
```

```ts
// books.ts
import { Hono } from 'hono'

const app = new Hono()
  .get('/', (c) => c.json('图书列表'))
  .post('/', (c) => c.json('创建图书', 201))
  .get('/:id', (c) => c.json(`获取 ${c.req.param('id')}`))

export default app
```

然后，您可以像往常一样导入子路由器，并确保也链接它们的处理程序，因为在这种情况下，这是应用程序的顶层，这是我们要导出的类型。

```ts
// index.ts
import { Hono } from 'hono'
import authors from './authors'
import books from './books'

const app = new Hono()

const routes = app.route('/authors', authors).route('/books', books)

export default app
export type AppType = typeof routes
```

您现在可以使用注册的 AppType 创建一个新客户端，并像往常一样使用它。

## 已知问题

### IDE 性能

使用 RPC 时，路由越多，IDE 就会越慢。主要原因之一是执行了大量的类型实例化来推断应用程序的类型。

例如，假设您的应用程序有如下路由：

```ts
// app.ts
export const app = new Hono().get('foo/:id', (c) =>
  c.json({ ok: true }, 200)
)
```

Hono 将如下推断类型：

```ts
export const app = Hono<BlankEnv, BlankSchema, '/'>().get<
  'foo/:id',
  'foo/:id',
  JSONRespondReturn<{ ok: boolean }, 200>,
  BlankInput,
  BlankEnv
>('foo/:id', (c) => c.json({ ok: true }, 200))
```

这是单个路由的类型实例化。虽然用户不需要手动编写这些类型参数，这是一件好事，但众所周知，类型实例化需要花费大量时间。IDE 中使用的 `tsserver` 每次使用该应用程序时都会执行这项耗时的任务。如果您有很多路由，这会显著降低 IDE 的速度。

但是，我们有一些技巧可以缓解这个问题。

#### Hono 版本不匹配

如果您的后端与前端分开，并且位于不同的目录中，则需要确保 Hono 版本匹配。如果您在后端使用一个 Hono 版本，而在前端使用另一个版本，则会遇到诸如“_类型实例化过深且可能无限_”之类的问题。

![](https://github.com/user-attachments/assets/e4393c80-29dd-408d-93ab-d55c11ccca05)

#### TypeScript 项目引用

与[Hono 版本不匹配](#hono-version-mismatch)的情况一样，如果您的后端和前端是分开的，您也会遇到问题。如果您想在前端访问后端的代码（例如 `AppType`），则需要使用[项目引用](https://www.typescriptlang.org/docs/handbook/project-references.html)。TypeScript 的项目引用允许一个 TypeScript 代码库访问和使用另一个 TypeScript 代码库中的代码。_（来源：[Hono RPC 和 TypeScript 项目引用](https://catalins.tech/hono-rpc-in-monorepos/)）_。

#### 在使用前编译您的代码（推荐）

`tsc` 可以在编译时执行繁重的任务，例如类型实例化！然后，`tsserver` 无需在每次使用时都实例化所有类型参数。这将使您的 IDE 速度更快！

编译您的客户端（包括服务器应用程序）可为您提供最佳性能。将以下代码放入您的项目中：

```ts
import { app } from './app'
import { hc } from 'hono/client'

// 这是一个在编译时计算类型的技巧
export type Client = ReturnType<typeof hc<typeof app>>

export const hcWithType = (...args: Parameters<typeof hc>): Client =>
  hc<typeof app>(...args)
```

编译后，您可以使用 `hcWithType` 代替 `hc` 来获取已计算类型的客户端。

```ts
const client = hcWithType('http://localhost:8787/')
const res = await client.posts.$post({
  form: {
    title: '你好',
    body: 'Hono 是一个很酷的项目',
  },
})
```

如果您的项目是 monorepo，则此解决方案非常适合。使用 [`turborepo`](https://turbo.build/repo/docs) 之类的工具，您可以轻松地分离服务器项目和客户端项目，并获得更好的集成来管理它们之间的依赖关系。这是一个[有效的示例](https://github.com/m-shaka/hono-rpc-perf-tips-example)。

您还可以使用 `concurrently` 或 `npm-run-all` 之类的工具手动协调构建过程。

#### 手动指定类型参数

这有点麻烦，但您可以手动指定类型参数以避免类型实例化。

```ts
const app = new Hono().get<'foo/:id'>('foo/:id', (c) =>
  c.json({ ok: true }, 200)
)
```

仅指定单个类型参数会在性能上产生差异，但如果您有很多路由，则可能需要花费大量时间和精力。

#### 将您的应用程序和客户端拆分为多个文件

如[在大型应用程序中使用 RPC](#using-rpc-with-larger-applications)中所述，您可以将您的应用程序拆分为多个应用程序。您还可以为每个应用程序创建一个客户端：

```ts
// authors-cli.ts
import { app as authorsApp } from './authors'
import { hc } from 'hono/client'

const authorsClient = hc<typeof authorsApp>('/authors')

// books-cli.ts
import { app as booksApp } from './books'
import { hc } from 'hono/client'

const booksClient = hc<typeof booksApp>('/books')
```

这样，`tsserver` 无需一次性为所有路由实例化类型。
