# 最佳实践

Hono 非常灵活。您可以随心所欲地编写您的应用程序。
然而，遵循一些最佳实践会更好。

## 尽可能不要创建“控制器”

如果可能，您不应该创建“类似 Ruby on Rails 的控制器”。

```ts
// 🙁
// 一个类似 RoR 的控制器
const booksList = (c: Context) => {
  return c.json('图书列表')
}

app.get('/books', booksList)
```

问题与类型有关。例如，在不编写复杂泛型的情况下，无法在控制器中推断路径参数。

```ts
// 🙁
// 一个类似 RoR 的控制器
const bookPermalink = (c: Context) => {
  const id = c.req.param('id') // 无法推断路径参数
  return c.json(`获取 ${id}`)
}
```

因此，您不需要创建类似 RoR 的控制器，而应直接在路径定义后编写处理程序。

```ts
// 😃
app.get('/books/:id', (c) => {
  const id = c.req.param('id') // 可以推断路径参数
  return c.json(`获取 ${id}`)
})
```

## `hono/factory` 中的 `factory.createHandlers()`

如果您仍然想创建一个类似 RoR 的控制器，请使用 [`hono/factory`](/docs/helpers/factory) 中的 `factory.createHandlers()`。如果使用此方法，类型推断将正常工作。

```ts
import { createFactory } from 'hono/factory'
import { logger } from 'hono/logger'

// ...

// 😃
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

## 构建更大的应用程序

使用 `app.route()` 构建更大的应用程序，而无需创建“类似 Ruby on Rails 的控制器”。

如果您的应用程序有 `/authors` 和 `/books` 端点，并且您希望将文件与 `index.ts` 分开，请创建 `authors.ts` 和 `books.ts`。

```ts
// authors.ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.json('作者列表'))
app.post('/', (c) => c.json('创建作者', 201))
app.get('/:id', (c) => c.json(`获取 ${c.req.param('id')}`))

export default app
```

```ts
// books.ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.json('图书列表'))
app.post('/', (c) => c.json('创建图书', 201))
app.get('/:id', (c) => c.json(`获取 ${c.req.param('id')}`))

export default app
```

然后，导入它们并使用 `app.route()` 将它们挂载到路径 `/authors` 和 `/books` 上。

```ts
// index.ts
import { Hono } from 'hono'
import authors from './authors'
import books from './books'

const app = new Hono()

// 😃
app.route('/authors', authors)
app.route('/books', books)

export default app
```

### 如果您想使用 RPC 功能

上面的代码在正常用例中运行良好。
但是，如果您想使用 `RPC` 功能，可以通过如下链接获得正确的类型。

```ts
// authors.ts
import { Hono } from 'hono'

const app = new Hono()
  .get('/', (c) => c.json('作者列表'))
  .post('/', (c) => c.json('创建作者', 201))
  .get('/:id', (c) => c.json(`获取 ${c.req.param('id')}`))

export default app
export type AppType = typeof app
```

如果将 `app` 的类型传递给 `hc`，它将获得正确的类型。

```ts
import type { AppType } from './authors'
import { hc } from 'hono/client'

// 😃
const client = hc<AppType>('http://localhost') // 类型正确
```

有关更详细的信息，请参阅 [RPC 页面](/docs/guides/rpc#using-rpc-with-larger-applications)。
