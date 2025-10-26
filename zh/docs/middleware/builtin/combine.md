# 组合中间件

组合中间件将多个中间件函数组合成一个中间件。它提供了三个函数：

- `some` - 仅运行给定的中间件之一。
- `every` - 运行所有给定的中间件。
- `except` - 仅在不满足条件时才运行所有给定的中间件。

## 导入

```ts
import { Hono } from 'hono'
import { some, every, except } from 'hono/combine'
```

## 用法

以下是使用组合中间件的复杂访问控制规则的示例。

```ts
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'
import { getConnInfo } from 'hono/cloudflare-workers'
import { every, some } from 'hono/combine'
import { ipRestriction } from 'hono/ip-restriction'
import { rateLimit } from '@/my-rate-limit'

const app = new Hono()

app.use(
  '*',
  some(
    every(
      ipRestriction(getConnInfo, { allowList: ['192.168.0.2'] }),
      bearerAuth({ token })
    ),
    // 如果两个条件都满足，则不会执行 rateLimit。
    rateLimit()
  )
)

app.get('/', (c) => c.text('你好 Hono！'))
```

### some

运行第一个返回 true 的中间件。中间件按顺序应用，如果任何中间件成功退出，则不会运行后续中间件。

```ts
import { some } from 'hono/combine'
import { bearerAuth } from 'hono/bearer-auth'
import { myRateLimit } from '@/rate-limit'

// 如果客户端具有有效令牌，则跳过速率限制。
// 否则，应用速率限制。
app.use(
  '/api/*',
  some(bearerAuth({ token }), myRateLimit({ limit: 100 }))
)
```

### every

运行所有中间件，如果其中任何一个失败，则停止。中间件按顺序应用，如果任何中间件抛出错误，则不会运行后续中间件。

```ts
import { some, every } from 'hono/combine'
import { bearerAuth } from 'hono/bearer-auth'
import { myCheckLocalNetwork } from '@/check-local-network'
import { myRateLimit } from '@/rate-limit'

// 如果客户端在本地网络中，则跳过身份验证和速率限制。
// 否则，应用身份验证和速率限制。
app.use(
  '/api/*',
  some(
    myCheckLocalNetwork(),
    every(bearerAuth({ token }), myRateLimit({ limit: 100 }))
  )
)
```

### except

除满足条件外，运行所有中间件。您可以将字符串或函数作为条件传递。如果需要匹配多个目标，请将它们作为数组传递。

```ts
import { except } from 'hono/combine'
import { bearerAuth } from 'hono/bearer-auth'

// 如果客户端正在访问公共 API，则跳过身份验证。
// 否则，需要一个有效的令牌。
app.use('/api/*', except('/api/public/*', bearerAuth({ token })))
```
