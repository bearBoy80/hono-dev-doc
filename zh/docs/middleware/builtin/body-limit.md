# 正文限制中间件

正文限制中间件可以限制请求正文的文件大小。

此中间件首先使用请求中 `Content-Length` 标头的值（如果存在）。
如果未设置，它将以流的形式读取正文，如果大于指定的文件大小，则执行错误处理程序。

## 导入

```ts
import { Hono } from 'hono'
import { bodyLimit } from 'hono/body-limit'
```

## 用法

```ts
const app = new Hono()

app.post(
  '/upload',
  bodyLimit({
    maxSize: 50 * 1024, // 50kb
    onError: (c) => {
      return c.text('溢出 :(', 413)
    },
  }),
  async (c) => {
    const body = await c.req.parseBody()
    if (body['file'] instanceof File) {
      console.log(`获取到文件大小：${body['file'].size}`)
    }
    return c.text('通过 :)')
  }
)
```

## 选项

### <Badge type="danger" text="必需" /> maxSize: `number`

您要限制的文件的最大文件大小。默认值为 `100 * 1024` - `100kb`。

### <Badge type="info" text="可选" /> onError: `OnError`

如果超过指定的文件大小，将调用的错误处理程序。

## 在 Bun 中用于大请求的用法

如果显式使用正文限制中间件以允许大于默认值的请求正文，则可能需要相应地更改 `Bun.serve` 配置。[在撰写本文时](https://github.com/oven-sh/bun/blob/f2cfa15e4ef9d730fc6842ad8b79fb7ab4c71cb9/packages/bun-types/bun.d.ts#L2191)，`Bun.serve` 的默认请求正文限制为 128MiB。如果将 Hono 的正文限制中间件设置为大于该值，您的请求仍将失败，此外，中间件中指定的 `onError` 处理程序将不会被调用。这是因为 `Bun.serve()` 会将状态码设置为 `413` 并在将请求传递给 Hono 之前终止连接。

如果您想使用 Hono 和 Bun 接受大于 128MiB 的请求，则还需要为 Bun 设置限制：

```ts
export default {
  port: process.env['PORT'] || 3000,
  fetch: app.fetch,
  maxRequestBodySize: 1024 * 1024 * 200, // 此处为您的值
}
```

或者，根据您的设置：

```ts
Bun.serve({
  fetch(req, server) {
    return app.fetch(req, { ip: server.requestIP(req) })
  },
  maxRequestBodySize: 1024 * 1024 * 200, // 此处为您的值
})
```
