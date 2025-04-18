---
title: Secure Headers 中间件
description: hono 内置的 Secure Headers 中间件，提供设置安全头功能。
---
# Secure Headers 中间件

Secure Headers 中间件简化了安全头的设置。部分灵感来自 Helmet 的功能，它允许你控制特定安全头的启用和禁用。

## 导入

```ts
import { Hono } from 'hono'
import { secureHeaders } from 'hono/secure-headers'
```

## 使用方法

你可以使用默认的最优设置。

```ts
const app = new Hono()
app.use(secureHeaders())
```

你可以通过将不需要的头设置为 false 来禁用它们。

```ts
const app = new Hono()
app.use(
  '*',
  secureHeaders({
    xFrameOptions: false,
    xXssProtection: false,
  })
)
```

你可以使用字符串覆盖默认的头部值。

```ts
const app = new Hono()
app.use(
  '*',
  secureHeaders({
    strictTransportSecurity:
      'max-age=63072000; includeSubDomains; preload',
    xFrameOptions: 'DENY',
    xXssProtection: '1',
  })
)
```

## 支持的选项

每个选项对应以下头部键值对。

| 选项                           | 头部                                                                                                                                | 值                                                                        | 默认值     |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ---------- |
| -                              | X-Powered-By                                                                                                                         | (删除头部)                                                                 | True       |
| contentSecurityPolicy          | [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)                                                     | 用法：[设置 Content-Security-Policy](#设置-content-security-policy)        | 无设置     |
| contentSecurityPolicyReportOnly| [Content-Security-Policy-Report-Only](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only) | 用法：[设置 Content-Security-Policy](#设置-content-security-policy)        | 无设置     |
| crossOriginEmbedderPolicy      | [Cross-Origin-Embedder-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)              | require-corp                                                                | **False**  |
| crossOriginResourcePolicy      | [Cross-Origin-Resource-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy)               | same-origin                                                                 | True       |
| crossOriginOpenerPolicy        | [Cross-Origin-Opener-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy)                   | same-origin                                                                 | True       |
| originAgentCluster             | [Origin-Agent-Cluster](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin-Agent-Cluster)                               | ?1                                                                          | True       |
| referrerPolicy                 | [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)                                         | no-referrer                                                                 | True       |
| reportingEndpoints             | [Reporting-Endpoints](https://www.w3.org/TR/reporting-1/#header)                                                                     | 用法：[设置 Content-Security-Policy](#设置-content-security-policy)        | 无设置     |
| reportTo                       | [Report-To](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)                             | 用法：[设置 Content-Security-Policy](#设置-content-security-policy)        | 无设置     |
| strictTransportSecurity        | [Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)                     | max-age=15552000; includeSubDomains                                         | True       |
| xContentTypeOptions            | [X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)                           | nosniff                                                                     | True       |
| xDnsPrefetchControl            | [X-DNS-Prefetch-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-DNS-Prefetch-Control)                           | off                                                                         | True       |
| xDownloadOptions               | [X-Download-Options](https://learn.microsoft.com/en-us/archive/blogs/ie/ie8-security-part-v-comprehensive-protection#mime-handling-force-save)                                                                                                                   | noopen                                                                      | True       |
| xFrameOptions                  | [X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)                                         | SAMEORIGIN                                                                  | True       |
| xPermittedCrossDomainPolicies  | [X-Permitted-Cross-Domain-Policies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Permitted-Cross-Domain-Policies)     | none                                                                        | True       |
| xXssProtection                 | [X-XSS-Protection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)                                       | 0                                                                           | True       |
| permissionPolicy               | [Permissions-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy)                                   | 用法：[设置 Permission-Policy](#设置-permission-policy)                    | 无设置     |

## 中间件冲突

在处理操作相同头部的中间件时，请注意指定的顺序。

在这种情况下，Secure-headers 会生效，`x-powered-by` 会被移除：

```ts
const app = new Hono()
app.use(secureHeaders())
app.use(poweredBy())
```

在这种情况下，Powered-By 会生效，`x-powered-by` 会被添加：

```ts
const app = new Hono()
app.use(poweredBy())
app.use(secureHeaders())
```

## 设置 Content-Security-Policy

```ts
const app = new Hono()
app.use(
  '/test',
  secureHeaders({
    reportingEndpoints: [
      {
        name: 'endpoint-1',
        url: 'https://example.com/reports',
      },
    ],
    // -- 或者使用以下替代方案
    // reportTo: [
    //   {
    //     group: 'endpoint-1',
    //     max_age: 10886400,
    //     endpoints: [{ url: 'https://example.com/reports' }],
    //   },
    // ],
    contentSecurityPolicy: {
      defaultSrc: ["'self'"],
      baseUri: ["'self'"],
      childSrc: ["'self'"],
      connectSrc: ["'self'"],
      fontSrc: ["'self'", 'https:', 'data:'],
      formAction: ["'self'"],
      frameAncestors: ["'self'"],
      frameSrc: ["'self'"],
      imgSrc: ["'self'", 'data:'],
      manifestSrc: ["'self'"],
      mediaSrc: ["'self'"],
      objectSrc: ["'none'"],
      reportTo: 'endpoint-1',
      sandbox: ['allow-same-origin', 'allow-scripts'],
      scriptSrc: ["'self'"],
      scriptSrcAttr: ["'none'"],
      scriptSrcElem: ["'self'"],
      styleSrc: ["'self'", 'https:', "'unsafe-inline'"],
      styleSrcAttr: ['none'],
      styleSrcElem: ["'self'", 'https:', "'unsafe-inline'"],
      upgradeInsecureRequests: [],
      workerSrc: ["'self'"],
    },
  })
)
```

### `nonce` 属性

你可以通过将从 `hono/secure-headers` 导入的 `NONCE` 添加到 `scriptSrc` 或 `styleSrc` 中，为 `script` 或 `style` 元素添加 [`nonce` 属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)：

```tsx
import { secureHeaders, NONCE } from 'hono/secure-headers'
import type { SecureHeadersVariables } from 'hono/secure-headers'

// 指定变量类型以推断 `c.get('secureHeadersNonce')`：
type Variables = SecureHeadersVariables

const app = new Hono<{ Variables: Variables }>()

// 将预定义的 nonce 值设置到 `scriptSrc`：
app.get(
  '*',
  secureHeaders({
    contentSecurityPolicy: {
      scriptSrc: [NONCE, 'https://allowed1.example.com'],
    },
  })
)

// 从 `c.get('secureHeadersNonce')` 获取值：
app.get('/', (c) => {
  return c.html(
    <html>
      <body>
        {/** 内容 */}
        <script
          src='/js/client.js'
          nonce={c.get('secureHeadersNonce')}
        />
      </body>
    </html>
  )
})
```

如果你想自己生成 nonce 值，也可以指定一个函数，如下所示：

```tsx
const app = new Hono<{
  Variables: { myNonce: string }
}>()

const myNonceGenerator: ContentSecurityPolicyOptionHandler = (c) => {
  // 此函数在每个请求时调用
  const nonce = Math.random().toString(36).slice(2)
  c.set('myNonce', nonce)
  return `'nonce-${nonce}'`
}

app.get(
  '*',
  secureHeaders({
    contentSecurityPolicy: {
      scriptSrc: [myNonceGenerator, 'https://allowed1.example.com'],
    },
  })
)

app.get('/', (c) => {
  return c.html(
    <html>
      <body>
        {/** 内容 */}
        <script src='/js/client.js' nonce={c.get('myNonce')} />
      </body>
    </html>
  )
})
```

## 设置 Permission-Policy

Permission-Policy 头允许你控制浏览器中可以使用哪些功能和 API。以下是设置示例：

```ts
const app = new Hono()
app.use(
  '*',
  secureHeaders({
    permissionsPolicy: {
      fullscreen: ['self'], // fullscreen=(self)
      bluetooth: ['none'], // bluetooth=(none)
      payment: ['self', 'https://example.com'], // payment=(self "https://example.com")
      syncXhr: [], // sync-xhr=()
      camera: false, // camera=none
      microphone: true, // microphone=*
      geolocation: ['*'], // geolocation=*
      usb: ['self', 'https://a.example.com', 'https://b.example.com'], // usb=(self "https://a.example.com" "https://b.example.com")
      accelerometer: ['https://*.example.com'], // accelerometer=("https://*.example.com")
      gyroscope: ['src'], // gyroscope=(src)
      magnetometer: [
        'https://a.example.com',
        'https://b.example.com',
      ], // magnetometer=("https://a.example.com" "https://b.example.com")
    },
  })
)
```
