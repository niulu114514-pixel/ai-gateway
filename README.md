# AI Gateway

部署在 Cloudflare Workers 上的轻量 AI 网关。它把多个 OpenAI/Anthropic 兼容上游聚合为统一入口，并提供代理密钥、上游密钥健康检查、故障切换和网页管理后台。

## 功能

- 兼容 OpenAI 风格的 `/v1/*` 请求和流式响应。
- 支持 OpenAI、Anthropic 及其他兼容供应商。
- 多上游、多密钥轮换；失败密钥自动冷却并恢复。
- 生成 `sk_cf_*` 代理密钥，避免向客户端暴露真实上游密钥。
- 提供模型列表、供应商配置、密钥状态和代理密钥管理页面。
- 默认提供 OpenCode Zen 免费模型配置，可在后台修改。

## 技术栈

- Cloudflare Workers
- Hono
- Workers KV（绑定名：`KV`）
- TypeScript

## 快速开始

```bash
cd ai-gateway
npm install
npm run dev
```

本地敏感变量可写入 `.dev.vars`：

```dotenv
ADMIN_USERNAME=admin
ADMIN_PASSWORD=请替换为强密码
```

## 部署

1. 创建 KV 命名空间，并把得到的命名空间配置到 `wrangler.toml` 的 `KV` 绑定中。

   ```bash
   npx wrangler kv namespace create KV
   ```

2. 设置管理凭据。

   ```bash
   npx wrangler secret put ADMIN_USERNAME
   npx wrangler secret put ADMIN_PASSWORD
   ```

3. 部署 Worker。

   ```bash
   npm run deploy
   ```

如果当前 Wrangler 能按资源名自动解析 KV，也仍建议确认部署输出中的绑定与目标命名空间一致。

## 使用

管理后台：

```text
https://<你的域名>/admin
```

OpenAI 兼容调用示例：

```bash
curl https://<你的域名>/v1/chat/completions \
  -H "Authorization: Bearer sk_cf_你的代理密钥" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4-flash-free","messages":[{"role":"user","content":"你好"}]}'
```

主要端点：

| 路径 | 用途 |
| --- | --- |
| `/` | 服务首页 |
| `/admin` | 管理后台 |
| `/v1/models` | 模型列表 |
| `/v1/*` | OpenAI/Anthropic 兼容代理入口 |

## 配置项

| 名称 | 类型 | 说明 |
| --- | --- | --- |
| `KV` | KV 绑定 | 保存供应商、上游密钥和代理密钥 |
| `ADMIN_USERNAME` | Secret/变量 | 管理员用户名 |
| `ADMIN_PASSWORD` | Secret | 管理员密码 |
| `OPENCODE_MIRRORS_URL` | 可选变量 | 自定义 OpenCode 镜像地址 |

## 脚本

```bash
npm run dev       # 本地开发
npm run build     # Wrangler dry-run
npm run deploy    # 部署到 Cloudflare
```

## 安全建议

- 管理密码和真实上游密钥只通过 Secret 或管理后台配置。
- 客户端只使用代理密钥；发现泄露时在后台立即吊销。
- 生产环境建议绑定自定义域名，并为管理后台增加 Cloudflare Access 等额外保护。
