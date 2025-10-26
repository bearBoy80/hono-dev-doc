# 代理辅助函数

代理辅助函数在使用 Hono 应用程序作为（反向）代理时提供了有用的函数。

## 导入

```ts
import { Hono } from 'hono'
import { proxy } from 'hono/proxy'
```

## `proxy()`

`proxy()` 是代理的 `fetch()` API 包装器。参数和返回值与 `fetch()` 相同（代理特定选项除外）。

`Accept-Encoding` 标头被替换为当前运行时可以处理的编码。不必要的响应标头被删除，并返回一个 `Response` 对象，您可以将其作为处理程序的响应返回。

### 示例

简单用法：

```ts
app.get('/proxy/:path', (c) => {
  return proxy(`http://${originServer}/${c.req.param('path')}`)
})
```

复杂用法：

```ts
app.get('/proxy/:path', async (c) => {
  const res = await proxy(
    `http://${originServer}/${c.req.param('path')}`,
    {
      headers: {
        ...c.req.header(), // 可选，仅在需要转发所有请求数据（包括凭据）时指定。
        'X-Forwarded-For': '127.0.0.1',
        'X-Forwarded-Host': c.req.header('host'),
        Authorization: undefined, // 不要传播 c.req.header('Authorization') 中包含的请求标头
      },
    }
  )
  res.headers.delete('Set-Cookie')
  return res
})
```

或者您可以将 `c.req` 作为参数传递。

```ts
app.all('/proxy/:path', (c) => {
  return proxy(`http://${originServer}/${c.req.param('path')}`, {
    ...c.req, // 可选，仅在需要转发所有请求数据（包括凭据）时指定。
    headers: {
      ...c.req.header(),
      'X-Forwarded-For': '127.0.0.1',
      'X-Forwarded-Host': c.req.header('host'),
      Authorization: undefined, // 不要传播 c.req.header('Authorization') 中包含的请求标头
    },
  })
})
```

您可以使用 `customFetch` 选项覆盖默认的全局 `fetch` 函数：

```ts
app.get('/proxy', (c) => {
  return proxy('https://example.com/', {
    customFetch,
  })
})
```

### 连接标头处理

默认情况下，`proxy()` 会忽略 `Connection` 标头以防止逐跳标头注入攻击。您可以使用 `strictConnectionProcessing` 选项启用严格的 RFC 9110 合规性：

```ts
// 默认行为（建议用于不受信任的客户端）
app.get('/proxy/:path', (c) => {
  return proxy(`http://${originServer}/${c.req.param('path')}`, c.req)
})

// 严格的 RFC 9110 合规性（仅在受信任的环境中使用）
app.get('/internal-proxy/:path', (c) => {
  return proxy(`http://${internalServer}/${c.req.param('path')}`, {
    ...c.req,
    strictConnectionProcessing: true,
  })
})
```

### `ProxyFetch`

`proxy()` 的类型定义为 `ProxyFetch`，如下所示

```ts
interface ProxyRequestInit extends Omit<RequestInit, 'headers'> {
  raw?: Request
  customFetch?: (request: Request) => Promise<Response>
  strictConnectionProcessing?: boolean
  headers?:
    | HeadersInit
    | [string, string][]
    | Record<RequestHeader, string | undefined>
    | Record<string, string | undefined>
}

interface ProxyFetch {
  (
    input: string | URL | Request,
    init?: ProxyRequestInit
  ): Promise<Response>
}
```
