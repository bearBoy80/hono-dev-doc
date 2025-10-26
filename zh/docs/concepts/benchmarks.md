# 基准测试

基准测试只是基准测试，但它们对我们很重要。

## 路由器

我们测量了许多 JavaScript 路由器的速度。
例如，`find-my-way` 是 Fastify 内部使用的一个非常快的路由器。

- @medley/router
- find-my-way
- koa-tree-router
- trek-router
- express (包括处理)
- koa-router

首先，我们为每个路由器注册了以下路由。
这些路由与现实世界中使用的相似。

```ts twoslash
interface Route {
  method: string
  path: string
}
// ---cut---
export const routes: Route[] = [
  { method: 'GET', path: '/user' },
  { method: 'GET', path: '/user/comments' },
  { method: 'GET', path: '/user/avatar' },
  { method: 'GET', path: '/user/lookup/username/:username' },
  { method: 'GET', path: '/user/lookup/email/:address' },
  { method: 'GET', path: '/event/:id' },
  { method: 'GET', path: '/event/:id/comments' },
  { method: 'POST', path: '/event/:id/comment' },
  { method: 'GET', path: '/map/:location/events' },
  { method: 'GET', path: '/status' },
  { method: 'GET', path: '/very/deeply/nested/route/hello/there' },
  { method: 'GET', path: '/static/*' },
]
```

然后我们向如下端点发送请求。

```ts twoslash
interface Route {
  method: string
  path: string
}
// ---cut---
const routes: (Route & { name: string })[] = [
  {
    name: 'short static',
    method: 'GET',
    path: '/user',
  },
  {
    name: 'static with same radix',
    method: 'GET',
    path: '/user/comments',
  },
  {
    name: 'dynamic route',
    method: 'GET',
    path: '/user/lookup/username/hey',
  },
  {
    name: 'mixed static dynamic',
    method: 'GET',
    path: '/event/abcd1234/comments',
  },
  {
    name: 'post',
    method: 'POST',
    path: '/event/abcd1234/comment',
  },
  {
    name: 'long static',
    method: 'GET',
    path: '/very/deeply/nested/route/hello/there',
  },
  {
    name: 'wildcard',
    method: 'GET',
    path: '/static/index.html',
  },
]
```

让我们看看结果。

### 在 Node.js 上

以下截图显示了在 Node.js 上的结果。

![](/images/bench01.png)

![](/images/bench02.png)

![](/images/bench03.png)

![](/images/bench04.png)

![](/images/bench05.png)

![](/images/bench06.png)

![](/images/bench07.png)

![](/images/bench08.png)

### 在 Bun 上

以下截图显示了在 Bun 上的结果。

![](/images/bench09.png)

![](/images/bench10.png)

![](/images/bench11.png)

![](/images/bench12.png)

![](/images/bench13.png)

![](/images/bench14.png)

![](/images/bench15.png)

![](/images/bench16.png)

## Cloudflare Workers

**Hono 是最快的**，与其他用于 Cloudflare Workers 的路由器相比。

- 机器: Apple MacBook Pro, 32 GiB, M1 Pro
- 脚本: [benchmarks/handle-event](https://github.com/honojs/hono/tree/main/benchmarks/handle-event)

```
Hono x 402,820 ops/sec ±4.78% (80 runs sampled)
itty-router x 212,598 ops/sec ±3.11% (87 runs sampled)
sunder x 297,036 ops/sec ±4.76% (77 runs sampled)
worktop x 197,345 ops/sec ±2.40% (88 runs sampled)
最快的是 Hono
✨  完成于 28.06s。
```

## Deno

**Hono 是最快的**，与其他用于 Deno 的框架相比。

- 机器: Apple MacBook Pro, 32 GiB, M1 Pro, Deno v1.22.0
- 脚本: [benchmarks/deno](https://github.com/honojs/hono/tree/main/benchmarks/deno)
- 方法: `bombardier --fasthttp -d 10s -c 100 'http://localhost:8000/user/lookup/username/foo'`

| 框架 |   版本    |                  结果 |
| --------- | :----------: | -----------------------: |
| **Hono**  |    3.0.0     | **请求数/秒: 136112** |
| Fast      | 4.0.0-beta.1 |     请求数/秒: 103214 |
| Megalo    |    0.3.0     |      请求数/秒: 64597 |
| Faster    |     5.7      |      请求数/秒: 54801 |
| oak       |    10.5.1    |      请求数/秒: 43326 |
| opine     |    2.2.0     |      请求数/秒: 30700 |

另一个基准测试结果: [denosaurs/bench](https://github.com/denosaurs/bench)

## Bun

Hono 是 Bun 最快的框架之一。
您可以在下面看到。

- [SaltyAom/bun-http-framework-benchmark](https://github.com/SaltyAom/bun-http-framework-benchmark)
