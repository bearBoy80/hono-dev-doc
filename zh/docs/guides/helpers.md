# 辅助函数

辅助函数可用于协助您开发应用程序。与中间件不同，它们不充当处理程序，而是提供有用的函数。

例如，以下是如何使用 [Cookie 辅助函数](/docs/helpers/cookie)：

```ts
import { getCookie, setCookie } from 'hono/cookie'

const app = new Hono()

app.get('/cookie', (c) => {
  const yummyCookie = getCookie(c, 'yummy_cookie')
  // ...
  setCookie(c, 'delicious_cookie', 'macha')
  //
})
```

## 可用的辅助函数

- [接受](/docs/helpers/accepts)
- [适配器](/docs/helpers/adapter)
- [Cookie](/docs/helpers/cookie)
- [css](/docs/helpers/css)
- [开发](/docs/helpers/dev)
- [工厂](/docs/helpers/factory)
- [html](/docs/helpers/html)
- [JWT](/docs/helpers/jwt)
- [SSG](/docs/helpers/ssg)
- [流式传输](/docs/helpers/streaming)
- [测试](/docs/helpers/testing)
- [WebSocket](/docs/helpers/websocket)
