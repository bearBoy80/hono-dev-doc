# JWK 身份验证中间件

JWK 身份验证中间件通过使用 JWK（JSON Web Key）验证令牌来对请求进行身份验证。它会检查 `Authorization` 标头以及其他已配置的来源（例如 cookie，如果已指定）。具体来说，它使用提供的 `keys` 验证令牌，如果指定了 `jwks_uri`，则从该 URI 检索密钥，如果设置了 `cookie` 选项，则支持从 cookie 中提取令牌。

:::info
客户端发送的 Authorization 标头必须具有指定的方案。

示例：`Bearer my.token.value` 或 `Basic my.token.value`
:::

## 导入

```ts
import { Hono } from 'hono'
import { jwk } from 'hono/jwk'
import { verifyWithJwks } from 'hono/jwt'
```

## 用法

```ts
const app = new Hono()

app.use(
  '/auth/*',
  jwk({
    jwks_uri: `https://${backendServer}/.well-known/jwks.json`,
  })
)

app.get('/auth/page', (c) => {
  return c.text('您已获得授权')
})
```

获取有效负载：

```ts
const app = new Hono()

app.use(
  '/auth/*',
  jwk({
    jwks_uri: `https://${backendServer}/.well-known/jwks.json`,
  })
)

app.get('/auth/page', (c) => {
  const payload = c.get('jwtPayload')
  return c.json(payload) // 例如：{ "sub": "1234567890", "name": "John Doe", "iat": 1516239022 }
})
```

匿名访问：

```ts
const app = new Hono()

app.use(
  '/auth/*',
  jwk({
    jwks_uri: (c) =>
      `https://${c.env.authServer}/.well-known/jwks.json`,
    allow_anon: true,
  })
)

app.get('/auth/page', (c) => {
  const payload = c.get('jwtPayload')
  return c.json(payload ?? { message: '你好匿名者' })
})
```

## 在中间件之外使用 `verifyWithJwks`

`verifyWithJwks` 实用函数可用于在 Hono 的中间件上下文之外验证 JWT 令牌，例如在 SvelteKit SSR 页面或其他服务器端环境中：

```ts
const id_payload = await verifyWithJwks(
  id_token,
  {
    jwks_uri: 'https://your-auth-server/.well-known/jwks.json',
  },
  {
    cf: { cacheEverything: true, cacheTtl: 3600 },
  }
)
```

## 选项

### <Badge type="info" text="可选" /> keys: `HonoJsonWebKey[] | (c: Context) => Promise<HonoJsonWebKey[]>`

您的公钥的值，或返回它们的函数。该函数接收 Context 对象。

### <Badge type="info" text="可选" /> jwks_uri: `string` | `(c: Context) => Promise<string>`

如果设置了此值，则尝试从此 URI 获取 JWK，期望得到一个带有 `keys` 的 JSON 响应，这些 `keys` 会被添加到提供的 `keys` 选项中。您还可以传递一个回调函数，以使用 Context 动态确定 JWKS URI。

### <Badge type="info" text="可选" /> allow_anon: `boolean`

如果此值设置为 `true`，则没有有效令牌的请求将被允许通过中间件。使用 `c.get('jwtPayload')` 检查请求是否已通过身份验证。默认值为 `false`。

### <Badge type="info" text="可选" /> cookie: `string`

如果设置了此值，则使用该值作为键从 cookie 标头中检索值，然后将其作为令牌进行验证。

### <Badge type="info" text="可选" /> headerName: `string`

用于查找 JWT 令牌的标头名称。默认值为 `Authorization`。
