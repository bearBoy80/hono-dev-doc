# Cloudflare Workers

[Cloudflare Workers](https://workers.cloudflare.com) 是 Cloudflare CDN 上的一个 JavaScript 边缘运行时。

您可以在本地开发应用程序，并使用 [Wrangler](https://developers.cloudflare.com/workers/wrangler/) 通过几个命令发布它。
Wrangler 包含转译编译器，因此我们可以用 TypeScript 编写代码。

让我们用 Hono 为 Cloudflare Workers 制作您的第一个应用程序。

## 1. 设置

有一个适用于 Cloudflare Workers 的入门模板。
使用“create-hono”命令开始您的项目。
本例选择 `cloudflare-workers` 模板。

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

移动到 `my-app` 并安装依赖项。

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

## 2. Hello World

像下面这样编辑 `src/index.ts`。

```ts
import { Hono } from 'hono'
const app = new Hono()

app.get('/', (c) => c.text('你好 Cloudflare Workers！'))

export default app
```

## 3. 运行

在本地运行开发服务器。然后，在您的 Web 浏览器中访问 `http://localhost:8787`。

::: code-group

```sh [npm]
npm run dev
```

```sh [yarn]
yarn dev
```

```sh [pnpm]
pnpm dev
```

```sh [bun]
bun run dev
```

:::

### 更改端口号

如果您需要更改端口号，可以按照此处的说明更新 `wrangler.toml` / `wrangler.json` / `wrangler.jsonc` 文件：
[Wrangler 配置](https://developers.cloudflare.com/workers/wrangler/configuration/#local-development-settings)

或者，您可以按照此处的说明设置 CLI 选项：
[Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/commands/#dev)

## 4. 部署

如果您有 Cloudflare 帐户，可以部署到 Cloudflare。在 `package.json` 中，需要将 `$npm_execpath` 更改为您选择的包管理器。

::: code-group

```sh [npm]
npm run deploy
```

```sh [yarn]
yarn deploy
```

```sh [pnpm]
pnpm run deploy
```

```sh [bun]
bun run deploy
```

:::

就这样！

## 将 Hono 与其他事件处理程序一起使用

您可以在_模块工作者模式_中将 Hono 与其他事件处理程序（例如 `scheduled`）集成。

为此，请将 `app.fetch` 导出为模块的 `fetch` 处理程序，然后根据需要实现其他处理程序：

```ts
const app = new Hono()

export default {
  fetch: app.fetch,
  scheduled: async (batch, env) => {},
}
```

## 提供静态文件

如果您想提供静态文件，可以使用 Cloudflare Workers 的[静态资产功能](https://developers.cloudflare.com/workers/static-assets/)。在 `wrangler.toml` 中指定文件的目录：

```toml
assets = { directory = "public" }
```

然后创建 `public` 目录并将文件放在那里。例如，`./public/static/hello.txt` 将作为 `/static/hello.txt` 提供。

```
.
├── package.json
├── public
│   ├── favicon.ico
│   └── static
│       └── hello.txt
├── src
│   └── index.ts
└── wrangler.toml
```

## 类型

如果您想拥有 workers 类型，则必须安装 `@cloudflare/workers-types`。

::: code-group

```sh [npm]
npm i --save-dev @cloudflare/workers-types
```

```sh [yarn]
yarn add -D @cloudflare/workers-types
```

```sh [pnpm]
pnpm add -D @cloudflare/workers-types
```

```sh [bun]
bun add --dev @cloudflare/workers-types
```

:::

## 测试

对于测试，我们建议使用 `@cloudflare/vitest-pool-workers`。
有关设置，请参阅[示例](https://github.com/honojs/examples)。

如果存在以下应用程序。

```ts
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('请测试我！'))
```

我们可以用这段代码测试它是否返回“_200 OK_”响应。

```ts
describe('测试应用程序', () => {
  it('应返回 200 响应', async () => {
    const res = await app.request('http://localhost/')
    expect(res.status).toBe(200)
  })
})
```

## 绑定

在 Cloudflare Workers 中，我们可以绑定环境变量、KV 命名空间、R2 存储桶或 Durable Object。您可以在 `c.env` 中访问它们。如果您将绑定的“_类型结构_”作为泛型传递给 `Hono`，它将具有类型。

```ts
type Bindings = {
  MY_BUCKET: R2Bucket
  USERNAME: string
  PASSWORD: string
}

const app = new Hono<{ Bindings: Bindings }>()

// 访问环境变量
app.put('/upload/:key', async (c, next) => {
  const key = c.req.param('key')
  await c.env.MY_BUCKET.put(key, c.req.body)
  return c.text(`成功放置 ${key}！`)
})
```

## 在中间件中使用变量

这仅适用于模块工作者模式。
如果您想在中间件中使用变量或秘密变量，例如，在基本身份验证中间件中使用“username”或“password”，则需要像下面这样编写。

```ts
import { basicAuth } from 'hono/basic-auth'

type Bindings = {
  USERNAME: string
  PASSWORD: string
}

const app = new Hono<{ Bindings: Bindings }>()

//...

app.use('/auth/*', async (c, next) => {
  const auth = basicAuth({
    username: c.env.USERNAME,
    password: c.env.PASSWORD,
  })
  return auth(c, next)
})
```

这同样适用于承载身份验证中间件、JWT 身份验证或其他。

## 从 GitHub Actions 部署

通过 CI 将代码部署到 Cloudflare 之前，您需要一个 Cloudflare 令牌。您可以从[用户 API 令牌](https://dash.cloudflare.com/profile/api-tokens)管理它。

如果是新创建的令牌，请选择**编辑 Cloudflare Workers**模板，如果您已经有另一个令牌，请确保该令牌具有相应的权限（不，令牌权限在 Cloudflare Pages 和 Cloudflare Workers 之间不共享）。

然后转到您的 GitHub 存储库设置仪表板：`设置->秘密和变量->操作->存储库秘密`，并添加一个名为 `CLOUDFLARE_API_TOKEN` 的新秘密。

然后在您的 Hono 项目根文件夹中创建 `.github/workflows/deploy.yml`，粘贴以下代码：

```yml
name: 部署

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: 部署
    steps:
      - uses: actions/checkout@v4
      - name: 部署
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

然后编辑 `wrangler.toml`，并在 `compatibility_date` 行后添加此代码。

```toml
main = "src/index.ts"
minify = true
```

一切准备就绪！现在推送代码并享受它吧。

## 本地开发时加载 env

要为本地开发配置环境变量，请在项目的根目录中创建一个 `.dev.vars` 文件或一个 `.env` 文件。
这些文件应使用 [dotenv](https://hexdocs.pm/dotenvy/dotenv-file-format.html) 语法进行格式化。例如：

```
SECRET_KEY=value
API_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

> 有关本节的更多信息，您可以在 Cloudflare 文档中找到：
> https://developers.cloudflare.com/workers/wrangler/configuration/#secrets

然后我们在代码中使用 `c.env.*` 来获取环境变量。

::: info
默认情况下，`process.env` 在 Cloudflare Workers 中不可用，因此建议从 `c.env` 获取环境变量。如果您想使用它，则需要启用 [`nodejs_compat_populate_process_env`](https://developers.cloudflare.com/workers/configuration/compatibility-flags/#enable-auto-populating-processenv) 标志。您还可以从 `cloudflare:workers` 导入 `env`。有关详细信息，请参阅[如何在 Cloudflare 文档上访问 `env`](https://developers.cloudflare.com/workers/runtime-apis/bindings/#how-to-access-env)
:::

```ts
type Bindings = {
  SECRET_KEY: string
}

const app = new Hono<{ Bindings: Bindings }>()

app.get('/env', (c) => {
  const SECRET_KEY = c.env.SECRET_KEY
  return c.text(SECRET_KEY)
})
```

在将项目部署到 Cloudflare 之前，请记住在 Cloudflare Workers 项目的配置中设置环境变量/秘密。

> 有关本节的更多信息，您可以在 Cloudflare 文档中找到：
> https://developers.cloudflare.com/workers/configuration/environment-variables/#add-environment-variables-via-the-dashboard
