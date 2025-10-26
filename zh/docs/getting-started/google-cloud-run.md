# Google Cloud Run

[Google Cloud Run](https://cloud.google.com/run) 是由 Google Cloud 构建的无服务器平台。您可以响应事件运行代码，Google 会自动为您管理底层计算资源。

Google Cloud Run 使用容器来运行您的服务。这意味着您可以通过提供 Dockerfile 来使用您喜欢的任何运行时（例如 Deno 或 Bun）。如果未提供 Dockerfile，Google Cloud Run 将使用默认的 Node.js buildpack。

本指南假定您已经拥有一个 Google Cloud 帐户和一个结算帐户。

## 1. 安装 CLI

使用 Google Cloud Platform 时，最简单的方法是使用 [gcloud CLI](https://cloud.google.com/sdk/docs/install)。

例如，在 macOS 上使用 Homebrew：

```sh
brew install --cask google-cloud-sdk
```

使用 CLI 进行身份验证。

```sh
gcloud auth login
```

## 2. 项目设置

创建一个项目。在提示符下接受自动生成的项目 ID。

```sh
gcloud projects create --set-as-default --name="my app"
```

为您的项目 ID 和项目编号创建环境变量，以便于重复使用。在使用 `gcloud projects list` 命令成功返回项目之前，可能需要大约 30 秒。

```sh
PROJECT_ID=$(gcloud projects list \
    --format='value(projectId)' \
    --filter='name="my app"')

PROJECT_NUMBER=$(gcloud projects list \
    --format='value(projectNumber)' \
    --filter='name="my app"')

echo $PROJECT_ID $PROJECT_NUMBER
```

找到您的结算帐户 ID。

```sh
gcloud billing accounts list
```

将您在上一个命令中获得的结算帐户添加到项目中。

```sh
gcloud billing projects link $PROJECT_ID \
    --billing-account=[billing_account_id]
```

启用所需的 API。

```sh
gcloud services enable run.googleapis.com \
    cloudbuild.googleapis.com
```

更新服务帐户权限以访问 Cloud Build。

```sh
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com \
    --role=roles/run.builder
```

## 3. Hello World

使用“create-hono”命令开始您的项目。选择 `nodejs`。

```sh
npm create hono@latest my-app
```

进入 `my-app` 并安装依赖项。

```sh
cd my-app
npm i
```

将 `src/index.ts` 中的端口更新为 `8080`。

<!-- prettier-ignore -->
```ts
import { serve } from '@hono/node-server'
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => {
  return c.text('你好 Hono！')
})

serve({
  fetch: app.fetch,
  port: 3000 // [!code --]
  port: 8080 // [!code ++]
}, (info) => {
  console.log(`服务器正在 http://localhost:${info.port} 上运行`)
})
```

在本地运行开发服务器。然后，在您的 Web 浏览器中访问 http://localhost:8080。

```sh
npm run dev
```

## 4. 部署

开始部署并按照交互式提示进行操作（例如，选择一个区域）。

```sh
gcloud run deploy my-app --source . --allow-unauthenticated
```

## 更改运行时

如果您想使用 Deno 或 Bun 运行时（或自定义的 Node.js 容器）进行部署，请添加一个带有您所需环境的 `Dockerfile`（以及可选的 `.dockerignore`）。

有关容器化的信息，请参阅：

- [Node.js](/docs/getting-started/nodejs#building-deployment)
- [Bun](https://bun.com/guides/ecosystem/docker)
- [Deno](https://docs.deno.com/examples/google_cloud_run_tutorial)
