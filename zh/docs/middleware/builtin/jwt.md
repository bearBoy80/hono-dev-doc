# JWT 身份验证中间件

JWT 身份验证中间件通过使用 JWT 验证令牌来提供身份验证。
如果未设置 `cookie` 选项，则中间件将检查 `Authorization` 标头。您可以使用 `headerName` 选项自定义标头名称。

:::info
客户端发送的 Authorization 标头必须具有指定的方案。

示例：`Bearer my.token.value` 或 `Basic my.token.value`
:::

## 导入

```ts
import { Hono } from 'hono'
import { jwt } from 'hono/jwt'
import type { JwtVariables } from 'hono/jwt'
```

## 用法

```ts
// 指定变量类型以推断 `c.get('jwtPayload')`：
type Variables = JwtVariables

const app = new Hono<{ Variables: Variables }>()

app.use(
  '/auth/*',
  jwt({
    secret: 'it-is-very-secret',
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
  jwt({
    secret: 'it-is-very-secret',
    issuer: 'my-trusted-issuer',
  })
)

app.get('/auth/page', (c) => {
  const payload = c.get('jwtPayload')
  return c.json(payload) // 例如：{ "sub": "1234567890", "name": "John Doe", "iat": 1516239022, "iss": "my-trusted-issuer" }
})
```

::: tip

`jwt()` 只是一个中间件函数。如果您想使用环境变量（例如 `c.env.JWT_SECRET`），可以按如下方式使用它：

```js
app.use('/auth/*', (c, next) => {
  const jwtMiddleware = jwt({
    secret: c.env.JWT_SECRET,
  })
  return jwtMiddleware(c, next)
})
```

:::

## 选项

### <Badge type="danger" text="必需" /> secret: `string`

您的密钥的值。

### <Badge type="info" text="可选" /> cookie: `string`

如果设置了此值，则使用该值作为键从 cookie 标头中检索值，然后将其作为令牌进行验证。

### <Badge type="info" text="可选" /> alg: `string`

用于验证的算法类型。默认值为 `HS256`。

可用类型为 `HS256` | `HS384` | `HS512` | `RS256` | `RS384` | `RS512` | `PS256` | `PS384` | `PS512` | `ES256` | `ES384` | `ES512` | `EdDSA`。

### <Badge type="info" text="可选" /> headerName: `string`

用于查找 JWT 令牌的标头名称。默认值为 `Authorization`。

```ts
app.use(
  '/auth/*',
  jwt({
    secret: 'it-is-very-secret',
    headerName: 'x-custom-auth-header',
  })
)
```

### <Badge type="info" text="可选" /> verifyOptions: `VerifyOptions`

控制令牌验证的选项。

#### <Badge type="info" text="可选" /> verifyOptions.iss: `string | RexExp`

用于令牌验证的预期颁发者。如果未设置此项，则**不会**检查 `iss` 声明。

#### <Badge type="info" text="可选" /> verifyOptions.nbf: `boolean`

如果存在且此项设置为 `true`，则将验证 `nbf`（不早于）声明。默认值为 `true`。

#### <Badge type="info" text="可选" /> verifyOptions.iat: `boolean`

如果存在且此项设置为 `true`，则将验证 `iat`（颁发于）声明。默认值为 `true`。

#### <Badge type="info" text="可选" /> verifyOptions.exp: `boolean`

如果存在且此项设置为 `true`，则将验证 `exp`（过期时间）声明。默认值为 `true`。
