# css 辅助函数

css 辅助函数 - `hono/css` - 是 Hono 内置的 CSS in JS(X)。

您可以在名为 `css` 的 JavaScript 模板字面量中编写 CSS in JSX。`css` 的返回值将是类名，该类名将设置为 class 属性的值。然后 `<Style />` 组件将包含 CSS 的值。

## 导入

```ts
import { Hono } from 'hono'
import { css, cx, keyframes, Style } from 'hono/css'
```

## `css` <Badge style="vertical-align: middle;" type="warning" text="实验性" />

您可以在 `css` 模板字面量中编写 CSS。在这种情况下，它使用 `headerClass` 作为 `class` 属性的值。不要忘记添加 `<Style />`，因为它包含 CSS 内容。

```ts{10,13}
app.get('/', (c) => {
  const headerClass = css`
    background-color: orange;
    color: white;
    padding: 1rem;
  `
  return c.html(
    <html>
      <head>
        <Style />
      </head>
      <body>
        <h1 class={headerClass}>你好！</h1>
      </body>
    </html>
  )
})
```

您可以使用[嵌套选择器](https://developer.mozilla.org/en-US/docs/Web/CSS/Nesting_selector) `&` 来设置伪类样式，例如 `:hover`：

```ts
const buttonClass = css`
  background-color: #fff;
  &:hover {
    background-color: red;
  }
`
```

### 扩展

您可以通过嵌入类名来扩展 CSS 定义。

```tsx
const baseClass = css`
  color: white;
  background-color: blue;
`

const header1Class = css`
  ${baseClass}
  font-size: 3rem;
`

const header2Class = css`
  ${baseClass}
  font-size: 2rem;
`
```

此外，`${baseClass} {}` 的语法允许嵌套类。

```tsx
const headerClass = css`
  color: white;
  background-color: blue;
`
const containerClass = css`
  ${headerClass} {
    h1 {
      font-size: 3rem;
    }
  }
`
return c.render(
  <div class={containerClass}>
    <header class={headerClass}>
      <h1>你好！</h1>
    </header>
  </div>
)
```

### 全局样式

一个名为 `:-hono-global` 的伪选择器允许您定义全局样式。

```tsx
const globalClass = css`
  :-hono-global {
    html {
      font-family: Arial, Helvetica, sans-serif;
    }
  }
`

return c.render(
  <div class={globalClass}>
    <h1>你好！</h1>
    <p>今天是个好天气。</p>
  </div>
)
```

或者，您可以使用 `css` 字面量在 `<Style />` 组件中编写 CSS。

```tsx
export const renderer = jsxRenderer(({ children, title }) => {
  return (
    <html>
      <head>
        <Style>{css`
          html {
            font-family: Arial, Helvetica, sans-serif;
          }
        `}</Style>
        <title>{title}</title>
      </head>
      <body>
        <div>{children}</div>
      </body>
    </html>
  )
})
```

## `keyframes` <Badge style="vertical-align: middle;" type="warning" text="实验性" />

您可以使用 `keyframes` 编写 `@keyframes` 的内容。在这种情况下，`fadeInAnimation` 将是动画的名称

```tsx
const fadeInAnimation = keyframes`
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
`
const headerClass = css`
  animation-name: ${fadeInAnimation};
  animation-duration: 2s;
`
const Header = () => <a class={headerClass}>你好！</a>
```

## `cx` <Badge style="vertical-align: middle;" type="warning" text="实验性" />

`cx` 复合了两个类名。

```tsx
const buttonClass = css`
  border-radius: 10px;
`
const primaryClass = css`
  background: orange;
`
const Button = () => (
  <a class={cx(buttonClass, primaryClass)}>单击！</a>
)
```

它还可以复合简单的字符串。

```tsx
const Header = () => <a class={cx('h1', primaryClass)}>嗨</a>
```

## 与[安全标头](/docs/middleware/builtin/secure-headers)中间件结合使用

如果您想将 css 辅助函数与[安全标头](/docs/middleware/builtin/secure-headers)中间件结合使用，可以将 [`nonce` 属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)添加到 `<Style nonce={c.get('secureHeadersNonce')} />` 中，以避免由 css 辅助函数引起的内容安全策略。

```tsx{8,23}
import { secureHeaders, NONCE } from 'hono/secure-headers'

app.get(
  '*',
  secureHeaders({
    contentSecurityPolicy: {
      // 将预定义的 nonce 值设置为 `styleSrc`：
      styleSrc: [NONCE],
    },
  })
)

app.get('/', (c) => {
  const headerClass = css`
    background-color: orange;
    color: white;
    padding: 1rem;
  `
  return c.html(
    <html>
      <head>
        {/* 在 css 辅助函数 `style` 和 `script` 元素上设置 `nonce` 属性 */}
        <Style nonce={c.get('secureHeadersNonce')} />
      </head>
      <body>
        <h1 class={headerClass}>你好！</h1>
      </body>
    </html>
  )
})
```

## 提示

如果您使用 VS Code，可以使用 [vscode-styled-components](https://marketplace.visualstudio.com/items?itemName=styled-components.vscode-styled-components) 来实现 css 标记字面量的语法高亮和 IntelliSense。

![](/images/css-ss.png)
