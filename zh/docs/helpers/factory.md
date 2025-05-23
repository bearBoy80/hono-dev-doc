---
title: Factory 工具类
description: 使用 Factory 工具类，可以方便地创建 Hono 的各种组件，如中间件等。
---

# Factory 工具类

Factory 工具类提供了一些实用的函数，用于创建 Hono 的各种组件，比如中间件。有时候设置正确的 TypeScript 类型可能比较困难，但这个工具类可以帮助你简化这个过程。

## 导入

```ts
import { Hono } from 'hono'
import { createFactory, createMiddleware } from 'hono/factory'
```

## `createFactory()`

`createFactory()` 用于创建 Factory 类的实例。

```ts
import { createFactory } from 'hono/factory'

const factory = createFactory()
```

你可以通过泛型传入你的环境类型：

```ts
type Env = {
  Variables: {
    foo: string
  }
}

const factory = createFactory<Env>()
```

### 选项

### <Badge type="info" text="可选" /> defaultAppOptions: `HonoOptions`

用于传递给 `createApp()` 创建的 Hono 应用程序的默认选项。

```ts
const factory = createFactory({
  defaultAppOptions: { strict: false },
})

const app = factory.createApp() // 应用 `strict: false` 选项
```

## `createMiddleware()`

`createMiddleware()` 是 `factory.createMiddleware()` 的快捷方式。
这个函数用于创建你的自定义中间件。

```ts
const messageMiddleware = createMiddleware(async (c, next) => {
  await next()
  c.res.headers.set('X-Message', '早上好！')
})
```

提示：如果你想要获取参数（比如 `message`），你可以像下面这样创建一个函数：

```ts
const messageMiddleware = (message: string) => {
  return createMiddleware(async (c, next) => {
    await next()
    c.res.headers.set('X-Message', message)
  })
}

app.use(messageMiddleware('晚上好！'))
```

## `factory.createHandlers()`

`createHandlers()` 帮助你在不同于 `app.get('/')` 的地方定义处理程序。

```ts
import { createFactory } from 'hono/factory'
import { logger } from 'hono/logger'

// ...

const factory = createFactory()

const middleware = factory.createMiddleware(async (c, next) => {
  c.set('foo', 'bar')
  await next()
})

const handlers = factory.createHandlers(logger(), middleware, (c) => {
  return c.json(c.var.foo)
})

app.get('/api', ...handlers)
```

## `factory.createApp()`

`createApp()` 帮助你创建具有正确类型的 Hono 实例。如果你将此方法与 `createFactory()` 一起使用，可以避免在定义 `Env` 类型时的重复。

如果你的应用程序像这样，你必须在两个地方设置 `Env`：

```ts
import { createMiddleware } from 'hono/factory'

type Env = {
  Variables: {
    myVar: string
  }
}

// 1. 在 `new Hono()` 中设置 `Env`
const app = new Hono<Env>()

// 2. 在 `createMiddleware()` 中设置 `Env`
const mw = createMiddleware<Env>(async (c, next) => {
  await next()
})

app.use(mw)
```

通过使用 `createFactory()` 和 `createApp()`，你只需要在一个地方设置 `Env`：

```ts
import { createFactory } from 'hono/factory'

// ...

// 在 `createFactory()` 中设置 `Env`
const factory = createFactory<Env>()

const app = factory.createApp()

// factory 也有 `createMiddleware()`
const mw = factory.createMiddleware(async (c, next) => {
  await next()
})
```

`createFactory()` 可以接收 `initApp` 选项来初始化由 `createApp()` 创建的 `app`。以下是使用该选项的示例：

```ts
// factory-with-db.ts
type Env = {
  Bindings: {
    MY_DB: D1Database
  }
  Variables: {
    db: DrizzleD1Database
  }
}

export default createFactory<Env>({
  initApp: (app) => {
    app.use(async (c, next) => {
      const db = drizzle(c.env.MY_DB)
      c.set('db', db)
      await next()
    })
  },
})
```

```ts
// crud.ts
import factoryWithDB from './factory-with-db'

const app = factoryWithDB.createApp()

app.post('/posts', (c) => {
  c.var.db.insert()
  // ...
})
```
