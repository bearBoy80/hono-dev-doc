# 超时中间件

超时中间件使您能够轻松地管理应用程序中的请求超时。它允许您为请求设置最大持续时间，并可选择性地定义在超过指定超时时返回的自定义错误响应。

## 导入

```ts
import { Hono } from 'hono'
import { timeout } from 'hono/timeout'
```

## 用法

以下是如何使用超时中间件的默认设置和自定义设置：

默认设置：

```ts
const app = new Hono()

// 应用 5 秒超时
app.use('/api', timeout(5000))

// 处理路由
app.get('/api/data', async (c) => {
  // 您的路由处理程序逻辑
  return c.json({ data: '此处为您的数据' })
})
```

自定义设置：

```ts
import { HTTPException } from 'hono/http-exception'

// 自定义异常工厂函数
const customTimeoutException = (context) =>
  new HTTPException(408, {
    message: `等待 ${context.req.headers.get(
      'Duration'
    )} 秒后请求超时。请稍后再试。`,
  })

// 用于静态异常消息
// const customTimeoutException = new HTTPException(408, {
//   message: '操作超时。请稍后再试。'
// });

// 应用 1 分钟超时并带有自定义异常
app.use('/api/long-process', timeout(60000, customTimeoutException))

app.get('/api/long-process', async (c) => {
  // 模拟一个长进程
  await new Promise((resolve) => setTimeout(resolve, 61000))
  return c.json({ data: '这通常需要更长的时间' })
})
```

## 注意事项

- 超时持续时间可以以毫秒为单位指定。如果超过指定的持续时间，中间件将自动拒绝 Promise 并可能抛出错误。

- 超时中间件不能与流一起使用，因此，请将 `stream.close` 和 `setTimeout` 一起使用。

```ts
app.get('/sse', async (c) => {
  let id = 0
  let running = true
  let timer: number | undefined

  return streamSSE(c, async (stream) => {
    timer = setTimeout(() => {
      console.log('流超时，正在关闭流')
      stream.close()
    }, 3000) as unknown as number

    stream.onAbort(async () => {
      console.log('客户端关闭连接')
      running = false
      clearTimeout(timer)
    })

    while (running) {
      const message = `现在是 ${new Date().toISOString()}`
      await stream.writeSSE({
        data: message,
        event: 'time-update',
        id: String(id++),
      })
      await stream.sleep(1000)
    }
  })
})
```

## 中间件冲突

请注意中间件的顺序，尤其是在使用错误处理或其他与时间相关的中间件时，因为它可能会影响此超时中间件的行为。
