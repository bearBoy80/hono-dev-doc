# 客户端组件

`hono/jsx` 不仅支持服务器端，还支持客户端。这意味着可以创建在浏览器中运行的交互式 UI。我们称之为客户端组件或 `hono/jsx/dom`。

它速度快且体积小。`hono/jsx/dom` 中的计数器程序经 Brotli 压缩后仅为 2.8KB。而 React 则为 47.8KB。

本节介绍客户端组件特有的功能。

## 计数器示例

这是一个简单计数器的示例，其代码与 React 中的相同。

```tsx
import { useState } from 'hono/jsx'
import { render } from 'hono/jsx/dom'

function Counter() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={() => setCount(count + 1)}>递增</button>
    </div>
  )
}

function App() {
  return (
    <html>
      <body>
        <Counter />
      </body>
    </html>
  )
}

const root = document.getElementById('root')
render(<App />, root)
```

## `render()`

您可以使用 `render()` 在指定的 HTML 元素内插入 JSX 组件。

```tsx
render(<Component />, container)
```

## 与 React 兼容的钩子

hono/jsx/dom 具有与 React 兼容或部分兼容的钩子。您可以通过查看 [React 文档](https://react.dev/reference/react/hooks)来了解这些 API。

- `useState()`
- `useEffect()`
- `useRef()`
- `useCallback()`
- `use()`
- `startTransition()`
- `useTransition()`
- `useDeferredValue()`
- `useMemo()`
- `useLayoutEffect()`
- `useReducer()`
- `useDebugValue()`
- `createElement()`
- `memo()`
- `isValidElement()`
- `useId()`
- `createRef()`
- `forwardRef()`
- `useImperativeHandle()`
- `useSyncExternalStore()`
- `useInsertionEffect()`
- `useFormStatus()`
- `useActionState()`
- `useOptimistic()`

## `startViewTransition()` 系列

`startViewTransition()` 系列包含用于轻松处理 [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API) 的原始钩子和函数。以下是如何使用它们的示例。

### 1. 一个最简单的示例

您可以使用 `startViewTransition()` 快速编写一个使用 `document.startViewTransition` 的过渡。

```tsx
import { useState, startViewTransition } from 'hono/jsx'
import { css, Style } from 'hono/css'

export default function App() {
  const [showLargeImage, setShowLargeImage] = useState(false)
  return (
    <>
      <Style />
      <button
        onClick={() =>
          startViewTransition(() =>
            setShowLargeImage((state) => !state)
          )
        }
      >
        单击！
      </button>
      <div>
        {!showLargeImage ? (
          <img src='https://hono.dev/images/logo.png' />
        ) : (
          <div
            class={css`
              background: url('https://hono.dev/images/logo-large.png');
              background-size: contain;
              background-repeat: no-repeat;
              background-position: center;
              width: 600px;
              height: 600px;
            `}
          ></div>
        )}
      </div>
    </>
  )
}
```

### 2. 将 `viewTransition()` 与 `keyframes()` 结合使用

`viewTransition()` 函数允许您获取唯一的 `view-transition-name`。

您可以将其与 `keyframes()` 结合使用，`::view-transition-old()` 会转换为 `::view-transition-old(${uniqueName))`。

```tsx
import { useState, startViewTransition } from 'hono/jsx'
import { viewTransition } from 'hono/jsx/dom/css'
import { css, keyframes, Style } from 'hono/css'

const rotate = keyframes`
  from {
    rotate: 0deg;
  }
  to {
    rotate: 360deg;
  }
`

export default function App() {
  const [showLargeImage, setShowLargeImage] = useState(false)
  const [transitionNameClass] = useState(() =>
    viewTransition(css`
      ::view-transition-old() {
        animation-name: ${rotate};
      }
      ::view-transition-new() {
        animation-name: ${rotate};
      }
    `)
  )
  return (
    <>
      <Style />
      <button
        onClick={() =>
          startViewTransition(() =>
            setShowLargeImage((state) => !state)
          )
        }
      >
        单击！
      </button>
      <div>
        {!showLargeImage ? (
          <img src='https://hono.dev/images/logo.png' />
        ) : (
          <div
            class={css`
              ${transitionNameClass}
              background: url('https://hono.dev/images/logo-large.png');
              background-size: contain;
              background-repeat: no-repeat;
              background-position: center;
              width: 600px;
              height: 600px;
            `}
          ></div>
        )}
      </div>
    </>
  )
}
```

### 3. 使用 `useViewTransition`

如果您只想在动画期间更改样式。您可以使用 `useViewTransition()`。此钩子返回 `[boolean, (callback: () => void) => void]`，它们分别是 `isUpdating` 标志和 `startViewTransition()` 函数。

使用此钩子时，组件将在以下两个时间点进行评估。

- 在对 `startViewTransition()` 的调用的回调内部。
- 当 [the `finish` promise becomes fulfilled](https://developer.mozilla.org/en-US/docs/Web/API/ViewTransition/finished)

```tsx
import { useState, useViewTransition } from 'hono/jsx'
import { viewTransition } from 'hono/jsx/dom/css'
import { css, keyframes, Style } from 'hono/css'

const rotate = keyframes`
  from {
    rotate: 0deg;
  }
  to {
    rotate: 360deg;
  }
`

export default function App() {
  const [isUpdating, startViewTransition] = useViewTransition()
  const [showLargeImage, setShowLargeImage] = useState(false)
  const [transitionNameClass] = useState(() =>
    viewTransition(css`
      ::view-transition-old() {
        animation-name: ${rotate};
      }
      ::view-transition-new() {
        animation-name: ${rotate};
      }
    `)
  )
  return (
    <>
      <Style />
      <button
        onClick={() =>
          startViewTransition(() =>
            setShowLargeImage((state) => !state)
          )
        }
      >
        单击！
      </button>
      <div>
        {!showLargeImage ? (
          <img src='https://hono.dev/images/logo.png' />
        ) : (
          <div
            class={css`
              ${transitionNameClass}
              background: url('https://hono.dev/images/logo-large.png');
              background-size: contain;
              background-repeat: no-repeat;
              background-position: center;
              width: 600px;
              height: 600px;
              position: relative;
              ${isUpdating &&
              css`
                &:before {
                  content: '正在加载...';
                  position: absolute;
                  top: 50%;
                  left: 50%;
                }
              `}
            `}
          ></div>
        )}
      </div>
    </>
  )
}
```

## `hono/jsx/dom` 运行时

有一个用于客户端组件的小型 JSX 运行时。使用它将比使用 `hono/jsx` 产生更小的捆绑结果。在 `tsconfig.json` 中指定 `hono/jsx/dom`。对于 Deno，请修改 deno.json。

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx/dom"
  }
}
```

或者，您可以在 `vite.config.ts` 的 esbuild 转换选项中指定 `hono/jsx/dom`。

```ts
import { defineConfig } from 'vite'

export default defineConfig({
  esbuild: {
    jsxImportSource: 'hono/jsx/dom',
  },
})
```
