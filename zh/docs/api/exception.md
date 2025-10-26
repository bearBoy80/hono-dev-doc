# HTTPException

当发生致命错误时，Hono（以及许多生态系统中间件）可能会抛出 `HTTPException`。这是一个自定义的 Hono `Error`，它简化了[返回错误响应](#handling-httpexceptions)的过程。

## 抛出 HTTPExceptions

您可以通过指定状态码以及消息或自定义响应来抛出自己的 HTTPExceptions。

### 自定义消息

对于基本的 `text` 响应，只需设置错误 `message`。

```ts twoslash
import { HTTPException } from 'hono/http-exception'

throw new HTTPException(401, { message: '未经授权' })
```

### 自定义响应

对于其他响应类型，或要设置响应标头，请使用 `res` 选项。_请注意，传递给构造函数的状态是用于创建响应的状态。_

```ts twoslash
import { HTTPException } from 'hono/http-exception'

const errorResponse = new Response('未经授权', {
  status: 401, // 这将被忽略
  headers: {
    Authenticate: 'error="invalid_token"',
  },
})

throw new HTTPException(401, { res: errorResponse })
```

### 原因

在任何一种情况下，您都可以使用 [`cause`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/cause) 选项向 HTTPException 添加任意数据。

```ts twoslash
import { Hono, Context } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const message: string
declare const authorize: (c: Context) => Promise<void>
// ---cut---
app.post('/login', async (c) => {
  try {
    await authorize(c)
  } catch (cause) {
    throw new HTTPException(401, { message, cause })
  }
  return c.redirect('/')
})
```

## 处理 HTTPExceptions

您可以使用 [`app.onError`](/docs/api/hono#error-handling) 处理未捕获的 HTTPExceptions。它们包含一个 `getResponse` 方法，该方法返回一个从错误 `status` 创建的新 `Response`，以及错误 `message` 或抛出错误时设置的[自定义响应](#custom-response)。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
import { HTTPException } from 'hono/http-exception'

// ...

app.onError((error, c) => {
  if (error instanceof HTTPException) {
    console.error(error.cause)
    // 获取自定义响应
    return error.getResponse()
  }
  // ...
  // ---cut-start---
  return c.text('意外错误')
  // ---cut-end---
})
```

::: warning
**`HTTPException.getResponse` 不知道 `Context`**。要包含已在 `Context` 中设置的标头，您必须将它们应用于新的 `Response`。
:::
