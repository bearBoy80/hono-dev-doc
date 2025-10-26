# 常见问题解答

本指南是有关 Hono 的常见问题 (FAQ) 及其解决方法。

## Hono 有官方的 Renovate 配置吗？

Hono 团队目前不维护 [Renovate](https://github.com/renovatebot/renovate) 配置。
因此，请使用第三方 renovate-config，如下所示。

在您的 `renovate.json` 中：

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>shinGangan/renovate-config-hono" // [!code ++]
  ]
}
```

有关更多详细信息，请参阅 [renovate-config-hono](https://github.com/shinGangan/renovate-config-hono) 存储库。
