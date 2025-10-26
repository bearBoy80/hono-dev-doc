# Node.js

[Node.js](https://nodejs.org/) 是一个开源、跨平台的 JavaScript 运行时环境。

Hono 最初并非为 Node.js 设计。但通过[Node.js 适配器](https://github.com/honojs/node-server)，它也可以在 Node.js 上运行。

::: info
它适用于高于 18.x 版本的 Node.js。具体所需的 Node.js 版本如下：

- 18.x => 18.14.1+
- 19.x => 19.7.0+
- 20.x => 20.0.0+

基本上，您只需使用每个主要版本的最新版本即可。
:::

## 1. 设置

有一个适用于 Node.js 的入门模板。
使用“create-hono”命令开始您的项目。
本例选择 `nodejs` 模板。

::: code-group

```sh [npm]
npm create hono@latest my-app
```

```sh [yarn]
yarn create hono my-app
```

```sh [pnpm]
pnpm create hono my-app
```

```sh [bun]
bun create hono@latest my-app
```

```sh [deno]
deno init --npm hono my-app
```

:::
进入 `my-app` 并安装依赖项。

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

## 2. Hello World

编辑 `src/index.ts`：

```ts
import { serve } from '@hono/node-server'
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('你好 Node.js！'))

serve(app)
```

如果您想优雅地关闭服务器，可以这样写：

```ts
const server = serve(app)

// 优雅关闭
process.on('SIGINT', () => {
  server.close()
  process.exit(0)
})
process.on('SIGTERM', () => {
  server.close((err) => {
    if (err) {
      console.error(err)
      process.exit(1)
    }
    process.exit(0)
  })
})
```

## 3. 运行

在本地运行开发服务器。然后，在您的 Web 浏览器中访问 `http://localhost:3000`。

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

:::

## 更改端口号

您可以使用 `port` 选项指定端口号。

```ts
serve({
  fetch: app.fetch,
  port: 8787,
})
```

## 访问原始 Node.js API

您可以从 `c.env.incoming` 和 `c.env.outgoing` 访问 Node.js API。

```ts
import { Hono } from 'hono'
import { serve, type HttpBindings } from '@hono/node-server'
// 或者如果您使用 HTTP2，则为 `Http2Bindings`

type Bindings = HttpBindings & {
  /* ... */
}

const app = new Hono<{ Bindings: Bindings }>()

app.get('/', (c) => {
  return c.json({
    remoteAddress: c.env.incoming.socket.remoteAddress,
  })
})

serve(app)
```

## 提供静态文件

您可以使用 `serveStatic` 从本地文件系统提供静态文件。例如，假设目录结构如下：

```sh
./
├── favicon.ico
├── index.ts
└── static
    ├── hello.txt
    └── image.png
```

如果收到对路径 `/static/*` 的请求，并且您想返回 `./static` 下的文件，可以这样写：

```ts
import { serveStatic } from '@hono/node-server/serve-static'

app.use('/static/*', serveStatic({ root: './' }))
```

使用 `path` 选项来提供目录根目录中的 `favicon.ico`：

```ts
app.use('/favicon.ico', serveStatic({ path: './favicon.ico' }))
```

如果收到对路径 `/hello.txt` 或 `/image.png` 的请求，并且您想返回名为 `./static/hello.txt` 或 `./static/image.png` 的文件，可以使用以下代码：

```ts
app.use('*', serveStatic({ root: './static' }))
```

### `rewriteRequestPath`

如果您想将 `http://localhost:3000/static/*` 映射到 `./statics`，可以使用 `rewriteRequestPath` 选项：

```ts
app.get(
  '/static/*',
  serveStatic({
    root: './',
    rewriteRequestPath: (path) =>
      path.replace(/^\/static/, '/statics'),
  })
)
```

## http2

您可以在 [Node.js http2 服务器](https://nodejs.org/api/http2.html)上运行 hono。

### 未加密的 http2

```ts
import { createServer } from 'node:http2'

const server = serve({
  fetch: app.fetch,
  createServer,
})
```

### 加密的 http2

```ts
import { createSecureServer } from 'node:http2'
import { readFileSync } from 'node:fs'

const server = serve({
  fetch: app.fetch,
  createServer: createSecureServer,
  serverOptions: {
    key: readFileSync('localhost-privkey.pem'),
    cert: readFileSync('localhost-cert.pem'),
  },
})
```

## 构建和部署

::: code-group

```sh [npm]
npm run build
```

```sh [yarn]
yarn run build
```

```sh [pnpm]
pnpm run build
```

```sh [bun]
bun run build
```

::: info
带有前端框架的应用程序可能需要使用 [Hono 的 Vite 插件](https://github.com/honojs/vite-plugins)。
:::

### Dockerfile

以下是 nodejs Dockerfile 的一个示例。

```Dockerfile
FROM node:22-alpine AS base

FROM base AS builder

RUN apk add --no-cache gcompat
WORKDIR /app

COPY package*json tsconfig.json src ./

RUN npm ci && \
    npm run build && \
    npm prune --production

FROM base AS runner
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 hono

COPY --from=builder --chown=hono:nodejs /app/node_modules /app/node_modules
COPY --from=builder --chown=hono:nodejs /app/dist /app/dist
COPY --from=builder --chown=hono:nodejs /app/package.json /app/package.json

USER hono
EXPOSE 3000

CMD ["node", "/app/dist/index.js"]
```
