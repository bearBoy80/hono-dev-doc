# 漂亮 JSON 中间件

漂亮 JSON 中间件为 JSON 响应正文启用“_JSON 漂亮打印_”。
将 `?pretty` 添加到 url 查询参数，JSON 字符串将被美化。

```js
// GET /
{"project":{"name":"Hono","repository":"https://github.com/honojs/hono"}}
```

将变为：

```js
// GET /?pretty
{
  "project": {
    "name": "Hono",
    "repository": "https://github.com/honojs/hono"
  }
}
```

## 导入

```ts
import { Hono } from 'hono'
import { prettyJSON } from 'hono/pretty-json'
```

## 用法

```ts
const app = new Hono()

app.use(prettyJSON()) // 带选项：prettyJSON({ space: 4 })
app.get('/', (c) => {
  return c.json({ message: 'Hono！' })
})
```

## 选项

### <Badge type="info" text="可选" /> space: `number`

用于缩进的空格数。默认值为 `2`。

### <Badge type="info" text="可选" /> query: `string`

要应用的查询字符串的名称。默认值为 `pretty`。
