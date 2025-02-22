---
title: Cloudflare Queue
description: 使用 Hono 与 Cloudflare Queue。
---

# Cloudflare Queue

使用 Hono 与 [Cloudflare Queue](https://developers.cloudflare.com/queues/)。

:::code-group

```ts [index.ts]
import { Hono } from 'hono'

type Environment = {
  readonly ERROR_QUEUE: Queue<Error>
  readonly ERROR_BUCKET: R2Bucket
}

const app = new Hono<{
  Bindings: Environment
}>()

app.get('/', (c) => {
  if (Math.random() < 0.5) {
    return c.text('成功！')
  }
  throw new Error('失败！')
})

app.onError(async (err, c) => {
  await c.env.ERROR_QUEUE.send(err)
  return c.text(err.message, { status: 500 })
})

export default {
  fetch: app.fetch,
  async queue(batch: MessageBatch<Error>, env: Environment) {
    let file = ''
    for (const message of batch.messages) {
      const error = message.body
      file += error.stack || error.message || String(error)
      file += '\r\n'
    }
    await env.ERROR_BUCKET.put(`errors/${Date.now()}.log`, file)
  },
}
```

```toml [wrangler.toml]
name = "my-worker"

[[queues.producers]]
  queue = "my-queue"
  binding = "ERROR_QUEUE"

[[queues.consumers]]
  queue = "my-queue"
  max_batch_size = 100
  max_batch_timeout = 30

[[r2_buckets]]
  bucket_name = "my-bucket"
  binding = "ERROR_BUCKET"
```

:::

## 另请参阅

- [Cloudflare 队列](https://developers.cloudflare.com/queues/)
- [Cloudflare 队列与 R2 示例](https://developers.cloudflare.com/queues/examples/send-errors-to-r2/)
