# 测试辅助函数

测试辅助函数提供了使 Hono 应用程序的测试更容易的函数。

## 导入

```ts
import { Hono } from 'hono'
import { testClient } from 'hono/testing'
```

## `testClient()`

`testClient()` 函数将 Hono 的实例作为其第一个参数，并返回一个根据您的 Hono 应用程序的路由类型化的对象，类似于 [Hono 客户端](/docs/guides/rpc#client)。这使您可以在测试中使用编辑器自动完成功能以类型安全的方式调用您定义的路由。

**关于类型推断的重要说明：**

为了让 `testClient` 正确推断您的路由类型并提供自动完成功能，**您必须直接在 `Hono` 实例上使用链式方法定义您的路由**。

类型推断依赖于流经链式 `.get()`、`.post()` 等调用的类型。如果您在创建 Hono 实例后单独定义路由（如“Hello World”示例中显示的常见模式：`const app = new Hono(); app.get(...)`），`testClient` 将没有特定路由所需的类型信息，您将无法获得类型安全的客户端功能。

**示例：**

此示例有效，因为 `.get()` 方法直接链接到 `new Hono()` 调用：

```ts
// index.ts
const app = new Hono().get('/search', (c) => {
  const query = c.req.query('q')
  return c.json({ query: query, results: ['result1', 'result2'] })
})

export default app
```

```ts
// index.test.ts
import { Hono } from 'hono'
import { testClient } from 'hono/testing'
import { describe, it, expect } from 'vitest' // 或您首选的测试运行器
import app from './app'

describe('搜索端点', () => {
  // 从应用实例创建测试客户端
  const client = testClient(app)

  it('应返回搜索结果', async () => {
    // 使用类型化的客户端调用端点
    // 注意查询参数的类型安全（如果在路由中定义）
    // 以及通过 .$get() 的直接访问
    const res = await client.search.$get({
      query: { q: 'hono' },
    })

    // 断言
    expect(res.status).toBe(200)
    expect(await res.json()).toEqual({
      query: 'hono',
      results: ['result1', 'result2'],
    })
  })
})
```

要在测试中包含标头，请在调用中将其作为第二个参数传递。第二个参数还可以将 `init` 属性作为 `RequestInit` 对象，允许您设置标头、方法、正文等。在此处了解有关 `init` 属性的更多信息[/docs/guides/rpc#init-option](/docs/guides/rpc#init-option)。

```ts
// index.test.ts
import { Hono } from 'hono'
import { testClient } from 'hono/testing'
import { describe, it, expect } from 'vitest' // 或您首选的测试运行器
import app from './app'

describe('搜索端点', () => {
  // 从应用实例创建测试客户端
  const client = testClient(app)

  it('应返回搜索结果', async () => {
    // 在标头中包含令牌并设置内容类型
    const token = 'this-is-a-very-clean-token'
    const res = await client.search.$get(
      {
        query: { q: 'hono' },
      },
      {
        headers: {
          Authorization: `Bearer ${token}`,
          'Content-Type': `application/json`,
        },
      }
    )

    // 断言
    expect(res.status).toBe(200)
    expect(await res.json()).toEqual({
      query: 'hono',
      results: ['result1', 'result2'],
    })
  })
})
```
