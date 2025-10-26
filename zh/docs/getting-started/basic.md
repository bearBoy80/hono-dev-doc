# 入门

使用 Hono 非常简单。我们可以快速设置项目、编写代码、使用本地服务器进行开发并快速部署。同一套代码可以在任何运行时上运行，只需使用不同的入口点。让我们来看看 Hono 的基本用法。

## 入门模板

每个平台都有可用的入门模板。使用以下“create-hono”命令。

::: code-group

```sh [npm]
npm create hono@latest my-app
```

```sh [yarn]
yarn create hono my-app
```

```sh [pnpm]
pnpm create hono@latest my-app
```

```sh [bun]
bun create hono@latest my-app
```

```sh [deno]
deno init --npm hono@latest my-app
```

:::

然后您将被问到您想使用哪个模板。
我们以 Cloudflare Workers 为例。

```
? 您想使用哪个模板？
    aws-lambda
    bun
    cloudflare-pages
❯   cloudflare-workers
    deno
    fastly
    nextjs
    nodejs
    vercel
```

模板将被拉取到 `my-app` 中，因此请进入该目录并安装依赖项。

::: code-group

```sh [npm]
cd my-app
npm i
```

```sh [yarn]
cd my-app
yarn
```

```sh [pnpm]
cd my-app
pnpm i
```

```sh [bun]
cd my-app
bun i
```

:::

包安装完成后，运行以下命令启动本地服务器。

::: code-group

```sh [npm]
npm run dev
```

```sh [yarn]
yarn dev
```

```sh [pnpm]
pnpm dev
```

```sh [bun]
bun run dev
```

:::

## Hello World

您可以使用 Cloudflare Workers 开发工具“Wrangler”、Deno、Bun 等在 TypeScript 中编写代码，而无需担心转译。

在 `src/index.ts` 中编写您的第一个 Hono 应用程序。以下示例是一个入门 Hono 应用程序。

`import` 和最后的 `export default` 部分可能因运行时而异，
但所有应用程序代码都将在任何地方运行相同的代码。

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => {
  return c.text('你好 Hono！')
})

export default app
```

启动开发服务器并在浏览器中访问 `http://localhost:8787`。

::: code-group

```sh [npm]
npm run dev
```

```sh [yarn]
yarn dev
```

```sh [pnpm]
pnpm dev
```

```sh [bun]
bun run dev
```

:::

## 返回 JSON

返回 JSON 也很简单。以下是处理对 `/api/hello` 的 GET 请求并返回 `application/json` 响应的示例。

```ts
app.get('/api/hello', (c) => {
  return c.json({
    ok: true,
    message: '你好 Hono！',
  })
})
```

## 请求和响应

获取路径参数、URL 查询值和附加响应头的写法如下。

```ts
app.get('/posts/:id', (c) => {
  const page = c.req.query('page')
  const id = c.req.param('id')
  c.header('X-Message', '你好！')
  return c.text(`您想查看 ${id} 的第 ${page} 页`)
})
```

我们不仅可以轻松处理 GET，还可以处理 POST、PUT 和 DELETE。

```ts
app.post('/posts', (c) => c.text('已创建！', 201))
app.delete('/posts/:id', (c) =>
  c.text(`${c.req.param('id')} 已删除！`)
)
```

## 返回 HTML

您可以使用 [html 助手](/docs/helpers/html) 或使用 [JSX](/docs/guides/jsx) 语法编写 HTML。如果您想使用 JSX，请将文件名重命名为 `src/index.tsx` 并进行配置（请与每个运行时核对，因为它们不同）。以下是使用 JSX 的示例。

```tsx
const View = () => {
  return (
    <html>
      <body>
        <h1>你好 Hono！</h1>
      </body>
    </html>
  )
}

app.get('/page', (c) => {
  return c.html(<View />)
})
```

## 返回原始响应

您也可以返回原始的 [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)。

```ts
app.get('/', () => {
  return new Response('早上好！')
})
```

## 使用中间件

中间件可以为您完成繁重的工作。
例如，添加基本身份验证。

```ts
import { basicAuth } from 'hono/basic-auth'

// ...

app.use(
  '/admin/*',
  basicAuth({
    username: 'admin',
    password: 'secret',
  })
)

app.get('/admin', (c) => {
  return c.text('您已获得授权！')
})
```

有许多有用的内置中间件，包括 Bearer 和使用 JWT 的身份验证、CORS 和 ETag。
Hono 还提供了使用外部库的第三方中间件，例如 GraphQL Server 和 Firebase Auth。
而且，您还可以制作自己的中间件。

## 适配器

对于平台相关的功能，有适配器，例如，处理静态文件或 WebSocket。
例如，要在 Cloudflare Workers 中处理 WebSocket，请导入 `hono/cloudflare-workers`。

```ts
import { upgradeWebSocket } from 'hono/cloudflare-workers'

app.get(
  '/ws',
  upgradeWebSocket((c) => {
    // ...
  })
)
```

## 下一步

大多数代码都可以在任何平台上运行，但每个平台都有指南。
例如，如何设置项目或如何部署。
请查看您想用来创建应用程序的确切平台的页面！
