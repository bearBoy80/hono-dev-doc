# 服务器计时中间件

[服务器计时](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing)中间件在响应标头中提供性能指标。

::: info
注意：在 Cloudflare Workers 上，计时器指标可能不准确，
因为[计时器仅显示上次 I/O 的时间](https://developers.cloudflare.com/workers/learning/security-model/#step-1-disallow-timers-and-multi-threading)。
:::

## 导入

```ts [npm]
import { Hono } from 'hono'
import { timing, setMetric, startTime, endTime } from 'hono/timing'
import type { TimingVariables } from 'hono/timing'
```

## 用法

```js
// 指定变量类型以推断 `c.get('metric')`：
type Variables = TimingVariables

const app = new Hono<{ Variables: Variables }>()

// 将中间件添加到您的路由器
app.use(timing());

app.get('/', async (c) => {

  // 添加自定义指标
  setMetric(c, 'region', 'europe-west3')

  // 添加带计时的自定义指标，必须以毫秒为单位
  setMetric(c, 'custom', 23.8, '我的自定义指标')

  // 启动一个新的计时器
  startTime(c, 'db');
  const data = await db.findMany(...);

  // 结束计时器
  endTime(c, 'db');

  return c.json({ response: data });
});
```

### 有条件地启用

```ts
const app = new Hono()

app.use(
  '*',
  timing({
    // c：请求的上下文
    enabled: (c) => c.req.method === 'POST',
  })
)
```

## 结果

![](/images/timing-example.png)

## 选项

### <Badge type="info" text="可选" /> total: `boolean`

显示总响应时间。默认值为 `true`。

### <Badge type="info" text="可选" /> enabled: `boolean` | `(c: Context) => boolean`

是否应将计时添加到标头。默认值为 `true`。

### <Badge type="info" text="可选" /> totalDescription: `boolean`

总响应时间的描述。默认值为 `Total Response Time`。

### <Badge type="info" text="可选" /> autoEnd: `boolean`

如果 `startTime()` 应在请求结束时自动结束。
如果禁用，则不会显示未手动结束的计时器。

### <Badge type="info" text="可选" /> crossOrigin: `boolean` | `string` | `(c: Context) => boolean | string`

此计时标头应可读的来源。

- 如果为 false，则仅来自当前来源。
- 如果为 true，则来自所有来源。
- 如果为字符串，则来自此域。多个域必须用逗号分隔。

默认值为 `false`。更多信息请参阅[文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin)。
