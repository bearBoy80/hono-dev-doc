# Cookie 辅助函数

Cookie 辅助函数提供了一个简单的界面来管理 cookie，使开发人员能够无缝地设置、解析和删除 cookie。

## 导入

```ts
import { Hono } from 'hono'
import {
  deleteCookie,
  getCookie,
  getSignedCookie,
  setCookie,
  setSignedCookie,
  generateCookie,
  generateSignedCookie,
} from 'hono/cookie'
```

## 用法

### 常规 cookie

```ts
app.get('/cookie', (c) => {
  setCookie(c, 'cookie_name', 'cookie_value')
  const yummyCookie = getCookie(c, 'cookie_name')
  deleteCookie(c, 'cookie_name')
  const allCookies = getCookie(c)
  // ...
})
```

### 签名 cookie

**注意**：设置和检索签名 cookie 会返回一个 Promise，这是因为用于创建 HMAC SHA-256 签名的 WebCrypto API 的异步特性。

```ts
app.get('/signed-cookie', (c) => {
  const secret = 'secret' // 确保它是一个足够长的字符串以确保安全

  await setSignedCookie(c, 'cookie_name0', 'cookie_value', secret)
  const fortuneCookie = await getSignedCookie(
    c,
    secret,
    'cookie_name0'
  )
  deleteCookie(c, 'cookie_name0')
  // 如果签名被篡改或无效，`getSignedCookie` 将为指定的 cookie 返回 `false`
  const allSignedCookies = await getSignedCookie(c, secret)
  // ...
})
```

### Cookie 生成

`generateCookie` 和 `generateSignedCookie` 函数允许您直接创建 cookie 字符串，而无需在响应标头中设置它们。

#### `generateCookie`

```ts
// 基本 cookie 生成
const cookie = generateCookie('delicious_cookie', 'macha')
// 返回：'delicious_cookie=macha; Path=/'

// 带选项的 cookie
const cookie = generateCookie('delicious_cookie', 'macha', {
  path: '/',
  secure: true,
  httpOnly: true,
  domain: 'example.com',
})
```

#### `generateSignedCookie`

```ts
// 基本签名 cookie 生成
const signedCookie = await generateSignedCookie(
  'delicious_cookie',
  'macha',
  'secret chocolate chips'
)

// 带选项的签名 cookie
const signedCookie = await generateSignedCookie(
  'delicious_cookie',
  'macha',
  'secret chocolate chips',
  {
    path: '/',
    secure: true,
    httpOnly: true,
  }
)
```

**注意**：与 `setCookie` 和 `setSignedCookie` 不同，这些函数仅生成 cookie 字符串。如果需要，您需要手动在标头中设置它们。

## 选项

### `setCookie` 和 `setSignedCookie`

- domain: `string`
- expires: `Date`
- httpOnly: `boolean`
- maxAge: `number`
- path: `string`
- secure: `boolean`
- sameSite: `'Strict'` | `'Lax'` | `'None'`
- priority: `'Low' | 'Medium' | 'High'`
- prefix: `secure` | `'host'`
- partitioned: `boolean`

示例：

```ts
// 常规 cookie
setCookie(c, 'great_cookie', 'banana', {
  path: '/',
  secure: true,
  domain: 'example.com',
  httpOnly: true,
  maxAge: 1000,
  expires: new Date(Date.UTC(2000, 11, 24, 10, 30, 59, 900)),
  sameSite: 'Strict',
})

// 签名 cookie
await setSignedCookie(
  c,
  'fortune_cookie',
  'lots-of-money',
  'secret ingredient',
  {
    path: '/',
    secure: true,
    domain: 'example.com',
    httpOnly: true,
    maxAge: 1000,
    expires: new Date(Date.UTC(2000, 11, 24, 10, 30, 59, 900)),
    sameSite: 'Strict',
  }
)
```

### `deleteCookie`

- path: `string`
- secure: `boolean`
- domain: `string`

示例：

```ts
deleteCookie(c, 'banana', {
  path: '/',
  secure: true,
  domain: 'example.com',
})
```

`deleteCookie` 返回已删除的值：

```ts
const deletedCookie = deleteCookie(c, 'delicious_cookie')
```

## `__Secure-` 和 `__Host-` 前缀

Cookie 辅助函数支持 cookie 名称的 `__Secure-` 和 `__Host-` 前缀。

如果要验证 cookie 名称是否具有前缀，请指定 prefix 选项。

```ts
const securePrefixCookie = getCookie(c, 'yummy_cookie', 'secure')
const hostPrefixCookie = getCookie(c, 'yummy_cookie', 'host')

const securePrefixSignedCookie = await getSignedCookie(
  c,
  secret,
  'fortune_cookie',
  'secure'
)
const hostPrefixSignedCookie = await getSignedCookie(
  c,
  secret,
  'fortune_cookie',
  'host'
)
```

此外，如果您希望在设置 cookie 时指定前缀，请为 prefix 选项指定一个值。

```ts
setCookie(c, 'delicious_cookie', 'macha', {
  prefix: 'secure', // 或 `host`
})

await setSignedCookie(
  c,
  'delicious_cookie',
  'macha',
  'secret choco chips',
  {
    prefix: 'secure', // 或 `host`
  }
)
```

## 遵循最佳实践

一个新的 Cookie RFC（也称为 cookie-bis）和 CHIPS 包括一些开发人员应遵循的 Cookie 设置最佳实践。

- [RFC6265bis-13](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis-13)
  - `Max-Age`/`Expires` 限制
  - `__Host-`/`__Secure-` 前缀限制
- [CHIPS-01](https://www.ietf.org/archive/id/draft-cutler-httpbis-partitioned-cookies-01.html)
  - `Partitioned` 限制

Hono 遵循最佳实践。
在以下情况下解析 cookie 时，cookie 辅助函数将抛出 `Error`：

- cookie 名称以 `__Secure-` 开头，但未设置 `secure` 选项。
- cookie 名称以 `__Host-` 开头，但未设置 `secure` 选项。
- cookie 名称以 `__Host-` 开头，但 `path` 不是 `/`。
- cookie 名称以 `__Host-` 开头，但设置了 `domain`。
- `maxAge` 选项值大于 400 天。
- `expires` 选项值比当前时间晚 400 天。
