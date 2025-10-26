# 方法覆盖中间件

此中间件根据表单、标头或查询的值，执行与请求的实际方法不同的指定方法的处理程序，并返回其响应。

## 导入

```ts
import { Hono } from 'hono'
import { methodOverride } from 'hono/method-override'
```

## 用法

```ts
const app = new Hono()

// 如果未指定任何选项，则表单中 `_method` 的值（例如 DELETE）将用作方法。
app.use('/posts', methodOverride({ app }))

app.delete('/posts', (c) => {
  // ....
})
```

## 例如

由于 HTML 表单无法发送 DELETE 方法，您可以将值 `DELETE` 放入名为 `_method` 的属性中并发送它。然后将执行 `app.delete()` 的处理程序。

HTML 表单：

```html
<form action="/posts" method="POST">
  <input type="hidden" name="_method" value="DELETE" />
  <input type="text" name="id" />
</form>
```

应用程序：

```ts
import { methodOverride } from 'hono/method-override'

const app = new Hono()
app.use('/posts', methodOverride({ app }))

app.delete('/posts', () => {
  // ...
})
```

您可以更改默认值或使用标头值和查询值：

```ts
app.use('/posts', methodOverride({ app, form: '_custom_name' }))
app.use(
  '/posts',
  methodOverride({ app, header: 'X-METHOD-OVERRIDE' })
)
app.use('/posts', methodOverride({ app, query: '_method' }))
```

## 选项

### <Badge type="danger" text="必需" /> app: `Hono`

您的应用程序中使用的 `Hono` 实例。

### <Badge type="info" text="可选" /> form: `string`

包含方法名称的值的表单键。
默认值为 `_method`。

### <Badge type="info" text="可选" /> header: `boolean`

包含方法名称的值的标头名称。

### <Badge type="info" text="可选" /> query: `boolean`

包含方法名称的值的查询参数键。
