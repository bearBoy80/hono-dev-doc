# Netlify

Netlify 提供静态站点托管和无服务器后端服务。[边缘函数](https://docs.netlify.com/edge-functions/overview/) 使我们能够使网页动态化。

边缘函数支持使用 Deno 和 TypeScript 编写，并且通过 [Netlify CLI](https://docs.netlify.com/cli/get-started/) 可以轻松部署。使用 Hono，您可以为 Netlify 边缘函数创建应用程序。

## 1. 设置

有一个适用于 Netlify 的入门模板。
使用“create-hono”命令开始您的项目。
本例选择 `netlify` 模板。

::: code-group

```sh [npm]
npm create hono@latest my-app
```

```sh [yarn]
yarn create hono my-app
```

```sh [pnpm]
pnpm create hono my-app
```

```sh [bun]
bun create hono@latest my-app
```

```sh [deno]
deno init --npm hono my-app
```

:::

进入 `my-app`。

## 2. Hello World

编辑 `netlify/edge-functions/index.ts`：

```ts
import { Hono } from 'jsr:@hono/hono'
import { handle } from 'jsr:@hono/hono/netlify'

const app = new Hono()

app.get('/', (c) => {
  return c.text('你好 Hono！')
})

export default handle(app)
```

## 3. 运行

使用 Netlify CLI 运行开发服务器。然后，在您的 Web 浏览器中访问 `http://localhost:8888`。

```sh
netlify dev
```

## 4. 部署

您可以使用 `netlify deploy` 命令进行部署。

```sh
netlify deploy --prod
```

## `Context`

您可以通过 `c.env` 访问 Netlify 的 `Context`：

```ts
import { Hono } from 'jsr:@hono/hono'
import { handle } from 'jsr:@hono/hono/netlify'

// 导入类型定义
import type { Context } from 'https://edge.netlify.com/'

export type Env = {
  Bindings: {
    context: Context
  }
}

const app = new Hono<Env>()

app.get('/country', (c) =>
  c.json({
    '您在': c.env.context.geo.country?.name,
  })
)

export default handle(app)
```
