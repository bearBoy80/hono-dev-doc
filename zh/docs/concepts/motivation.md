# 理念

在本节中，我们将讨论 Hono 的概念或理念。

## 动机

起初，我只是想在 Cloudflare Workers 上创建一个 Web 应用程序。
但是，没有一个好的框架可以在 Cloudflare Workers 上运行。
于是，我开始构建 Hono。

我认为这将是学习如何使用 Trie 树构建路由器的绝佳机会。
然后，一个朋友带着一个名为“RegExpRouter”的超快路由器出现了。
我还有一个朋友创建了基本身份验证中间件。

仅使用 Web 标准 API，我们就可以让它在 Deno 和 Bun 上运行。当人们问“Bun 有 Express 吗？”，我们可以回答，“没有，但有 Hono”。
（尽管 Express 现在可以在 Bun 上运行了。）

我们还有一些朋友制作了 GraphQL 服务器、Firebase 身份验证和 Sentry 中间件。
而且，我们还有一个 Node.js 适配器。
一个生态系统就这样诞生了。

换句话说，Hono 速度极快，可以实现很多事情，并且可以在任何地方运行。
我们可以想象，Hono 可能会成为 **Web 标准的标准**。
