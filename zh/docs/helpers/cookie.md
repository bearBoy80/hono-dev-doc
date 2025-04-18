---
title: Cookie 工具类
description: 使用 Cookie 工具类，可以方便地设置、解析和删除 cookies。
---

# Cookie 工具类

Cookie 工具类提供了一个简单的接口来管理 cookies，使开发者能够无缝地设置、解析和删除 cookies。

## 导入

```ts
import { Hono } from 'hono'
import {
  getCookie,
  getSignedCookie,
  setCookie,
  setSignedCookie,
  deleteCookie,
} from 'hono/cookie'
```

## 使用方法

### 普通 cookies

```ts
app.get('/cookie', (c) => {
  setCookie(c, 'cookie_name', 'cookie_value')
  const yummyCookie = getCookie(c, 'cookie_name')
  deleteCookie(c, 'cookie_name')
  const allCookies = getCookie(c)
  // ...
})
```
### 签名 cookies

**注意**：由于使用 WebCrypto API 创建 HMAC SHA-256 签名的异步特性，设置和获取签名 cookies 会返回 Promise。

```ts
app.get('/signed-cookie', (c) => {
  const secret = 'secret' // 确保使用足够长的字符串来保证安全性

  await setSignedCookie(c, 'cookie_name0', 'cookie_value', secret)
  const fortuneCookie = await getSignedCookie(c, secret, 'cookie_name0')
  deleteCookie(c, 'cookie_name0')
  // 如果签名被篡改或无效，`getSignedCookie` 将对指定的 cookie 返回 `false`
  const allSignedCookies = await getSignedCookie(c, secret)
  // ...
})
```

## 选项

### `setCookie` 和 `setSignedCookie`

- domain: `string` - 域名
- expires: `Date` - 过期时间
- httpOnly: `boolean` - 是否仅允许 HTTP 访问
- maxAge: `number` - 最大存活时间
- path: `string` - 路径
- secure: `boolean` - 是否仅通过 HTTPS 传输
- sameSite: `'Strict'` | `'Lax'` | `'None'` - 跨站点设置
- priority: `'Low' | 'Medium' | 'High'` - 优先级
- prefix: `'secure'` | `'host'` - 前缀
- partitioned: `boolean` - 是否分区

示例：

```ts
// 普通 cookies
setCookie(c, 'great_cookie', 'banana', {
  path: '/',
  secure: true,
  domain: 'example.com',
  httpOnly: true,
  maxAge: 1000,
  expires: new Date(Date.UTC(2000, 11, 24, 10, 30, 59, 900)),
  sameSite: 'Strict',
})

// 签名 cookies
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

- path: `string` - 路径
- secure: `boolean` - 是否仅通过 HTTPS 传输
- domain: `string` - 域名

示例：

```ts
deleteCookie(c, 'banana', {
  path: '/',
  secure: true,
  domain: 'example.com',
})
```

`deleteCookie` 会返回被删除的值：

```ts
const deletedCookie = deleteCookie(c, 'delicious_cookie')
```

## `__Secure-` 和 `__Host-` 前缀

Cookie 工具类支持 cookie 名称使用 `__Secure-` 和 `__Host-` 前缀。

如果你想验证 cookie 名称是否有前缀，请指定前缀选项：

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

同样，如果你想在设置 cookie 时指定前缀，请为前缀选项指定一个值：

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

新的 Cookie RFC（又称 cookie-bis）和 CHIPS 包含了一些开发者应该遵循的 Cookie 设置最佳实践。

- [RFC6265bis-13](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis-13)
  - `Max-Age`/`Expires` 限制
  - `__Host-`/`__Secure-` 前缀限制
- [CHIPS-01](https://www.ietf.org/archive/id/draft-cutler-httpbis-partitioned-cookies-01.html)
  - `Partitioned` 限制

Hono 遵循这些最佳实践。
在以下情况下，cookie 工具类在解析 cookies 时会抛出 `Error`：

- cookie 名称以 `__Secure-` 开头，但未设置 `secure` 选项
- cookie 名称以 `__Host-` 开头，但未设置 `secure` 选项
- cookie 名称以 `__Host-` 开头，但 `path` 不是 `/`
- cookie 名称以 `__Host-` 开头，但设置了 `domain`
- `maxAge` 选项值大于 400 天
- `expires` 选项值超过当前时间 400 天
