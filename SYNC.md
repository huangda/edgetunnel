# 自动同步说明

这个仓库是 [cmliu/edgetunnel](https://github.com/cmliu/edgetunnel) 的 fork，
只用来给 Cloudflare Worker **hhhh** 做自动更新。

## 工作方式

`.github/workflows/sync-deploy.yml` 每天 04:00（北京时间）执行一次，也可以手动触发：

1. 从上游 `cmliu/edgetunnel` 的 `main` 分支拉取最新的 `_worker.js`；
2. 计算 sha256，和 `.upstream-sha256` 比较（没变化就跳过，避免无意义地刷版本）；
3. 有变化时用 `wrangler deploy` 部署到 Cloudflare Worker `hhhh`；
4. 把新的 `_worker.js` 和 sha256 提交回本仓库，方便追溯当前线上跑的是哪一版。

如果想随时强制重新部署一次：Actions → 同步上游 edgetunnel 并部署 → Run workflow → 勾选 `force`。

## 需要的仓库配置

| 名称 | 类型 | 说明 |
| --- | --- | --- |
| `CLOUDFLARE_API_TOKEN` | Secret | Cloudflare API Token，权限：Workers Scripts:Edit、Workers KV Storage:Edit、Account Settings:Read |
| `CLOUDFLARE_ACCOUNT_ID` | Variable | `bbd703be74eff26d29ce628c751705ba` |

## 部署目标

`wrangler.toml` 里定义了目标 Worker，和上游仓库自带的 `wrangler.toml` 不同，**不要**用上游的覆盖它：

- Worker 名称：`hhhh`
- 自定义域名：`dada2006.ccwu.cc`
- 备用域名：`https://hhhh.huangda1995.workers.dev`
- KV 绑定：`KV` → `b6f9573a44ea4a80bb7f0defdb7829db`
- 变量：`HOST = dada2006.ccwu.cc`
- `ADMIN`（后台密码）只在 Cloudflare 控制台维护：
  Workers & Pages → hhhh → 设置 → 变量和密钥。**不要**写进仓库。
  配置里的 `keep_vars = true` 会让每次 CI 部署自动保留控制台里的这个值。

### 想改后台密码 / 订阅 token

直接在 Cloudflare 控制台改 `ADMIN` 的值并保存，Worker 立即生效。
注意代码里 UUID 和订阅 token 都是由 `ADMIN + KEY` 派生出来的，
改完密码后订阅地址会变（旧订阅地址会失效），需要到 `/admin` 后台重新复制。

## 注意事项

自动跟随上游意味着上游的改动会直接上线。如果哪天想固定在某个版本，
把工作流里 `UPSTREAM_BRANCH` 改成一个具体的 commit sha 即可。
