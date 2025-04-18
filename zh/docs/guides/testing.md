---
title: 测试
description: 测试 Hono 应用程序
---

# 测试

[Vitest]: https://vitest.dev/

测试很重要。
实际上，测试 Hono 应用程序非常简单。
虽然创建测试环境的方式因运行时而异，但基本步骤是相同的。
在本节中，我们将使用 Cloudflare Workers 和 [Vitest] 进行测试。

::: tip
Cloudflare 推荐使用 [Vitest] 和 [@cloudflare/vitest-pool-workers](https://www.npmjs.com/package/@cloudflare/vitest-pool-workers)。更多详情，请参考 Cloudflare Workers 文档中的 [Vitest 集成](https://developers.cloudflare.com/workers/testing/vitest-integration/)。
:::

## 请求和响应

你只需要创建一个请求并将其传递给 Hono 应用程序来验证响应。
另外，你可以使用 `app.request` 这个实用的方法。

::: tip
关于类型化的测试客户端，请参考[测试辅助工具](/zh/docs/helpers/testing)。
:::

例如，考虑一个提供以下 REST API 的应用程序。

```ts
app.get('/posts', (c) => {
  return c.text('Many posts')
})

app.post('/posts', (c) => {
  return c.json(
    {
      message: 'Created',
    },
    201,
    {
      'X-Custom': 'Thank you',
    }
  )
})
```

向 `GET /posts` 发送请求并测试响应。

```ts
describe('示例', () => {
  test('GET /posts', async () => {
    const res = await app.request('/posts')
    expect(res.status).toBe(200)
    expect(await res.text()).toBe('Many posts')
  })
})
```

要向 `POST /posts` 发送请求，可以这样做：

```ts
test('POST /posts', async () => {
  const res = await app.request('/posts', {
    method: 'POST',
  })
  expect(res.status).toBe(201)
  expect(res.headers.get('X-Custom')).toBe('Thank you')
  expect(await res.json()).toEqual({
    message: 'Created',
  })
})
```

要向 `POST /posts` 发送带有 `JSON` 数据的请求，可以这样做：

```ts
test('POST /posts', async () => {
  const res = await app.request('/posts', {
    method: 'POST',
    body: JSON.stringify({ message: 'hello hono' }),
    headers: new Headers({ 'Content-Type': 'application/json' }),
  })
  expect(res.status).toBe(201)
  expect(res.headers.get('X-Custom')).toBe('Thank you')
  expect(await res.json()).toEqual({
    message: 'Created',
  })
})
```

要向 `POST /posts` 发送带有 `multipart/form-data` 数据的请求，可以这样做：

```ts
test('POST /posts', async () => {
  const formData = new FormData()
  formData.append('message', 'hello')
  const res = await app.request('/posts', {
    method: 'POST',
    body: formData,
  })
  expect(res.status).toBe(201)
  expect(res.headers.get('X-Custom')).toBe('Thank you')
  expect(await res.json()).toEqual({
    message: 'Created',
  })
})
```

你也可以传递一个 Request 类的实例：

```ts
test('POST /posts', async () => {
  const req = new Request('http://localhost/posts', {
    method: 'POST',
  })
  const res = await app.request(req)
  expect(res.status).toBe(201)
  expect(res.headers.get('X-Custom')).toBe('Thank you')
  expect(await res.json()).toEqual({
    message: 'Created',
  })
})
```

通过这种方式，你可以进行类似端到端的测试。

## 环境变量

要在测试中设置 `c.env`，你可以将其作为第三个参数传递给 `app.request`。这对于模拟 [Cloudflare Workers Bindings](https://hono.dev/getting-started/cloudflare-workers#bindings) 等值很有用：

```ts
const MOCK_ENV = {
  API_HOST: 'example.com',
  DB: {
    prepare: () => {
      /* 模拟 D1 */
    },
  },
}

test('GET /posts', async () => {
  const res = await app.request('/posts', {}, MOCK_ENV)
})
```
