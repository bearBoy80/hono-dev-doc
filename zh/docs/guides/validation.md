# 验证

Hono 仅提供一个非常薄的验证器。
但是，当与第三方验证器结合使用时，它会变得非常强大。
此外，RPC 功能允许您通过类型与客户端共享 API 规范。

## 手动验证器

首先，介绍一种不使用第三方验证器来验证传入值的方法。

从 `hono/validator` 导入 `validator`。

```ts
import { validator } from 'hono/validator'
```

要验证表单数据，请将 `form` 指定为第一个参数，并将回调指定为第二个参数。
在回调中，验证值并在最后返回已验证的值。
`validator` 可以用作中间件。

```ts
app.post(
  '/posts',
  validator('form', (value, c) => {
    const body = value['body']
    if (!body || typeof body !== 'string') {
      return c.text('无效！', 400)
    }
    return {
      body: body,
    }
  }),
  //...
```

在处理程序中，您可以使用 `c.req.valid('form')` 获取已验证的值。

```ts
, (c) => {
  const { body } = c.req.valid('form')
  // ... 执行某些操作
  return c.json(
    {
      message: '已创建！',
    },
    201
  )
}
```

除 `form` 外，验证目标还包括 `json`、`query`、`header`、`param` 和 `cookie`。

::: warning
当您验证 `json` 或 `form` 时，请求_必须_包含一个匹配的 `content-type` 标头（例如，对于 `json` 为 `Content-Type: application/json`）。否则，将不会解析请求正文，并且您将在回调中收到一个空对象 (`{}`) 作为值。

在使用
[`app.request()`](../api/request.md) 进行测试时，设置 `content-type` 标头很重要。

给定一个这样的应用程序。

```ts
const app = new Hono()
app.post(
  '/testing',
  validator('json', (value, c) => {
    // 直通验证器
    return value
  }),
  (c) => {
    const body = c.req.valid('json')
    return c.json(body)
  }
)
```

您的测试可以这样编写。

```ts
// ❌ 这不起作用
const res = await app.request('/testing', {
  method: 'POST',
  body: JSON.stringify({ key: 'value' }),
})
const data = await res.json()
console.log(data) // {}

// ✅ 这将起作用
const res = await app.request('/testing', {
  method: 'POST',
  body: JSON.stringify({ key: 'value' }),
  headers: new Headers({ 'Content-Type': 'application/json' }),
})
const data = await res.json()
console.log(data) // { key: 'value' }
```

:::

::: warning
当您验证 `header` 时，您需要使用**小写**名称作为键。

如果您想验证 `Idempotency-Key` 标头，您需要使用 `idempotency-key` 作为键。

```ts
// ❌ 这不起作用
app.post(
  '/api',
  validator('header', (value, c) => {
    // idempotencyKey 始终为 undefined
    // 因此该中间件始终返回 400（非预期）
    const idempotencyKey = value['Idempotency-Key']

    if (idempotencyKey == undefined || idempotencyKey === '') {
      throw new HTTPException(400, {
        message: 'Idempotency-Key is required',
      })
    }
    return { idempotencyKey }
  }),
  (c) => {
    const { idempotencyKey } = c.req.valid('header')
    // ...
  }
)

// ✅ 这将起作用
app.post(
  '/api',
  validator('header', (value, c) => {
    // 可以按预期检索标头的值
    const idempotencyKey = value['idempotency-key']

    if (idempotencyKey == undefined || idempotencyKey === '') {
      throw new HTTPException(400, {
        message: 'Idempotency-Key is required',
      })
    }
    return { idempotencyKey }
  }),
  (c) => {
    const { idempotencyKey } = c.req.valid('header')
    // ...
  }
)
```

:::

## 多个验证器

您还可以包含多个验证器来验证请求的不同部分：

```ts
app.post(
  '/posts/:id',
  validator('param', ...),
  validator('query', ...),
  validator('json', ...),
  (c) => {
    //...
  }
```

## 与 Zod 配合使用

您可以使用 [Zod](https://zod.dev)，一种第三方验证器。
我们建议使用第三方验证器。

从 Npm 注册表安装。

::: code-group

```sh [npm]
npm i zod
```

```sh [yarn]
yarn add zod
```

```sh [pnpm]
pnpm add zod
```

```sh [bun]
bun add zod
```

:::

从 `zod` 导入 `z`。

```ts
import * as z from 'zod'
```

编写您的架构。

```ts
const schema = z.object({
  body: z.string(),
})
```

您可以在回调函数中使用该架构进行验证并返回已验证的值。

```ts
const route = app.post(
  '/posts',
  validator('form', (value, c) => {
    const parsed = schema.safeParse(value)
    if (!parsed.success) {
      return c.text('无效！', 401)
    }
    return parsed.data
  }),
  (c) => {
    const { body } = c.req.valid('form')
    // ... 执行某些操作
    return c.json(
      {
        message: '已创建！',
      },
      201
    )
  }
)
```

## Zod 验证器中间件

您可以使用 [Zod 验证器中间件](https://github.com/honojs/middleware/tree/main/packages/zod-validator) 使其更容易。

::: code-group

```sh [npm]
npm i @hono/zod-validator
```

```sh [yarn]
yarn add @hono/zod-validator
```

```sh [pnpm]
pnpm add @hono/zod-validator
```

```sh [bun]
bun add @hono/zod-validator
```

:::

并导入 `zValidator`。

```ts
import { zValidator } from '@hono/zod-validator'
```

并按如下方式编写。

```ts
const route = app.post(
  '/posts',
  zValidator(
    'form',
    z.object({
      body: z.string(),
    })
  ),
  (c) => {
    const validated = c.req.valid('form')
    // ... 使用您已验证的数据
  }
)
```

## 标准架构验证器中间件

[标准架构](https://standardschema.dev/) 是一个为 TypeScript 验证库提供通用接口的规范。它由 Zod、Valibot 和 ArkType 的维护者创建，允许生态系统工具与任何验证库一起工作，而无需自定义适配器。

[标准架构验证器中间件](https://github.com/honojs/middleware/tree/main/packages/standard-validator) 允许您将任何与标准架构兼容的验证库与 Hono 一起使用，让您可以灵活地选择您喜欢的验证器，同时保持一致的类型安全。

::: code-group

```sh [npm]
npm i @hono/standard-validator
```

```sh [yarn]
yarn add @hono/standard-validator
```

```sh [pnpm]
pnpm add @hono/standard-validator
```

```sh [bun]
bun add @hono/standard-validator
```

:::

从包中导入 `sValidator`：

```ts
import { sValidator } from '@hono/standard-validator'
```

### 与 Zod 配合使用

您可以将 Zod 与标准架构验证器一起使用：

::: code-group

```sh [npm]
npm i zod
```

```sh [yarn]
yarn add zod
```

```sh [pnpm]
pnpm add zod
```

```sh [bun]
bun add zod
```

:::

```ts
import { z } from 'zod'
import { sValidator } from '@hono/standard-validator'

const schema = z.object({
  name: z.string(),
  age: z.number(),
})

app.post('/author', sValidator('json', schema), (c) => {
  const data = c.req.valid('json')
  return c.json({
    success: true,
    message: `${data.name} is ${data.age}`,
  })
})
```

### 与 Valibot 配合使用

[Valibot](https://valibot.dev/) 是 Zod 的轻量级替代品，采用模块化设计：

::: code-group

```sh [npm]
npm i valibot
```

```sh [yarn]
yarn add valibot
```

```sh [pnpm]
pnpm add valibot
```

```sh [bun]
bun add valibot
```

:::

```ts
import * as v from 'valibot'
import { sValidator } from '@hono/standard-validator'

const schema = v.object({
  name: v.string(),
  age: v.number(),
})

app.post('/author', sValidator('json', schema), (c) => {
  const data = c.req.valid('json')
  return c.json({
    success: true,
    message: `${data.name} is ${data.age}`,
  })
})
```

### 与 ArkType 配合使用

[ArkType](https://arktype.io/) 提供 TypeScript 原生语法进行运行时验证：

::: code-group

```sh [npm]
npm i arktype
```

```sh [yarn]
yarn add arktype
```

```sh [pnpm]
pnpm add arktype
```

```sh [bun]
bun add arktype
```

:::

```ts
import { type } from 'arktype'
import { sValidator } from '@hono/standard-validator'

const schema = type({
  name: 'string',
  age: 'number',
})

app.post('/author', sValidator('json', schema), (c) => {
  const data = c.req.valid('json')
  return c.json({
    success: true,
    message: `${data.name} is ${data.age}`,
  })
})
```
