# Dev 辅助函数

Dev 辅助函数提供了可在开发中使用的有用方法。

```ts
import { Hono } from 'hono'
import { getRouterName, showRoutes } from 'hono/dev'
```

## `getRouterName()`

您可以使用 `getRouterName()` 获取当前使用的路由器的名称。

```ts
const app = new Hono()

// ...

console.log(getRouterName(app))
```

## `showRoutes()`

`showRoutes()` 函数在您的控制台中显示已注册的路由。

考虑如下应用程序：

```ts
const app = new Hono().basePath('/v1')

app.get('/posts', (c) => {
  // ...
})

app.get('/posts/:id', (c) => {
  // ...
})

app.post('/posts', (c) => {
  // ...
})

showRoutes(app, {
  verbose: true,
})
```

当此应用程序开始运行时，路由将如下所示显示在您的控制台中：

```txt
GET   /v1/posts
GET   /v1/posts/:id
POST  /v1/posts
```

## 选项

### <Badge type="info" text="可选" /> verbose: `boolean`

当设置为 `true` 时，它会显示详细信息。

### <Badge type="info" text="可选" /> colorize: `boolean`

当设置为 `false` 时，输出将不会着色。
