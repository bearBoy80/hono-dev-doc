# 基本身份验证中间件

此中间件可以对指定路径应用基本身份验证。
使用 Cloudflare Workers 或其他平台实现基本身份验证比看起来更复杂，但使用此中间件，一切都变得轻而易举。

有关基本身份验证方案工作原理的更多信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#basic_authentication_scheme)。

## 导入

```ts
import { Hono } from 'hono'
import { basicAuth } from 'hono/basic-auth'
```

## 用法

```ts
const app = new Hono()

app.use(
  '/auth/*',
  basicAuth({
    username: 'hono',
    password: 'acoolproject',
  })
)

app.get('/auth/page', (c) => {
  return c.text('您已获得授权')
})
```

要限制到特定的路由 + 方法：

```ts
const app = new Hono()

app.get('/auth/page', (c) => {
  return c.text('正在查看页面')
})

app.delete(
  '/auth/page',
  basicAuth({ username: 'hono', password: 'acoolproject' }),
  (c) => {
    return c.text('页面已删除')
  }
)
```

如果您想自己验证用户，请指定 `verifyUser` 选项；返回 `true` 表示接受。

```ts
const app = new Hono()

app.use(
  basicAuth({
    verifyUser: (username, password, c) => {
      return (
        username === 'dynamic-user' && password === 'hono-password'
      )
    },
  })
)
```

## 选项

### <Badge type="danger" text="必需" /> username: `string`

进行身份验证的用户的用户名。

### <Badge type="danger" text="必需" /> password: `string`

用于对提供的用户名进行身份验证的密码值。

### <Badge type="info" text="可选" /> realm: `string`

域的域名，作为返回的 WWW-Authenticate 质询标头的一部分。默认值为 `"Secure Area"`。
更多信息请参阅：https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate#directives

### <Badge type="info" text="可选" /> hashFunction: `Function`

用于处理哈希以安全比较密码的函数。

### <Badge type="info" text="可选" /> verifyUser: `(username: string, password: string, c: Context) => boolean | Promise<boolean>`

用于验证用户的函数。

### <Badge type="info" text="可选" /> invalidUserMessage: `string | object | MessageFunction`

`MessageFunction` 是 `(c: Context) => string | object | Promise<string | object>`。如果用户无效，则显示自定义消息。

## 更多选项

### <Badge type="info" text="可选" /> ...users: `{ username: string, password: string }[]`

## 方案

### 定义多个用户

此中间件还允许您传递包含定义更多 `username` 和 `password` 对的对象的任意参数。

```ts
app.use(
  '/auth/*',
  basicAuth(
    {
      username: 'hono',
      password: 'acoolproject',
      // 在第一个对象中定义其他参数
      realm: 'www.example.com',
    },
    {
      username: 'hono-admin',
      password: 'super-secure',
      // 此处无法重新定义其他参数
    },
    {
      username: 'hono-user-1',
      password: 'a-secret',
      // 或此处
    }
  )
)
```

或者不那么硬编码：

```ts
import { users } from '../config/users'

app.use(
  '/auth/*',
  basicAuth(
    {
      realm: 'www.example.com',
      ...users[0],
    },
    ...users.slice(1)
  )
)
```
