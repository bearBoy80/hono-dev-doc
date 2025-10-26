# Supabase 边缘函数

[Supabase](https://supabase.com/) 是 Firebase 的一个开源替代品，提供了一套类似于 Firebase 功能的工具，包括数据库、身份验证、存储以及现在的无服务器函数。

Supabase 边缘函数是服务器端 TypeScript 函数，它们分布在全球各地，运行在离您的用户更近的地方，以提高性能。这些函数是使用 [Deno](https://deno.com/) 开发的，这带来了几个好处，包括更高的安全性和现代的 JavaScript/TypeScript 运行时。

以下是如何开始使用 Supabase 边缘函数：

## 1. 设置

### 先决条件

在开始之前，请确保您已安装 Supabase CLI。如果您尚未安装，请按照[官方文档](https://supabase.com/docs/guides/cli/getting-started)中的说明进行操作。

### 创建一个新项目

1. 打开您的终端或命令提示符。

2. 通过运行以下命令在您的本地计算机上的目录中创建一个新的 Supabase 项目：

```bash
supabase init

```

此命令会在当前目录中初始化一个新的 Supabase 项目。

### 添加一个边缘函数

3. 在您的 Supabase 项目中，创建一个名为 `hello-world` 的新边缘函数：

```bash
supabase functions new hello-world

```

此命令会在您的项目中创建一个具有指定名称的新边缘函数。

## 2. Hello World

通过修改文件 `supabase/functions/hello-world/index.ts` 来编辑 `hello-world` 函数：

```ts
import { Hono } from 'jsr:@hono/hono'

// 将此更改为您的函数名称
const functionName = 'hello-world'
const app = new Hono().basePath(`/${functionName}`)

app.get('/hello', (c) => c.text('来自 hono-server 的你好！'))

Deno.serve(app.fetch)
```

## 3. 运行

要本地运行该函数，请使用以下命令：

1. 使用以下命令来提供该函数：

```bash
supabase start # 启动 supabase 堆栈
supabase functions serve --no-verify-jwt # 启动函数监视程序
```

`--no-verify-jwt` 标志允许您在本地开发期间绕过 JWT 验证。

2. 使用 cURL 或 Postman 向 `http://127.0.0.1:54321/functions/v1/hello-world/hello` 发出 GET 请求：

```bash
curl  --location  'http://127.0.0.1:54321/functions/v1/hello-world/hello'
```

此请求应返回文本“来自 hono-server 的你好！”。

## 4. 部署

您可以使用一个命令部署 Supabase 中的所有边缘函数：

```bash
supabase functions deploy
```

或者，您可以通过在部署命令中指定函数的名称来部署单个边缘函数：

```bash
supabase functions deploy hello-world

```

有关更多部署方法，请访问 Supabase 关于[部署到生产环境](https://supabase.com/docs/guides/functions/deploy)的文档。
