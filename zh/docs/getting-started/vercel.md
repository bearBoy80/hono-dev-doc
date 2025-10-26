# Vercel

Vercel 是 AI 云，提供开发人员工具和云基础设施来构建、扩展和保护更快、更个性化的 Web。

Hono 可以零配置部署到 Vercel。

## 1. 设置

Vercel 的入门模板已提供。
使用“create-hono”命令开始您的项目。
为此示例选择 `vercel` 模板。

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

进入 `my-app` 并安装依赖项。

::: code-group

```sh [npm]
cd my-app
npm i
```

```sh [yarn]
cd my-app
yarn
```

```sh [pnpm]
cd my-app
pnpm i
```

```sh [bun]
cd my-app
bun i
```

:::

下一步我们将在本地使用 Vercel CLI 来处理应用程序。如果您还没有安装，请按照 [Vercel CLI 文档](https://vercel.com/docs/cli) 全局安装它。

## 2. Hello World

在您项目的 `index.ts` 或 `src/index.ts` 中，将 Hono 应用程序导出为默认导出。

```ts
import { Hono } from 'hono'

const app = new Hono()

const welcomeStrings = [
  '你好 Hono！',
  '要了解有关 Vercel 上 Hono 的更多信息，请访问 https://vercel.com/docs/frameworks/hono',
]

app.get('/', (c) => {
  return c.text(welcomeStrings.join('\n\n'))
})

export default app
```

如果您是从 `vercel` 模板开始的，那么这已经为您设置好了。

## 3. 运行

要在本地运行开发服务器：

```sh
vercel dev
```

访问 `localhost:3000` 将会得到文本响应。

## 4. 部署

使用 `vc deploy` 部署到 Vercel。

```sh
vercel deploy
```

## 延伸阅读

[在 Vercel 文档中了解有关 Hono 的更多信息](https://vercel.com/docs/frameworks/backend/hono)。
