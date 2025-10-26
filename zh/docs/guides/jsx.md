# JSX

您可以使用 `hono/jsx` 通过 JSX 语法编写 HTML。

虽然 `hono/jsx` 可以在客户端运行，但您最常使用它来在服务器端呈现内容。以下是一些与服务器端和客户端通用的 JSX 相关内容。

## 设置

要使用 JSX，请修改 `tsconfig.json`：

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx"
  }
}
```

或者，使用 pragma 指令：

```ts
/** @jsx jsx */
/** @jsxImportSource hono/jsx */
```

对于 Deno，您必须修改 `deno.json` 而不是 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "jsx": "precompile",
    "jsxImportSource": "hono/jsx"
  }
}
```

## 用法

:::info
如果您直接从[快速入门](/docs/#quick-start)开始，主文件的扩展名为 `.ts` - 您需要将其更改为 `.tsx` - 否则您将根本无法运行该应用程序。您还应修改 `package.json`（或如果您使用 Deno，则为 `deno.json`）以反映该更改（例如，dev 脚本中不应有 `bun run --hot src/index.ts`，而应有 `bun run --hot src/index.tsx`）。
:::

`index.tsx`:

```tsx
import { Hono } from 'hono'
import type { FC } from 'hono/jsx'

const app = new Hono()

const Layout: FC = (props) => {
  return (
    <html>
      <body>{props.children}</body>
    </html>
  )
}

const Top: FC<{ messages: string[] }> = (props: {
  messages: string[]
}) => {
  return (
    <Layout>
      <h1>你好 Hono！</h1>
      <ul>
        {props.messages.map((message) => {
          return <li>{message}!!</li>
        })}
      </ul>
    </Layout>
  )
}

app.get('/', (c) => {
  const messages = ['早上好', '晚上好', '晚安']
  return c.html(<Top messages={messages} />)
})

export default app
```

## 元数据提升

您可以直接在组件内部编写文档元数据标签，例如 `<title>`、`<link>` 和 `<meta>`。这些标签将自动提升到文档的 `<head>` 部分。当 `<head>` 元素远离确定适当元数据的组件呈现时，这尤其有用。

```tsx
import { Hono } from 'hono'

const app = new Hono()

app.use('*', async (c, next) => {
  c.setRenderer((content) => {
    return c.html(
      <html>
        <head></head>
        <body>{content}</body>
      </html>
    )
  })
  await next()
})

app.get('/about', (c) => {
  return c.render(
    <>
      <title>关于页面</title>
      <meta name='description' content='这是关于页面。' />
      关于页面内容
    </>
  )
})

export default app
```

## Fragment

使用 Fragment 对多个元素进行分组，而无需添加额外的节点：

```tsx
import { Fragment } from 'hono/jsx'

const List = () => (
  <Fragment>
    <p>第一个子元素</p>
    <p>第二个子元素</p>
    <p>第三个子元素</p>
  </Fragment>
)
```

或者，如果设置正确，您可以使用 `<></>` 来编写它。

```tsx
const List = () => (
  <>
    <p>第一个子元素</p>
    <p>第二个子元素</p>
    <p>第三个子元素</p>
  </>
)
```

## `PropsWithChildren`

您可以使用 `PropsWithChildren` 在函数组件中正确推断子元素。

```tsx
import { PropsWithChildren } from 'hono/jsx'

type Post = {
  id: number
  title: string
}

function Component({ title, children }: PropsWithChildren<Post>) {
  return (
    <div>
      <h1>{title}</h1>
      {children}
    </div>
  )
}
```

## 插入原始 HTML

要直接插入 HTML，请使用 `dangerouslySetInnerHTML`：

```tsx
app.get('/foo', (c) => {
  const inner = { __html: 'JSX &middot; SSR' }
  const Div = <div dangerouslySetInnerHTML={inner} />
})
```

## 记忆化

通过使用 `memo` 记忆化计算出的字符串来优化您的组件：

```tsx
import { memo } from 'hono/jsx'

const Header = memo(() => <header>欢迎来到 Hono</header>)
const Footer = memo(() => <footer>由 Hono 提供支持</footer>)
const Layout = (
  <div>
    <Header />
    <p>Hono 很酷！</p>
    <Footer />
  </div>
)
```

## 上下文

通过使用 `useContext`，您可以在组件树的任何级别全局共享数据，而无需通过 props 传递值。

```tsx
import type { FC } from 'hono/jsx'
import { createContext, useContext } from 'hono/jsx'

const themes = {
  light: {
    color: '#000000',
    background: '#eeeeee',
  },
  dark: {
    color: '#ffffff',
    background: '#222222',
  },
}

const ThemeContext = createContext(themes.light)

const Button: FC = () => {
  const theme = useContext(ThemeContext)
  return <button style={theme}>按下！</button>
}

const Toolbar: FC = () => {
  return (
    <div>
      <Button />
    </div>
  )
}

// ...

app.get('/', (c) => {
  return c.html(
    <div>
      <ThemeContext.Provider value={themes.dark}>
        <Toolbar />
      </ThemeContext.Provider>
    </div>
  )
})
```

## 异步组件

`hono/jsx` 支持异步组件，因此您可以在组件中使用 `async`/`await`。
如果使用 `c.html()` 呈现它，它将自动等待。

```tsx
const AsyncComponent = async () => {
  await new Promise((r) => setTimeout(r, 1000)) // 休眠 1 秒
  return <div>完成！</div>
}

app.get('/', (c) => {
  return c.html(
    <html>
      <body>
        <AsyncComponent />
      </body>
    </html>
  )
})
```

## Suspense <Badge style="vertical-align: middle;" type="warning" text="实验性" />

类似 React 的 `Suspense` 功能可用。
如果使用 `Suspense` 包装异步组件，则会首先呈现 fallback 中的内容，一旦 Promise 被解析，就会显示等待的内容。
您可以将其与 `renderToReadableStream()` 一起使用。

```tsx
import { renderToReadableStream, Suspense } from 'hono/jsx/streaming'

//...

app.get('/', (c) => {
  const stream = renderToReadableStream(
    <html>
      <body>
        <Suspense fallback={<div>加载中...</div>}>
          <Component />
        </Suspense>
      </body>
    </html>
  )
  return c.body(stream, {
    headers: {
      'Content-Type': 'text/html; charset=UTF-8',
      'Transfer-Encoding': 'chunked',
    },
  })
})
```

## ErrorBoundary <Badge style="vertical-align: middle;" type="warning" text="实验性" />

您可以使用 `ErrorBoundary` 捕获子组件中的错误。

在下面的示例中，如果发生错误，它将显示 `fallback` 中指定的内容。

```tsx
function SyncComponent() {
  throw new Error('错误')
  return <div>你好</div>
}

app.get('/sync', async (c) => {
  return c.html(
    <html>
      <body>
        <ErrorBoundary fallback={<div>服务不可用</div>}>
          <SyncComponent />
        </ErrorBoundary>
      </body>
    </html>
  )
})
```

`ErrorBoundary` 也可以与异步组件和 `Suspense` 一起使用。

```tsx
async function AsyncComponent() {
  await new Promise((resolve) => setTimeout(resolve, 2000))
  throw new Error('错误')
  return <div>你好</div>
}

app.get('/with-suspense', async (c) => {
  return c.html(
    <html>
      <body>
        <ErrorBoundary fallback={<div>服务不可用</div>}>
          <Suspense fallback={<div>加载中...</div>}>
            <AsyncComponent />
          </Suspense>
        </ErrorBoundary>
      </body>
    </html>
  )
})
```

## StreamingContext <Badge style="vertical-align: middle;" type="warning" text="实验性" />

您可以使用 `StreamingContext` 为 `Suspense` 和 `ErrorBoundary` 等流式组件提供配置。这对于将 nonce 值添加到这些组件生成的脚本标签以实现内容安全策略 (CSP) 非常有用。

```tsx
import { Suspense, StreamingContext } from 'hono/jsx/streaming'

// ...

app.get('/', (c) => {
  const stream = renderToReadableStream(
    <html>
      <body>
        <StreamingContext
          value={{ scriptNonce: 'random-nonce-value' }}
        >
          <Suspense fallback={<div>加载中...</div>}>
            <AsyncComponent />
          </Suspense>
        </StreamingContext>
      </body>
    </html>
  )

  return c.body(stream, {
    headers: {
      'Content-Type': 'text/html; charset=UTF-8',
      'Transfer-Encoding': 'chunked',
      'Content-Security-Policy':
        "script-src 'nonce-random-nonce-value'",
    },
  })
})
```

`scriptNonce` 值将自动添加到由 `Suspense` 和 `ErrorBoundary` 组件生成的任何 `<script>` 标签中。

## 与 html 中间件集成

结合 JSX 和 html 中间件以实现强大的模板化。
有关深入的详细信息，请查阅 [html 中间件文档](/docs/helpers/html)。

```tsx
import { Hono } from 'hono'
import { html } from 'hono/html'

const app = new Hono()

interface SiteData {
  title: string
  children?: any
}

const Layout = (props: SiteData) =>
  html`<!doctype html>
    <html>
      <head>
        <title>${props.title}</title>
      </head>
      <body>
        ${props.children}
      </body>
    </html>`

const Content = (props: { siteData: SiteData; name: string }) => (
  <Layout {...props.siteData}>
    <h1>你好 {props.name}</h1>
  </Layout>
)

app.get('/:name', (c) => {
  const { name } = c.req.param()
  const props = {
    name: name,
    siteData: {
      title: '带 html 示例的 JSX',
    },
  }
  return c.html(<Content {...props} />)
})

export default app
```

## 与 JSX 渲染器中间件一起使用

[JSX 渲染器中间件](/docs/middleware/builtin/jsx-renderer) 允许您更轻松地使用 JSX 创建 HTML 页面。

## 覆盖类型定义

您可以覆盖类型定义以添加您的自定义元素和属性。

```ts
declare module 'hono/jsx' {
  namespace JSX {
    interface IntrinsicElements {
      'my-custom-element': HTMLAttributes & {
        'x-event'?: 'click' | 'scroll'
      }
    }
  }
}
```
