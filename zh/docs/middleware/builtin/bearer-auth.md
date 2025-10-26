# Bearer身份验证中间件

Bearer身份验证中间件通过验证请求标头中的 API 令牌来提供身份验证。
访问端点的 HTTP 客户端将在 `Authorization` 标头中添加 `Bearer {token}` 作为标头值。

从终端使用 `curl`，它看起来像这样：

```sh
curl -H 'Authorization: Bearer honoiscool' http://localhost:8787/auth/page
```

## 导入

```ts
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'
```

## 用法

> [!NOTE]
> 您的 `token` 必须匹配正则表达式 `/[A-Za-z0-9._~+/-]+=*/`，否则将返回 400 错误。值得注意的是，此正则表达式同时适用于 URL 安全的 Base64 编码和标准的 Base64 编码的 JWT。此中间件不要求不记名令牌是 JWT，只要它匹配上述正则表达式即可。

```ts
const app = new Hono()

const token = 'honoiscool'

app.use('/api/*', bearerAuth({ token }))

app.get('/api/page', (c) => {
  return c.json({ message: '您已获得授权' })
})
```

要限制到特定的路由 + 方法：

```ts
const app = new Hono()

const token = 'honoiscool'

app.get('/api/page', (c) => {
  return c.json({ message: '阅读帖子' })
})

app.post('/api/page', bearerAuth({ token }), (c) => {
  return c.json({ message: '已创建帖子！' }, 201)
})
```

要实现多个令牌（例如，任何有效令牌都可以读取，但创建/更新/删除仅限于特权令牌）：

```ts
const app = new Hono()

const readToken = 'read'
const privilegedToken = 'read+write'
const privilegedMethods = ['POST', 'PUT', 'PATCH', 'DELETE']

app.on('GET', '/api/page/*', async (c, next) => {
  // 有效令牌列表
  const bearer = bearerAuth({ token: [readToken, privilegedToken] })
  return bearer(c, next)
})
app.on(privilegedMethods, '/api/page/*', async (c, next) => {
  // 单个有效的特权令牌
  const bearer = bearerAuth({ token: privilegedToken })
  return bearer(c, next)
})

// 为 GET、POST 等定义处理程序。
```

如果您想自己验证令牌的值，请指定 `verifyToken` 选项；返回 `true` 表示接受。

```ts
const app = new Hono()

app.use(
  '/auth-verify-token/*',
  bearerAuth({
    verifyToken: async (token, c) => {
      return token === 'dynamic-token'
    },
  })
)
```

## 选项

### <Badge type="danger" text="必需" /> token: `string` | `string[]`

用于验证传入的不记名令牌的字符串。

### <Badge type="info" text="可选" /> realm: `string`

域的域名，作为返回的 WWW-Authenticate 质询标头的一部分。默认值为 `""`。
更多信息请参阅：https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate#directives

### <Badge type="info" text="可选" /> prefix: `string`

Authorization 标头值的前缀（也称为 `schema`）。默认值为 `"Bearer"`。

### <Badge type="info" text="可选" /> headerName: `string`

标头名称。默认值为 `Authorization`。

### <Badge type="info" text="可选" /> hashFunction: `Function`

用于处理哈希以安全比较身份验证令牌的函数。

### <Badge type="info" text="可选" /> verifyToken: `(token: string, c: Context) => boolean | Promise<boolean>`

用于验证令牌的函数。

### <Badge type="info" text="可选" /> noAuthenticationHeaderMessage: `string | object | MessageFunction`

`MessageFunction` 是 `(c: Context) => string | object | Promise<string | object>`。如果它没有身份验证标头，则显示自定义消息。

### <Badge type="info" text="可选" /> invalidAuthenticationHeaderMessage: `string | object | MessageFunction`

如果身份验证标头无效，则显示自定义消息。

### <Badge type="info" text="可选" /> invalidTokenMessage: `string | object | MessageFunction`

如果令牌无效，则显示自定义消息。
