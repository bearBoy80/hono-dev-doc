# Hono 技术栈

Hono 让简单的事情变得简单，让困难的事情也变得简单。
它不仅适用于返回 JSON。
它也非常适合构建包括 REST API 服务器和客户端的全栈应用程序。

## RPC

Hono 的 RPC 功能允许您以少量代码改动即可共享 API 规范。
由 `hc` 生成的客户端将读取规范并以类型安全的方式访问端点。

以下库使其成为可能。

- Hono - API 服务器
- [Zod](https://zod.dev) - 验证器
- [Zod 验证器中间件](https://github.com/honojs/middleware/tree/main/packages/zod-validator)
- `hc` - HTTP 客户端

我们可以将这些组件的集合称为 **Hono 技术栈**。
现在让我们用它来创建一个 API 服务器和一个客户端。

## 编写 API

首先，编写一个接收 GET 请求并返回 JSON 的端点。

```ts twoslash
import { Hono } from 'hono'

const app = new Hono()

app.get('/hello', (c) => {
  return c.json({
    message: `你好！`,
  })
})
```

## 使用 Zod 进行验证

使用 Zod 进行验证以接收查询参数的值。

![](/images/sc01.gif)

```ts
import { zValidator } from '@hono/zod-validator'
import { z } from 'zod'

app.get(
  '/hello',
  zValidator(
    'query',
    z.object({
      name: z.string(),
    })
  ),
  (c) => {
    const { name } = c.req.valid('query')
    return c.json({
      message: `你好！ ${name}`,
    })
  }
)
```

## 共享类型

要发出端点规范，请导出其类型。

::: warning

为了让 RPC 正确推断路由，所有包含的方法都必须被链接，并且端点或应用程序类型必须从声明的变量中推断。更多信息，请参阅 [RPC 最佳实践](https://hono.dev/docs/guides/best-practices#if-you-want-to-use-rpc-features)。

:::

```ts{1,17}
const route = app.get(
  '/hello',
  zValidator(
    'query',
    z.object({
      name: z.string(),
    })
  ),
  (c) => {
    const { name } = c.req.valid('query')
    return c.json({
      message: `你好！ ${name}`,
    })
  }
)

export type AppType = typeof route
```

## 客户端

接下来。客户端实现。
通过将 `AppType` 类型作为泛型传递给 `hc` 来创建客户端对象。
然后，神奇地，补全功能开始工作，并建议端点路径和请求类型。

![](/images/sc03.gif)

```ts
import { AppType } from './server'
import { hc } from 'hono/client'

const client = hc<AppType>('/api')
const res = await client.hello.$get({
  query: {
    name: 'Hono',
  },
})
```

`Response` 与 fetch API 兼容，但可以使用 `json()` 检索的数据具有类型。

![](/images/sc04.gif)

```ts
const data = await res.json()
console.log(`${data.message}`)
```

共享 API 规范意味着您可以意识到服务器端的更改。

![](/images/ss03.png)

## 与 React 配合使用

您可以使用 React 在 Cloudflare Pages 上创建应用程序。

API 服务器。

```ts
// functions/api/[[route]].ts
import { Hono } from 'hono'
import { handle } from 'hono/cloudflare-pages'
import { z } from 'zod'
import { zValidator } from '@hono/zod-validator'

const app = new Hono()

const schema = z.object({
  id: z.string(),
  title: z.string(),
})

type Todo = z.infer<typeof schema>

const todos: Todo[] = []

const route = app
  .post('/todo', zValidator('form', schema), (c) => {
    const todo = c.req.valid('form')
    todos.push(todo)
    return c.json({
      message: '已创建！',
    })
  })
  .get((c) => {
    return c.json({
      todos,
    })
  })

export type AppType = typeof route

export const onRequest = handle(app, '/api')
```

使用 React 和 React Query 的客户端。

```tsx
// src/App.tsx
import {
  useQuery,
  useMutation,
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'
import { AppType } from '../functions/api/[[route]]'
import { hc, InferResponseType, InferRequestType } from 'hono/client'

const queryClient = new QueryClient()
const client = hc<AppType>('/api')

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Todos />
    </QueryClientProvider>
  )
}

const Todos = () => {
  const query = useQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const res = await client.todo.$get()
      return await res.json()
    },
  })

  const $post = client.todo.$post

  const mutation = useMutation<
    InferResponseType<typeof $post>,
    Error,
    InferRequestType<typeof $post>['form']
  >({
    mutationFn: async (todo) => {
      const res = await $post({
        form: todo,
      })
      return await res.json()
    },
    onSuccess: async () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
    onError: (error) => {
      console.log(error)
    },
  })

  return (
    <div>
      <button
        onClick={() => {
          mutation.mutate({
            id: Date.now().toString(),
            title: '写代码',
          })
        }}
      >
        添加待办事项
      </button>

      <ul>
        {query.data?.todos.map((todo) => (
          <li key={todo.id}>{todo.title}</li>
        ))}
      </ul>
    </div>
  )
}
```
