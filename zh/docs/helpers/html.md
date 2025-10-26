# html 辅助函数

html 辅助函数可让您在名为 `html` 的标签的 JavaScript 模板字面量中编写 HTML。使用 `raw()`，内容将按原样呈现。您必须自己对这些字符串进行转义。

## 导入

```ts
import { Hono } from 'hono'
import { html, raw } from 'hono/html'
```

## `html`

```ts
const app = new Hono()

app.get('/:username', (c) => {
  const { username } = c.req.param()
  return c.html(
    html`<!doctype html>
      <h1>你好！ ${username}！</h1>`
  )
})
```

### 将代码片段插入 JSX

将内联脚本插入 JSX：

```tsx
app.get('/', (c) => {
  return c.html(
    <html>
      <head>
        <title>测试站点</title>
        {html`
          <script>
            // 无需使用 dangerouslySetInnerHTML。
            // 如果您在此处编写，它将不会被转义。
          </script>
        `}
      </head>
      <body>你好！</body>
    </html>
  )
})
```

### 充当函数式组件

由于 `html` 返回一个 HtmlEscapedString，因此它可以充当一个功能齐全的组件，而无需使用 JSX。

#### 使用 `html` 代替 `memo` 来加快进程

```typescript
const Footer = () => html`
  <footer>
    <address>我的地址...</address>
  </footer>
`
```

### 接收 props 并嵌入值

```typescript
interface SiteData {
  title: string
  description: string
  image: string
  children?: any
}
const Layout = (props: SiteData) => html`
<html>
<head>
  <meta charset="UTF-8">
  <title>${props.title}</title>
  <meta name="description" content="${props.description}">
  <head prefix="og: http://ogp.me/ns#">
  <meta property="og:type" content="article">
  <!-- 更多元素会减慢 JSX，但不会减慢模板字面量。 -->
  <meta property="og:title" content="${props.title}">
  <meta property="og:image" content="${props.image}">
</head>
<body>
  ${props.children}
</body>
</html>
`

const Content = (props: { siteData: SiteData; name: string }) => (
  <Layout {...props.siteData}>
    <h1>你好 {props.name}</h1>
  </Layout>
)

app.get('/', (c) => {
  const props = {
    name: '世界',
    siteData: {
      title: '你好 <> 世界',
      description: '这是一个描述',
      image: 'https://example.com/image.png',
    },
  }
  return c.html(<Content {...props} />)
})
```

## `raw()`

```ts
app.get('/', (c) => {
  const name = 'John &quot;Johnny&quot; Smith'
  return c.html(html`<p>我是 ${raw(name)}。</p>`)
})
```

## 提示

得益于这些库，Visual Studio Code 和 vim 也可以将模板字面量解释为 HTML，从而可以应用语法高亮和格式化。

- <https://marketplace.visualstudio.com/items?itemName=bierner.lit-html>
- <https://github.com/MaxMEllon/vim-jsx-pretty>
