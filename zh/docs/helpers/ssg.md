# SSG 辅助函数

SSG 辅助函数从您的 Hono 应用程序生成静态站点。它将检索已注册路由的内容并将其保存为静态文件。

## 用法

### 手动

如果您有一个简单的 Hono 应用程序，如下所示：

```tsx
// index.tsx
const app = new Hono()

app.get('/', (c) => c.html('你好，世界！'))
app.use('/about', async (c, next) => {
  c.setRenderer((content, head) => {
    return c.html(
      <html>
        <head>
          <title>{head.title ?? ''}</title>
        </head>
        <body>
          <p>{content}</p>
        </body>
      </html>
    )
  })
  await next()
})
app.get('/about', (c) => {
  return c.render('你好！', { title: 'Hono SSG 页面' })
})

export default app
```

对于 Node.js，创建一个如下所示的构建脚本：

```ts
// build.ts
import app from './index'
import { toSSG } from 'hono/ssg'
import fs from 'fs/promises'

toSSG(app, fs)
```

通过执行该脚本，文件将输出如下：

```bash
ls ./static
about.html  index.html
```

### Vite 插件

使用 `@hono/vite-ssg` Vite 插件，您可以轻松处理该过程。

有关更多详细信息，请参阅此处：

https://github.com/honojs/vite-plugins/tree/main/packages/ssg

## toSSG

`toSSG` 是用于生成静态站点的主要函数，它以应用程序和文件系统模块作为参数。它基于以下内容：

### 输入

toSSG 的参数在 ToSSGInterface 中指定。

```ts
export interface ToSSGInterface {
  (
    app: Hono,
    fsModule: FileSystemModule,
    options?: ToSSGOptions
  ): Promise<ToSSGResult>
}
```

- `app` 指定已注册路由的 `new Hono()`。
- `fs` 指定以下对象，假设为 `node:fs/promise`。

```ts
export interface FileSystemModule {
  writeFile(path: string, data: string | Uint8Array): Promise<void>
  mkdir(
    path: string,
    options: { recursive: boolean }
  ): Promise<void | string>
}
```

### 为 Deno 和 Bun 使用适配器

如果您想在 Deno 或 Bun 上使用 SSG，则为每个文件系统提供了一个 `toSSG` 函数。

对于 Deno：

```ts
import { toSSG } from 'hono/deno'

toSSG(app) // 第二个参数是类型为 `ToSSGOptions` 的选项。
```

对于 Bun：

```ts
import { toSSG } from 'hono/bun'

toSSG(app) // 第二个参数是类型为 `ToSSGOptions` 的选项。
```

### 选项

选项在 ToSSGOptions 接口中指定。

```ts
export interface ToSSGOptions {
  dir?: string
  concurrency?: number
  extensionMap?: Record<string, string>
  plugins?: SSGPlugin[]
}
```

- `dir` 是静态文件的输出目标。默认值为 `./static`。
- `concurrency` 是同时生成的文件数。默认值为 `2`。
- `extensionMap` 是一个映射，其中 `Content-Type` 作为键，扩展名的字符串作为值。这用于确定输出文件的文件扩展名。
- `plugins` 是一个 SSG 插件数组，用于扩展静态站点生成过程的功能。

### 输出

`toSSG` 以以下 Result 类型返回结果。

```ts
export interface ToSSGResult {
  success: boolean
  files: string[]
  error?: Error
}
```

## 生成文件

### 路由和文件名

以下规则适用于已注册的路由信息和生成的文件名。默认的 `./static` 行为如下：

- `/` -> `./static/index.html`
- `/path` -> `./static/path.html`
- `/path/` -> `./static/path/index.html`

### 文件扩展名

文件扩展名取决于每个路由返回的 `Content-Type`。例如，来自 `c.html` 的响应将保存为 `.html`。

如果您想自定义文件扩展名，请设置 `extensionMap` 选项。

```ts
import { toSSG, defaultExtensionMap } from 'hono/ssg'

// 将 `application/x-html` 内容保存为 `.html`
toSSG(app, fs, {
  extensionMap: {
    'application/x-html': 'html',
    ...defaultExtensionMap,
  },
})
```

请注意，以斜杠结尾的路径将保存为 index.ext，而不管扩展名如何。

```ts
// 保存到 ./static/html/index.html
app.get('/html/', (c) => c.html('html'))

// 保存到 ./static/text/index.txt
app.get('/text/', (c) => c.text('text'))
```

## 中间件

介绍支持 SSG 的内置中间件。

### ssgParams

您可以使用类似 Next.js 的 `generateStaticParams` 的 API。

示例：

```ts
app.get(
  '/shops/:id',
  ssgParams(async () => {
    const shops = await getShops()
    return shops.map((shop) => ({ id: shop.id }))
  }),
  async (c) => {
    const shop = await getShop(c.req.param('id'))
    if (!shop) {
      return c.notFound()
    }
    return c.render(
      <div>
        <h1>{shop.name}</h1>
      </div>
    )
  }
)
```

### disableSSG

设置了 `disableSSG` 中间件的路由将被 `toSSG` 从静态文件生成中排除。

```ts
app.get('/api', disableSSG(), (c) => c.text('an-api'))
```

### onlySSG

设置了 `onlySSG` 中间件的路由将在 `toSSG` 执行后被 `c.notFound()` 覆盖。

```ts
app.get('/static-page', onlySSG(), (c) => c.html(<h1>欢迎访问我的网站</h1>))
```

## 插件

插件允许您扩展静态站点生成过程的功能。它们使用钩子在不同阶段自定义生成过程。

### 默认插件

默认情况下，`toSSG` 使用 `defaultPlugin`，它会跳过非 200 状态的响应（例如重定向、错误或 404）。这可以防止为不成功的响应生成文件。

```ts
import { toSSG, defaultPlugin } from 'hono/ssg'

// 未指定插件时自动应用 defaultPlugin
toSSG(app, fs)

// 等效于：
toSSG(app, fs, { plugins: [defaultPlugin] })
```

如果指定了自定义插件，则**不会**自动包含 `defaultPlugin`。要在添加自定义插件的同时保持默认行为，请明确包含它：

```ts
toSSG(app, fs, {
  plugins: [defaultPlugin, myCustomPlugin],
})
```

### 钩子类型

插件可以使用以下钩子来自定义 `toSSG` 过程：

```ts
export type BeforeRequestHook = (req: Request) => Request | false
export type AfterResponseHook = (res: Response) => Response | false
export type AfterGenerateHook = (
  result: ToSSGResult
) => void | Promise<void>
```

- **BeforeRequestHook**：在处理每个请求之前调用。返回 `false` 以跳过该路由。
- **AfterResponseHook**：在收到每个响应后调用。返回 `false` 以跳过文件生成。
- **AfterGenerateHook**：在整个生成过程完成后调用。

### 插件接口

```ts
export interface SSGPlugin {
  beforeRequestHook?: BeforeRequestHook | BeforeRequestHook[]
  afterResponseHook?: AfterResponseHook | AfterResponseHook[]
  afterGenerateHook?: AfterGenerateHook | AfterGenerateHook[]
}
```

### 基本插件示例

仅过滤 GET 请求：

```ts
const getOnlyPlugin: SSGPlugin = {
  beforeRequestHook: (req) => {
    if (req.method === 'GET') {
      return req
    }
    return false
  },
}
```

按状态码过滤：

```ts
const statusFilterPlugin: SSGPlugin = {
  afterResponseHook: (res) => {
    if (res.status === 200 || res.status === 500) {
      return res
    }
    return false
  },
}
```

记录生成的文件：

```ts
const logFilesPlugin: SSGPlugin = {
  afterGenerateHook: (result) => {
    if (result.files) {
      result.files.forEach((file) => console.log(file))
    }
  },
}
```

### 高级插件示例

这是一个创建站点地图插件的示例，该插件会生成一个 `sitemap.xml` 文件：

```ts
// plugins.ts
import fs from 'node:fs/promises'
import path from 'node:path'
import type { SSGPlugin } from 'hono/ssg'
import { DEFAULT_OUTPUT_DIR } from 'hono/ssg'

export const sitemapPlugin = (baseURL: string): SSGPlugin => {
  return {
    afterGenerateHook: (result, fsModule, options) => {
      const outputDir = options?.dir ?? DEFAULT_OUTPUT_DIR
      const filePath = path.join(outputDir, 'sitemap.xml')
      const urls = result.files.map((file) =>
        new URL(file, baseURL).toString()
      )
      const siteMapText = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${urls.map((url) => `<url><loc>${url}</loc></url>`).join('\n')}
</urlset>`
      fsModule.writeFile(filePath, siteMapText)
    },
  }
}
```

应用插件：

```ts
import app from './index'
import { toSSG } from 'hono/ssg'
import { sitemapPlugin } from './plugins'

toSSG(app, fs, {
  plugins: [
    getOnlyPlugin,
    statusFilterPlugin,
    logFilesPlugin,
    sitemapPlugin('https://example.com'),
  ],
})
```
