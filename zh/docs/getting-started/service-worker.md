# Service Worker

[Service Worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) 是一个在浏览器后台运行的脚本，用于处理缓存和推送通知等任务。使用 Service Worker 适配器，您可以在浏览器中将使用 Hono 制作的应用程序作为 [FetchEvent](https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent) 处理程序运行。

本页展示了使用 [Vite](https://vitejs.dev/) 创建项目的示例。

## 1. 设置

首先，创建并进入您的项目目录：

```sh
mkdir my-app
cd my-app
```

为项目创建必要的文件。创建一个包含以下内容的 `package.json` 文件：

```json
{
  "name": "my-app",
  "private": true,
  "scripts": {
    "dev": "vite dev"
  },
  "type": "module"
}
```

同样，创建一个包含以下内容的 `tsconfig.json` 文件：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "WebWorker"],
    "moduleResolution": "bundler"
  },
  "include": ["./"],
  "exclude": ["node_modules"]
}
```

接下来，安装必要的模块。

::: code-group

```sh [npm]
npm i hono
npm i -D vite
```

```sh [yarn]
yarn add hono
yarn add -D vite
```

```sh [pnpm]
pnpm add hono
pnpm add -D vite
```

```sh [bun]
bun add hono
bun add -D vite
```

:::

## 2. Hello World

编辑 `index.html`：

```html
<!doctype html>
<html>
  <body>
    <a href="/sw">Service Worker 的 Hello World</a>
    <script type="module" src="/main.ts"></script>
  </body>
</html>
```

`main.ts` 是一个用于注册 Service Worker 的脚本：

```ts
function register() {
  navigator.serviceWorker
    .register('/sw.ts', { scope: '/sw', type: 'module' })
    .then(
      function (_registration) {
        console.log('注册 Service Worker：成功')
      },
      function (_error) {
        console.log('注册 Service Worker：错误')
      }
    )
}
function start() {
  navigator.serviceWorker
    .getRegistrations()
    .then(function (registrations) {
      for (const registration of registrations) {
        console.log('注销 Service Worker')
        registration.unregister()
      }
      register()
    })
}
start()
```

在 `sw.ts` 中，使用 Hono 创建一个应用程序，并使用 Service Worker 适配器的 `handle` 函数将其注册到 `fetch` 事件。这允许 Hono 应用程序拦截对 `/sw` 的访问。

```ts
// 支持类型
// https://github.com/microsoft/TypeScript/issues/14877
declare const self: ServiceWorkerGlobalScope

import { Hono } from 'hono'
import { handle } from 'hono/service-worker'

const app = new Hono().basePath('/sw')
app.get('/', (c) => c.text('Hello World'))

self.addEventListener('fetch', handle(app))
```

### 使用 `fire()`

`fire()` 函数会自动为您调用 `addEventListener('fetch', handle(app))`，使代码更简洁。

```ts
import { Hono } from 'hono'
import { fire } from 'hono/service-worker'

const app = new Hono().basePath('/sw')
app.get('/', (c) => c.text('Hello World'))

fire(app)
```

## 3. 运行

启动开发服务器。

::: code-group

```sh [npm]
npm run dev
```

```sh [yarn]
yarn dev
```

```sh [pnpm]
pnpm run dev
```

```sh [bun]
bun run dev
```

:::

默认情况下，开发服务器将在端口 `5173` 上运行。在您的浏览器中访问 `http://localhost:5173/` 以完成 Service Worker 注册。然后，访问 `/sw` 以查看 Hono 应用程序的响应。
