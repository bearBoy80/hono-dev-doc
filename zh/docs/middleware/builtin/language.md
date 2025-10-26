# 语言中间件

语言检测器中间件自动从各种来源确定用户的首选语言（区域设置），并通过 `c.get('language')` 使其可用。检测策略包括查询参数、cookie、标头和 URL 路径段。非常适合国际化 (i18n) 和特定于区域设置的内容。

## 导入

```ts
import { Hono } from 'hono'
import { languageDetector } from 'hono/language'
```

## 基本用法

从查询字符串、cookie 和标头（默认顺序）检测语言，回退到英语：

```ts
const app = new Hono()

app.use(
  languageDetector({
    supportedLanguages: ['en', 'ar', 'ja'], // 必须包含回退语言
    fallbackLanguage: 'en', // 必需
  })
)

app.get('/', (c) => {
  const lang = c.get('language')
  return c.text(`你好！您的语言是 ${lang}`)
})
```

### 客户端示例

```sh
# 通过路径
curl http://localhost:8787/ar/home

# 通过查询参数
curl http://localhost:8787/?lang=ar

# 通过 cookie
curl -H 'Cookie: language=ja' http://localhost:8787/

# 通过标头
curl -H 'Accept-Language: ar,en;q=0.9' http://localhost:8787/
```

## 默认配置

```ts
export const DEFAULT_OPTIONS: DetectorOptions = {
  order: ['querystring', 'cookie', 'header'],
  lookupQueryString: 'lang',
  lookupCookie: 'language',
  lookupFromHeaderKey: 'accept-language',
  lookupFromPathIndex: 0,
  caches: ['cookie'],
  ignoreCase: true,
  fallbackLanguage: 'en',
  supportedLanguages: ['en'],
  cookieOptions: {
    sameSite: 'Strict',
    secure: true,
    maxAge: 365 * 24 * 60 * 60,
    httpOnly: true,
  },
  debug: false,
}
```

## 主要行为

### 检测工作流程

1.  **顺序**：默认按此顺序检查来源：
    - 查询参数 (?lang=ar)
    - Cookie (language=ar)
    - Accept-Language 标头

2.  **缓存**：将检测到的语言存储在 cookie 中（默认为 1 年）

3.  **回退**：如果未检测到有效语言，则使用 `fallbackLanguage`（必须在 `supportedLanguages` 中）

## 高级配置

### 自定义检测顺序

优先检测 URL 路径（例如 /en/about）：

```ts
app.use(
  languageDetector({
    order: ['path', 'cookie', 'querystring', 'header'],
    lookupFromPathIndex: 0, // /en/profile → 索引 0 = 'en'
    supportedLanguages: ['en', 'ar'],
    fallbackLanguage: 'en',
  })
)
```

### 语言代码转换

规范化复杂代码（例如 en-US → en）：

```ts
app.use(
  languageDetector({
    convertDetectedLanguage: (lang) => lang.split('-')[0],
    supportedLanguages: ['en', 'ja'],
    fallbackLanguage: 'en',
  })
)
```

### Cookie 配置

```ts
app.use(
  languageDetector({
    lookupCookie: 'app_lang',
    caches: ['cookie'],
    cookieOptions: {
      path: '/', // Cookie 路径
      sameSite: 'Lax', // Cookie same-site 策略
      secure: true, // 仅通过 HTTPS 发送
      maxAge: 86400 * 365, // 1 年有效期
      httpOnly: true, // 无法通过 JavaScript 访问
      domain: '.example.com', // 可选：特定域
    },
  })
)
```

要禁用 cookie 缓存：

```ts
languageDetector({
  caches: false,
})
```

### 调试

记录检测步骤：

```ts
languageDetector({
  debug: true, // 显示：“从查询字符串检测到：ar”
})
```

## 选项参考

### 基本选项

| 选项               | 类型             | 默认                               | 必需 | 描述            |
| :------------------- | :--------------- | :------------------------------------ | :------- | :--------------------- |
| `supportedLanguages` | `string[]`       | `['en']`                              | 是      | 允许的语言代码 |
| `fallbackLanguage`   | `string`         | `'en'`                                | 是      | 默认语言       |
| `order`              | `DetectorType[]` | `['querystring', 'cookie', 'header']` | 否       | 检测顺序     |
| `debug`              | `boolean`        | `false`                               | 否       | 启用日志记录         |

### 检测选项

| 选项                | 类型     | 默认             | 描述          |
| :-------------------- | :------- | :------------------ | :------------------- |
| `lookupQueryString`   | `string` | `'lang'`            | 查询参数名称 |
| `lookupCookie`        | `string` | `'language'`        | Cookie 名称          |
| `lookupFromHeaderKey` | `string` | `'accept-language'` | 标头名称          |
| `lookupFromPathIndex` | `number` | `0`                 | 路径段索引   |

### Cookie 选项

| 选项                   | 类型                          | 默认      | 描述          |
| :----------------------- | :---------------------------- | :----------- | :------------------- |
| `caches`                 | `CacheType[] \| false`        | `['cookie']` | 缓存设置       |
| `cookieOptions.path`     | `string`                      | `'/'`        | Cookie 路径          |
| `cookieOptions.sameSite` | `'Strict' \| 'Lax' \| 'None'` | `'Strict'`   | SameSite 策略      |
| `cookieOptions.secure`   | `boolean`                     | `true`       | 仅 HTTPS           |
| `cookieOptions.maxAge`   | `number`                      | `31536000`   | 有效期（秒） |
| `cookieOptions.httpOnly` | `boolean`                     | `true`       | JS 可访问性     |
| `cookieOptions.domain`   | `string`                      | `undefined`  | Cookie 域        |

### 高级选项

| 选项                    | 类型                       | 默认     | 描述               |
| :------------------------ | :------------------------- | :---------- | :------------------------ |
| `ignoreCase`              | `boolean`                  | `true`      | 不区分大小写匹配 |
| `convertDetectedLanguage` | `(lang: string) => string` | `undefined` | 语言代码转换器 |

## 验证和错误处理

- `fallbackLanguage` 必须在 `supportedLanguages` 中（在设置期间抛出错误）
- `lookupFromPathIndex` 必须 ≥ 0
- 无效配置会在中间件初始化期间抛出错误
- 检测失败会静默使用 `fallbackLanguage`

## 常见方案

### 基于路径的路由

```ts
app.get('/:lang/home', (c) => {
  const lang = c.get('language') // 'en'、'ar' 等。
  return c.json({ message: getLocalizedContent(lang) })
})
```

### 多种支持的语言

```ts
languageDetector({
  supportedLanguages: ['en', 'en-GB', 'ar', 'ar-EG'],
  convertDetectedLanguage: (lang) => lang.replace('_', '-'), // 规范化
})
```
