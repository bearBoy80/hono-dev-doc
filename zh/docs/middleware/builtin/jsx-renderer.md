# JSX 渲染器中间件

JSX 渲染器中间件允许您在使用 `c.render()` 函数呈现 JSX 时设置布局，而无需使用 `c.setRenderer()`。此外，它还允许通过使用 `useRequestContext()` 在组件内访问 Context 的实例。

## 导入

```ts
import { Hono } from 'hono'
import { jsxRenderer, useRequestContext } from 'hono/jsx-renderer'
```

## 用法

```jsx
const app = new Hono()

app.get(
  '/page/*',
  jsxRenderer(({ children }) => {
    return (
      <html>
        <body>
          <header>菜单</header>
          <div>{children}</div>
        </body>
      </html>
    )
  })
)

app.get('/page/about', (c) => {
  return c.render(<h1>关于我！</h1>)
})
```

## 选项

### <Badge type="info" text="可选" /> docType: `boolean` | `string`

如果您不想在 HTML 的开头添加 DOCTYPE，请将 `docType` 选项设置为 `false`。

```tsx
app.use(
  '*',
  jsxRenderer(
    ({ children }) => {
      return (
        <html>
          <body>{children}</body>
        </html>
      )
    },
    { docType: false }
  )
)
```

您还可以指定 DOCTYPE。

```tsx
app.use(
  '*',
  jsxRenderer(
    ({ children }) => {
      return (
        <html>
          <body>{children}</body>
        </html>
      )
    },
    {
      docType:
        '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">',
    }
  )
)
```

### <Badge type="info" text="可选" /> stream: `boolean` | `Record<string, string>`

如果将其设置为 `true` 或提供 Record 值，它将呈现为流式响应。

```tsx
const AsyncComponent = async () => {
  await new Promise((r) => setTimeout(r, 1000)) // 休眠 1 秒
  return <div>嗨！</div>
}

app.get(
  '*',
  jsxRenderer(
    ({ children }) => {
      return (
        <html>
          <body>
            <h1>SSR 流式传输</h1>
            {children}
          </body>
        </html>
      )
    },
    { stream: true }
  )
)

app.get('/', (c) => {
  return c.render(
    <Suspense fallback={<div>加载中...</div>}>
      <AsyncComponent />
    </Suspense>
  )
})
```

如果设置为 `true`，则会添加以下标头：

```ts
{
  'Transfer-Encoding': 'chunked',
  'Content-Type': 'text/html; charset=UTF-8',
  'Content-Encoding': 'Identity'
}
```

您可以通过指定 Record 值来自定义标头值。

## 嵌套布局

`Layout` 组件支持嵌套布局。

```tsx
app.use(
  jsxRenderer(({ children }) => {
    return (
      <html>
        <body>{children}</body>
      </html>
    )
  })
)

const blog = new Hono()
blog.use(
  jsxRenderer(({ children, Layout }) => {
    return (
      <Layout>
        <nav>博客菜单</nav>
        <div>{children}</div>
      </Layout>
    )
  })
)

app.route('/blog', blog)
```

## `useRequestContext()`

`useRequestContext()` 返回 Context 的实例。

```tsx
import { useRequestContext, jsxRenderer } from 'hono/jsx-renderer'

const app = new Hono()
app.use(jsxRenderer())

const RequestUrlBadge: FC = () => {
  const c = useRequestContext()
  return <b>{c.req.url}</b>
}

app.get('/page/info', (c) => {
  return c.render(
    <div>
      您正在访问：<RequestUrlBadge />
    </div>
  )
})
```

::: warning
您不能将 `useRequestContext()` 与 Deno 的 `precompile` JSX 选项一起使用。请使用 `react-jsx`：

```json
   "compilerOptions": {
     "jsx": "precompile", // [!code --]
     "jsx": "react-jsx", // [!code ++]
     "jsxImportSource": "hono/jsx"
   }
 }
```

:::

## 扩展 `ContextRenderer`

通过如下所示定义 `ContextRenderer`，您可以将其他内容传递给渲染器。例如，当您想根据页面更改 head 标签的内容时，这非常方便。

```tsx
declare module 'hono' {
  interface ContextRenderer {
    (
      content: string | Promise<string>,
      props: { title: string }
    ): Response
  }
}

const app = new Hono()

app.get(
  '/page/*',
  jsxRenderer(({ children, title }) => {
    return (
      <html>
        <head>
          <title>{title}</title>
        </head>
        <body>
          <header>菜单</header>
          <div>{children}</div>
        </body>
      </html>
    )
  })
)

app.get('/page/favorites', (c) => {
  return c.render(
    <div>
      <ul>
        <li>吃寿司</li>
        <li>看棒球比赛</li>
      </ul>
    </div>,
    {
      title: '我的最爱',
    }
  )
})
```
