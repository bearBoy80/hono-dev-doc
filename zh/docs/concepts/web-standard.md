# Web 标准

Hono 仅使用 **Web 标准**，如 Fetch。
它们最初用于 `fetch` 函数，并由处理 HTTP 请求和响应的基本对象组成。
除了 `Requests` 和 `Responses`，还有 `URL`、`URLSearchParam`、`Headers` 等。

Cloudflare Workers、Deno 和 Bun 也基于 Web 标准构建。
例如，一个返回“Hello World”的服务器可以如下编写。这可以在 Cloudflare Workers 和 Bun 上运行。

```ts twoslash
export default {
  async fetch() {
    return new Response('Hello World')
  },
}
```

Hono 仅使用 Web 标准，这意味着 Hono 可以在任何支持它们的运行时上运行。
此外，我们还有一个 Node.js 适配器。Hono 可以在以下运行时上运行：

- Cloudflare Workers (`workerd`)
- Deno
- Bun
- Fastly Compute
- AWS Lambda
- Node.js
- Vercel (edge-light)

它也适用于 Netlify 和其他平台。
同一套代码可以在所有平台上运行。

Cloudflare Workers、Deno、Shopify 等公司发起了 [WinterCG](https://wintercg.org)，讨论使用 Web 标准实现“网络互操作性”的可能性。
Hono 将追随他们的脚步，致力于成为 **Web 标准的标准**。
