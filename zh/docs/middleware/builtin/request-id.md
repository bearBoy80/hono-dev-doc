# 请求 ID 中间件

请求 ID 中间件为每个请求生成一个唯一的 ID，您可以在处理程序中使用它。

::: info
**Node.js**：此中间件使用 `crypto.randomUUID()` 生成 ID。全局 `crypto` 是在 Node.js 20 或更高版本中引入的。因此，在早期版本中可能会发生错误。在这种情况下，请指定 `generator`。但是，如果您使用的是[Node.js 适配器](https://github.com/honojs/node-server)，它会自动全局设置 `crypto`，因此不需要这样做。
:::

## 导入

```ts
import { Hono } from 'hono'
import { requestId } from 'hono/request-id'
```

## 用法

您可以通过已应用请求 ID 中间件的处理程序和中间件中的 `requestId` 变量访问请求 ID。

```ts
const app = new Hono()

app.use('*', requestId())

app.get('/', (c) => {
  return c.text(`您的请求 ID 是 ${c.get('requestId')}`)
})
```

如果您想显式指定类型，请导入 `RequestIdVariables` 并将其传递给 `new Hono()` 的泛型。

```ts
import type { RequestIdVariables } from 'hono/request-id'

const app = new Hono<{
  Variables: RequestIdVariables
}>()
```

### 设置请求 ID

您可以在标头（默认为 `X-Request-Id`）中设置自定义请求 ID，中间件将使用该值而不是生成新值：

```ts
const app = new Hono()

app.use('*', requestId())

app.get('/', (c) => {
  return c.text(`${c.get('requestId')}`)
})

const res = await app.request('/', {
  headers: {
    'X-Request-Id': 'your-custom-id',
  },
})
console.log(await res.text()) // your-custom-id
```

如果您想禁用此功能，请将 [`headerName` 选项](#headername-string)设置为空字符串。

## 选项

### <Badge type="info" text="可选" /> limitLength: `number`

请求 ID 的最大长度。默认值为 `255`。

### <Badge type="info" text="可选" /> headerName: `string`

用于请求 ID 的标头名称。默认值为 `X-Request-Id`。

### <Badge type="info" text="可选" /> generator: `(c: Context) => string`

请求 ID 生成函数。默认情况下，它使用 `crypto.randomUUID()`。

## 平台特定的请求 ID

某些平台（例如 AWS Lambda）已经为每个请求生成自己的请求 ID。
在没有任何其他配置的情况下，此中间件不知道这些特定的请求 ID
并生成一个新的请求 ID。这在查看您的应用程序日志时可能会导致混淆。

要统一这些 ID，请使用 `generator` 函数捕获平台特定的请求 ID 并在该中间件中使用它。

### 平台特定链接

- AWS Lambda
  - [AWS 文档：上下文对象](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-context.html)
  - [Hono：访问 AWS Lambda 对象](/docs/getting-started/aws-lambda#access-aws-lambda-object)
- Cloudflare
  - [Cloudflare Ray ID
    ](https://developers.cloudflare.com/fundamentals/reference/cloudflare-ray-id/)
- Deno
  - [Deno 博客上的请求 ID](https://deno.com/blog/zero-config-debugging-deno-opentelemetry#:~:text=s%20automatically%20have-,unique%20request%20IDs,-associated%20with%20them)
- Fastly
  - [Fastly 文档：req.xid](https://www.fastly.com/documentation/reference/vcl/variables/client-request/req-xid/)
