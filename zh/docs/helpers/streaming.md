# 流式辅助函数

流式辅助函数提供了用于流式响应的方法。

## 导入

```ts
import { Hono } from 'hono'
import { stream, streamText, streamSSE } from 'hono/streaming'
```

## `stream()`

它以 `Response` 对象的形式返回一个简单的流式响应。

```ts
app.get('/stream', (c) => {
  return stream(c, async (stream) => {
    // 写入一个在中止时执行的进程。
    stream.onAbort(() => {
      console.log('已中止！')
    })
    // 写入一个 Uint8Array。
    await stream.write(new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f]))
    // 管道一个可读流。
    await stream.pipe(anotherReadableStream)
  })
})
```

## `streamText()`

它返回一个带有 `Content-Type:text/plain`、`Transfer-Encoding:chunked` 和 `X-Content-Type-Options:nosniff` 标头的流式响应。

```ts
app.get('/streamText', (c) => {
  return streamText(c, async (stream) => {
    // 写入一行带换行符 ('\n') 的文本。
    await stream.writeln('你好')
    // 等待 1 秒。
    await stream.sleep(1000)
    // 写入一行不带换行符的文本。
    await stream.write(`Hono！`)
  })
})
```

::: warning

如果您正在为 Cloudflare Workers 开发应用程序，流式传输在 Wrangler 上可能无法正常工作。如果是这样，请为 `Content-Encoding` 标头添加 `Identity`。

```ts
app.get('/streamText', (c) => {
  c.header('Content-Encoding', 'Identity')
  return streamText(c, async (stream) => {
    // ...
  })
})
```

:::

## `streamSSE()`

它允许您无缝地流式传输服务器发送事件 (SSE)。

```ts
const app = new Hono()
let id = 0

app.get('/sse', async (c) => {
  return streamSSE(c, async (stream) => {
    while (true) {
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

## 错误处理

流式辅助函数的第三个参数是错误处理程序。
此参数是可选的，如果您不指定它，错误将作为控制台错误输出。

```ts
app.get('/stream', (c) => {
  return stream(
    c,
    async (stream) => {
      // 写入一个在中止时执行的进程。
      stream.onAbort(() => {
        console.log('已中止！')
      })
      // 写入一个 Uint8Array。
      await stream.write(
        new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f])
      )
      // 管道一个可读流。
      await stream.pipe(anotherReadableStream)
    },
    (err, stream) => {
      stream.writeln('发生错误！')
      console.error(err)
    }
  )
})
```

回调执行后，流将自动关闭。

::: warning

如果流式辅助函数的回调函数抛出错误，则不会触发 Hono 的 `onError` 事件。

`onError` 是一个在发送响应之前处理错误并覆盖响应的钩子。但是，当执行回调函数时，流已经开始，因此无法覆盖。

:::
