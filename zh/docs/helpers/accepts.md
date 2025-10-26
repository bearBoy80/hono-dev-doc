# Accepts 辅助函数

Accepts 辅助函数有助于处理请求中的 Accept 标头。

## 导入

```ts
import { Hono } from 'hono'
import { accepts } from 'hono/accepts'
```

## `accepts()`

`accepts()` 函数会查看 Accept 标头，例如 Accept-Encoding 和 Accept-Language，并返回适当的值。

```ts
import { accepts } from 'hono/accepts'

app.get('/', (c) => {
  const accept = accepts(c, {
    header: 'Accept-Language',
    supports: ['en', 'ja', 'zh'],
    default: 'en',
  })
  return c.json({ lang: accept })
})
```

### `AcceptHeader` 类型

`AcceptHeader` 类型的定义如下。

```ts
export type AcceptHeader =
  | 'Accept'
  | 'Accept-Charset'
  | 'Accept-Encoding'
  | 'Accept-Language'
  | 'Accept-Patch'
  | 'Accept-Post'
  | 'Accept-Ranges'
```

## 选项

### <Badge type="danger" text="必需" /> header: `AcceptHeader`

目标 accept 标头。

### <Badge type="danger" text="必需" /> supports: `string[]`

您的应用程序支持的标头值。

### <Badge type="danger" text="必需" /> default: `string`

默认值。

### <Badge type="info" text="可选" /> match: `(accepts: Accept[], config: acceptsConfig) => string`

自定义匹配函数。
