# 路由

Hono 的路由灵活直观。
我们来看看。

## 基础

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// HTTP 方法
app.get('/', (c) => c.text('GET /'))
app.post('/', (c) => c.text('POST /'))
app.put('/', (c) => c.text('PUT /'))
app.delete('/', (c) => c.text('DELETE /'))

// 通配符
app.get('/wild/*/card', (c) => {
  return c.text('GET /wild/*/card')
})

// 任意 HTTP 方法
app.all('/hello', (c) => c.text('任意方法 /hello'))

// 自定义 HTTP 方法
app.on('PURGE', '/cache', (c) => c.text('PURGE 方法 /cache'))

// 多种方法
app.on(['PUT', 'DELETE'], '/post', (c) =>
  c.text('PUT 或 DELETE /post')
)

// 多个路径
app.on('GET', ['/hello', '/ja/hello', '/en/hello'], (c) =>
  c.text('你好')
)
```

## 路径参数

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/user/:name', async (c) => {
  const name = c.req.param('name')
  //       ^?
  // ...
})
```

或一次性获取所有参数：

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/posts/:id/comment/:comment_id', async (c) => {
  const { id, comment_id } = c.req.param()
  //       ^?
  // ...
})
```

## 可选参数

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// 将匹配 `/api/animal` 和 `/api/animal/:type`
app.get('/api/animal/:type?', (c) => c.text('动物！'))
```

## 正则表达式

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/post/:date{[0-9]+}/:title{[a-z]+}', async (c) => {
  const { date, title } = c.req.param()
  //       ^?
  // ...
})
```

## 包含斜杠

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/posts/:filename{.+\\.png}', async (c) => {
  //...
})
```

## 链式路由

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app
  .get('/endpoint', (c) => {
    return c.text('GET /endpoint')
  })
  .post((c) => {
    return c.text('POST /endpoint')
  })
  .delete((c) => {
    return c.text('DELETE /endpoint')
  })
```

## 分组

您可以使用 Hono 实例对路由进行分组，并使用路由方法将它们添加到主应用中。

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const book = new Hono()

book.get('/', (c) => c.text('图书列表')) // GET /book
book.get('/:id', (c) => {
  // GET /book/:id
  const id = c.req.param('id')
  return c.text('获取图书：' + id)
})
book.post('/', (c) => c.text('创建图书')) // POST /book

const app = new Hono()
app.route('/book', book)
```

## 不改变基础路径的分组

您也可以在保持基础路径的同时对多个实例进行分组。

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const book = new Hono()
book.get('/book', (c) => c.text('图书列表')) // GET /book
book.post('/book', (c) => c.text('创建图书')) // POST /book

const user = new Hono().basePath('/user')
user.get('/', (c) => c.text('用户列表')) // GET /user
user.post('/', (c) => c.text('创建用户')) // POST /user

const app = new Hono()
app.route('/', book) // 处理 /book
app.route('/', user) // 处理 /user
```

## 基础路径

您可以指定基础路径。

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const api = new Hono().basePath('/api')
api.get('/book', (c) => c.text('图书列表')) // GET /api/book
```

## 带主机名的路由

如果包含主机名，它也能正常工作。

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const app = new Hono({
  getPath: (req) => req.url.replace(/^https?:\/([^?]+).*$/, '$1'),
})

app.get('/www1.example.com/hello', (c) => c.text('你好 www1'))
app.get('/www2.example.com/hello', (c) => c.text('你好 www2'))
```

## 带 `host` 标头值的路由

如果您在 Hono 构造函数中设置了 `getPath()` 函数，Hono 可以处理 `host` 标头值。

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const app = new Hono({
  getPath: (req) =>
    '/' +
    req.headers.get('host') +
    req.url.replace(/^https?:\/\/[^/]+(\/[^?]*).*/, '$1'),
})

app.get('/www1.example.com/hello', (c) => c.text('你好 www1'))

// 以下请求将匹配该路由：
// new Request('http://www1.example.com/hello', {
//  headers: { host: 'www1.example.com' },
// })
```

通过应用此方法，例如，您可以根据 `User-Agent` 标头更改路由。

## 路由优先级

处理程序或中间件将按注册顺序执行。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/book/a', (c) => c.text('a')) // a
app.get('/book/:slug', (c) => c.text('通用')) // 通用
```

```
GET /book/a ---> `a`
GET /book/b ---> `通用`
```

当执行处理程序时，该过程将停止。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('*', (c) => c.text('通用')) // 通用
app.get('/foo', (c) => c.text('foo')) // foo
```

```
GET /foo ---> `通用` // foo 将不会被分发
```

如果您有要执行的中间件，请将代码写在处理程序之上。

```ts twoslash
import { Hono } from 'hono'
import { logger } from 'hono/logger'
const app = new Hono()
// ---cut---
app.use(logger())
app.get('/foo', (c) => c.text('foo'))
```

如果您想有一个“_回退_”处理程序，请将代码写在其他处理程序之下。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/bar', (c) => c.text('bar')) // bar
app.get('*', (c) => c.text('回退')) // 回退
```

```
GET /bar ---> `bar`
GET /foo ---> `回退`
```

## 分组顺序

请注意，分组路由的错误很难被发现。
`route()` 函数从第二个参数（例如 `three` 或 `two`）中获取存储的路由，并将其添加到自己的（`two` 或 `app`）路由中。

```ts
three.get('/hi', (c) => c.text('hi'))
two.route('/three', three)
app.route('/two', two)

export default app
```

它将返回 200 响应。

```
GET /two/three/hi ---> `hi`
```

但是，如果它们的顺序错误，它将返回 404。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
const two = new Hono()
const three = new Hono()
// ---cut---
three.get('/hi', (c) => c.text('hi'))
app.route('/two', two) // `two` 没有路由
two.route('/three', three)

export default app
```

```
GET /two/three/hi ---> 404 未找到
```
