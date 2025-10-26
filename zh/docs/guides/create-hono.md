# Create-hono

`create-hono` 支持的命令行选项——当您运行 `npm create hono@latest`、`npx create-hono@latest` 或 `pnpm create hono@latest` 时运行的项目初始化程序。

> [!NOTE]
> **为什么会有这个页面？** 安装/快速入门示例通常会显示一个最小的 `npm create hono@latest my-app` 命令。`create-hono` 支持几个有用的标志，您可以传递这些标志来自动化和自定义项目创建（选择模板、跳过提示、选择包管理器、使用本地缓存等）。

## 传递参数：

当您使用 `npm create`（或 `npx`）时，用于初始化脚本的参数必须放在 `--` **之后**。`--` 之后的任何内容都会转发给初始化程序。

::: code-group

```sh [npm]
# 将参数转发给 create-hono（npm 需要 `--`）
npm create hono@latest my-app -- --template cloudflare-workers
```

```sh [yarn]
# "--template cloudflare-workers" 选择 Cloudflare Workers 模板
yarn create hono my-app --template cloudflare-workers
```

```sh [pnpm]
# "--template cloudflare-workers" 选择 Cloudflare Workers 模板
pnpm create hono@latest my-app --template cloudflare-workers
```

```sh [bun]
# "--template cloudflare-workers" 选择 Cloudflare Workers 模板
bun create hono@latest my-app --template cloudflare-workers
```

```sh [deno]
# "--template cloudflare-workers" 选择 Cloudflare Workers 模板
deno init --npm hono@latest my-app --template cloudflare-workers
```

:::

## 常用参数

| 参数                | 描述                                                                                                                                      | 示例                         |
| :---------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------ |
| `--template <模板>` | 选择一个入门模板并跳过交互式模板提示。模板可能包括 `bun`、`cloudflare-workers`、`vercel` 等名称。 | `--template cloudflare-workers` |
| `--install`             | 创建模板后自动安装依赖项。                                                                                | `--install`                     |
| `--pm <包管理器>` | 指定安装依赖项时要运行的包管理器。常用值：`npm`、`pnpm`、`yarn`。                                         | `--pm pnpm`                     |
| `--offline`             | 使用本地缓存/模板，而不是获取最新的远程模板。对于离线环境或确定性的本地运行很有用。      | `--offline`                     |

> [!NOTE]
> 模板和可用选项的确切集合由 `create-hono` 项目维护。本文档页面总结了最常用的标志——有关完整的权威参考，请参阅下面链接的存储库。

## 示例流程

### 最小化，交互式

```bash
npm create hono@latest my-app
```

这将提示您输入模板和选项。

### 非交互式，选择模板和包管理器

```bash
npm create hono@latest my-app -- --template vercel --pm npm --install
```

这将使用 `vercel` 模板创建 `my-app`，使用 `npm` 安装依赖项，并跳过交互式提示。

### 使用离线缓存（无网络）

```bash
pnpm create hono@latest my-app --template deno --offline
```

## 故障排除和提示

- 如果某个选项似乎无法识别，请确保在使用 `npm create` / `npx` 时使用 `--` 将其转发。
- 要查看最新的模板和标志列表，请查阅 `create-hono` 存储库或在本地运行初始化程序并遵循其帮助输出。

## 链接和参考

- `create-hono` 存储库：[create-hono](https://github.com/honojs/create-hono)
