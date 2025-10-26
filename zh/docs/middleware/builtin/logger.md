# 日志记录器中间件

这是一个简单的日志记录器。

## 导入

```ts
import { Hono } from 'hono'
import { logger } from 'hono/logger'
```

## 用法

```ts
const app = new Hono()

app.use(logger())
app.get('/', (c) => c.text('你好 Hono！'))
```

## 日志记录详细信息

日志记录器中间件为每个请求记录以下详细信息：

- **传入请求**：记录 HTTP 方法、请求路径和传入请求。
- **传出响应**：记录 HTTP 方法、请求路径、响应状态码和请求/响应时间。
- **状态码着色**：响应状态码会进行颜色编码，以获得更好的可见性并快速识别状态类别。不同的状态码类别由不同的颜色表示。
- **经过时间**：请求/响应周期所花费的时间以人类可读的格式记录，单位为毫秒 (ms) 或秒 (s)。

通过使用日志记录器中间件，您可以轻松监控 Hono 应用程序中请求和响应的流程，并快速识别任何问题或性能瓶颈。

您还可以通过提供自己的 `PrintFunc` 函数来进一步扩展中间件，以实现量身定制的日志记录行为。

## PrintFunc

日志记录器中间件接受一个可选的 `PrintFunc` 函数作为参数。此函数允许您自定义日志记录器并添加其他日志。

## 选项

### <Badge type="info" text="可选" /> fn: `PrintFunc(str: string, ...rest: string[])`

- `str`：由日志记录器传递。
- `...rest`：要打印到控制台的其他字符串属性。

### 示例

为日志记录器中间件设置自定义 `PrintFunc` 函数：

```ts
export const customLogger = (message: string, ...rest: string[]) => {
  console.log(message, ...rest)
}

app.use(logger(customLogger))
```

在路由中设置自定义日志记录器：

```ts
app.post('/blog', (c) => {
  // 路由逻辑

  customLogger('博客已保存：', `路径：${blog.url},`, `ID：${blog.id}`)
  // 输出
  // <-- POST /blog
  // 博客已保存：路径：/blog/example, ID：1
  // --> POST /blog 201 93ms

  // 返回上下文
})
```
