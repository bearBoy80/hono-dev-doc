# 压缩中间件

此中间件根据 `Accept-Encoding` 请求标头压缩响应正文。

::: info
**注意**：在 Cloudflare Workers 和 Deno Deploy 上，响应正文将自动压缩，因此无需使用此中间件。

**Bun**：此中间件使用 `CompressionStream`，bun 尚不支持该功能。
:::

## 导入

```ts
import { Hono } from 'hono'
import { compress } from 'hono/compress'
```

## 用法

```ts
const app = new Hono()

app.use(compress())
```

## 选项

### <Badge type="info" text="可选" /> encoding: `'gzip'` | `'deflate'`

允许响应压缩的压缩方案。可以是 `gzip` 或 `deflate`。如果未定义，则两者都允许，并将根据 `Accept-Encoding` 标头使用。如果未提供此选项且客户端在 `Accept-Encoding` 标头中同时提供了两者，则优先使用 `gzip`。

### <Badge type="info" text="可选" /> threshold: `number`

要压缩的最小大小（以字节为单位）。默认为 1024 字节。
