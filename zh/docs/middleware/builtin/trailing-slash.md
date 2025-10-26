# 尾部斜杠中间件

此中间件处理 GET 请求中 URL 的尾部斜杠。

`appendTrailingSlash` 会在未找到内容时将 URL 重定向到添加了尾部斜杠的 URL。此外，`trimTrailingSlash` 将删除尾部斜杠。

## 导入

```ts
import { Hono } from 'hono'
import {
  appendTrailingSlash,
  trimTrailingSlash,
} from 'hono/trailing-slash'
```

## 用法

将 `/about/me` 的 GET 请求重定向到 `/about/me/` 的示例。

```ts
import { Hono } from 'hono'
import { appendTrailingSlash } from 'hono/trailing-slash'

const app = new Hono({ strict: true })

app.use(appendTrailingSlash())
app.get('/about/me/', (c) => c.text('带尾部斜杠'))
```

将 `/about/me/` 的 GET 请求重定向到 `/about/me` 的示例。

```ts
import { Hono } from 'hono'
import { trimTrailingSlash } from 'hono/trailing-slash'

const app = new Hono({ strict: true })

app.use(trimTrailingSlash())
app.get('/about/me', (c) => c.text('不带尾部斜杠'))
```

## 注意

当请求方法为 `GET` 且响应状态为 `404` 时，它将被启用。
