# Lambda@Edge

[Lambda@Edge](https://aws.amazon.com/lambda/edge/) 是亚马逊网络服务提供的一个无服务器平台。它允许您在亚马逊 CloudFront 的边缘站点运行 Lambda 函数，从而能够自定义 HTTP 请求/响应的行为。

Hono 支持 Lambda@Edge 与 Node.js 18+ 环境。

## 1. 设置

在 Lambda@Edge 上创建应用程序时，
[CDK](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-cdk.html)
对于设置诸如 CloudFront、IAM 角色、API 网关等函数非常有用。

使用 `cdk` CLI 初始化您的项目。

::: code-group

```sh [npm]
mkdir my-app
cd my-app
cdk init app -l typescript
npm i hono
mkdir lambda
```

```sh [yarn]
mkdir my-app
cd my-app
cdk init app -l typescript
yarn add hono
mkdir lambda
```

```sh [pnpm]
mkdir my-app
cd my-app
cdk init app -l typescript
pnpm add hono
mkdir lambda
```

```sh [bun]
mkdir my-app
cd my-app
cdk init app -l typescript
bun add hono
mkdir lambda
```

:::

## 2. Hello World

编辑 `lambda/index_edge.ts`。

```ts
import { Hono } from 'hono'
import { handle } from 'hono/lambda-edge'

const app = new Hono()

app.get('/', (c) => c.text('你好 Hono on Lambda@Edge！'))

export const handler = handle(app)
```

## 3. 部署

编辑 `bin/my-app.ts`。

```ts
#!/usr/bin/env node
import 'source-map-support/register'
import * as cdk from 'aws-cdk-lib'
import { MyAppStack } from '../lib/my-app-stack'

const app = new cdk.App()
new MyAppStack(app, 'MyAppStack', {
  env: {
    account: process.env.CDK_DEFAULT_ACCOUNT,
    region: 'us-east-1',
  },
})
```

编辑 `lambda/cdk-stack.ts`。

```ts
import { Construct } from 'constructs'
import * as cdk from 'aws-cdk-lib'
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront'
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins'
import * as lambda from 'aws-cdk-lib/aws-lambda'
import { NodejsFunction } from 'aws-cdk-lib/aws-lambda-nodejs'
import * as s3 from 'aws-cdk-lib/aws-s3'

export class MyAppStack extends cdk.Stack {
  public readonly edgeFn: lambda.Function

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props)
    const edgeFn = new NodejsFunction(this, 'edgeViewer', {
      entry: 'lambda/index_edge.ts',
      handler: 'handler',
      runtime: lambda.Runtime.NODEJS_20_X,
    })

    // 上传任何 html
    const originBucket = new s3.Bucket(this, 'originBucket')

    new cloudfront.Distribution(this, 'Cdn', {
      defaultBehavior: {
        origin: new origins.S3Origin(originBucket),
        edgeLambdas: [
          {
            functionVersion: edgeFn.currentVersion,
            eventType: cloudfront.LambdaEdgeEventType.VIEWER_REQUEST,
          },
        ],
      },
    })
  }
}
```

最后，运行命令进行部署：

```sh
cdk deploy
```

## 回调

如果您想添加基本身份验证并在验证后继续处理请求，可以使用 `c.env.callback()`

```ts
import { Hono } from 'hono'
import { basicAuth } from 'hono/basic-auth'
import type { Callback, CloudFrontRequest } from 'hono/lambda-edge'
import { handle } from 'hono/lambda-edge'

type Bindings = {
  callback: Callback
  request: CloudFrontRequest
}

const app = new Hono<{ Bindings: Bindings }>()

app.get(
  '*',
  basicAuth({
    username: 'hono',
    password: 'acoolproject',
  })
)

app.get('/', async (c, next) => {
  await next()
  c.env.callback(null, c.env.request)
})

export const handler = handle(app)
```
