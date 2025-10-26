# Deno

[Deno](https://deno.com/) 是一个基于 V8 构建的 JavaScript 运行时。它不是 Node.js。
Hono 也可以在 Deno 上运行。

您可以使用 Hono，用 TypeScript 编写代码，使用 `deno` 命令运行应用程序，并将其部署到“Deno Deploy”。

## 1. 安装 Deno

首先，安装 `deno` 命令。
请参阅[官方文档](https://docs.deno.com/runtime/getting_started/installation/)。

## 2. 设置

有一个适用于 Deno 的入门模板。
使用 [`deno init`](https://docs.deno.com/runtime/reference/cli/init/) 命令开始您的项目。

```sh
deno init --npm hono my-app --template=deno
```

进入 `my-app`。对于 Deno，您不必显式安装 Hono。

```sh
cd my-app
```

## 3. Hello World

编辑 `main.ts`：

```ts [main.ts]
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('你好 Deno！'))

Deno.serve(app.fetch)
```

## 4. 运行

在本地运行开发服务器。然后，在您的 Web 浏览器中访问 `http://localhost:8000`。

```sh
deno task start
```

## 更改端口号

您可以通过更新 `main.ts` 中 `Deno.serve` 的参数来指定端口号：

```ts
Deno.serve(app.fetch) // [!code --]
Deno.serve({ port: 8787 }, app.fetch) // [!code ++]
```

## 提供静态文件

要提供静态文件，请使用从 `hono/deno` 导入的 `serveStatic`。

```ts
import { Hono } from 'hono'
import { serveStatic } from 'hono/deno'

const app = new Hono()

app.use('/static/*', serveStatic({ root: './' }))
app.use('/favicon.ico', serveStatic({ path: './favicon.ico' }))
app.get('/', (c) => c.text('您可以访问：/static/hello.txt'))
app.get('*', serveStatic({ path: './static/fallback.txt' }))

Deno.serve(app.fetch)
```

对于上面的代码，它将与以下目录结构很好地配合使用。

```
./
├── favicon.ico
├── index.ts
└── static
    ├── demo
    │   └── index.html
    ├── fallback.txt
    ├── hello.txt
    └── images
        └── dinotocat.png
```

### `rewriteRequestPath`

如果您想将 `http://localhost:8000/static/*` 映射到 `./statics`，可以使用 `rewriteRequestPath` 选项：

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

### `mimes`

您可以使用 `mimes` 添加 MIME 类型：

```ts
app.get(
  '/static/*',
  serveStatic({
    mimes: {
      m3u8: 'application/vnd.apple.mpegurl',
      ts: 'video/mp2t',
    },
  })
)
```

### `onFound`

您可以使用 `onFound` 指定找到请求文件时的处理方式：

```ts
app.get(
  '/static/*',
  serveStatic({
    // ...
    onFound: (_path, c) => {
      c.header('Cache-Control', `public, immutable, max-age=31536000`)
    },
  })
)
```

### `onNotFound`

您可以使用 `onNotFound` 指定未找到请求文件时的处理方式：

```ts
app.get(
  '/static/*',
  serveStatic({
    onNotFound: (path, c) => {
      console.log(`${path} 未找到，您访问了 ${c.req.path}`)
    },
  })
)
```

### `precompressed`

`precompressed` 选项会检查是否存在带有 `.br` 或 `.gz` 等扩展名的文件，并根据 `Accept-Encoding` 标头提供它们。它优先使用 Brotli，然后是 Zstd，最后是 Gzip。如果都不存在，则提供原始文件。

```ts
app.get(
  '/static/*',
  serveStatic({
    precompressed: true,
  })
)
```

## Deno Deploy

Deno Deploy 是一个无服务器平台，用于在云中运行 JavaScript 和 TypeScript 应用程序。
它提供了一个管理平面，用于通过 GitHub 部署等集成来部署和运行应用程序。

Hono 也可以在 Deno Deploy 上运行。请参阅[官方文档](https://docs.deno.com/deploy/manual/)。

## 测试

在 Deno 上测试应用程序很容易。
您可以使用 `Deno.test` 编写测试，并使用 [@std/assert](https://jsr.io/@std/assert) 中的 `assert` 或 `assertEquals`。

```sh
deno add jsr:@std/assert
```

```ts [hello.ts]
import { Hono } from 'hono'
import { assertEquals } from '@std/assert'

Deno.test('Hello World', async () => {
  const app = new Hono()
  app.get('/', (c) => c.text('请测试我'))

  const res = await app.request('http://localhost/')
  assertEquals(res.status, 200)
})
```

然后运行命令：

```sh
deno test hello.ts
```

## npm 和 JSR

Hono 在 [npm](https://www.npmjs.com/package/hono) 和 [JSR](https://jsr.io/@hono/hono)（JavaScript 注册表）上都可用。您可以在 `deno.json` 中使用 `npm:hono` 或 `jsr:@hono/hono`：

```json
{
  "imports": {
    "hono": "jsr:@hono/hono" // [!code --]
    "hono": "npm:hono" // [!code ++]
  }
}
```

使用第三方中间件时，您可能需要使用与中间件来自同一注册表的 Hono，以进行正确的 TypeScript 类型推断。例如，如果使用 npm 上的中间件，则还应使用 npm 上的 Hono：

```json
{
  "imports": {
    "hono": "npm:hono",
    "zod": "npm:zod",
    "@hono/zod-validator": "npm:@hono/zod-validator"
  }
}
```

我们还在 [JSR](https://jsr.io/@hono) 上提供了许多第三方中间件包。在 JSR 上使用中间件时，请使用 JSR 上的 Hono：

```json
{
  "imports": {
    "hono": "jsr:@hono/hono",
    "zod": "npm:zod",
    "@hono/zod-validator": "jsr:@hono/zod-validator"
  }
}
```
