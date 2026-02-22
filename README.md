# Cloudflare Worker 静态站点管理器

这是一个部署在 Cloudflare Workers 上的轻量静态站点托管工具。  
通过 `/admin` 管理页面上传 HTML 和静态资源，上传完成后即可通过 `/site/{slug}` 访问。

## 功能特性

- 管理页面 `/admin`（口令登录）
- 上传 HTML（文件或直接粘贴）
- 上传静态资源并保留相对路径
- 自动生成站点 URL
- 站点列表展示与一键访问
- KV 存储（适合小文件/少量资源）

## 路由与访问

- 管理页面：`/admin`
- 登录提交：`POST /admin/login`
- 上传接口：`POST /admin/upload`
- 站点列表：`GET /admin/list`
- 站点首页：`/site/{slug}`
- 资源文件：`/site/{slug}/{path}`

示例：

```
https://your-worker.example.com/site/my-site
https://your-worker.example.com/site/my-site/assets/style.css
```

## 存储结构（KV 设计）

KV 中的 key 组织：

- 站点清单：`site:{slug}:manifest`
- 资源文件：`site:{slug}:file:{path}`

清单包含：

- `slug`
- `createdAt`
- `totalBytes`
- `files[]`（path / size / contentType）

## 运行环境

- Node.js 18+（推荐）
- Wrangler CLI（`npm install` 自动安装）
- Cloudflare 账号与 KV 命名空间

## 快速开始（Wrangler CLI）

### 1. 安装依赖

```bash
npm install
```

### 2. 创建 KV 命名空间

```bash
npx wrangler kv:namespace create "SITE_KV"
npx wrangler kv:namespace create "SITE_KV" --preview
```

把返回的 ID 写入 `wrangler.toml`：

```
[[kv_namespaces]]
binding = "SITE_KV"
id = "REPLACE_WITH_KV_ID"
preview_id = "REPLACE_WITH_KV_PREVIEW_ID"
```

### 3. 设置 secrets

```bash
npx wrangler secret put ADMIN_PASS
npx wrangler secret put SESSION_SECRET
```

> 提示：如果命令提示 “There doesn't seem to be a Worker...”，输入 `y` 让 Wrangler 自动创建 Worker。

### 4. 本地开发

```bash
npm run dev
```

### 5. 部署

```bash
npm run deploy
```

## 使用说明

1. 打开 `/admin` 并输入 `ADMIN_PASS` 登录。
2. 填写 `slug`：
   - 规则：3–32 位
   - 仅允许：小写字母、数字、短横线（`a-z0-9-`）
3. 上传 HTML：
   - 上传 `index.html` 文件  
   - 或者在文本框里粘贴 HTML
4. 上传静态资源（可选）：
   - 选择文件夹上传可以保留相对路径（依赖浏览器 `webkitdirectory`）
5. 上传成功后页面会显示可访问 URL。

## 配置参数说明

`wrangler.toml`：

- `name`：Worker 名称
- `main`：入口文件（`src/index.js`）
- `compatibility_date`：兼容日期
- `SITE_KV`：KV 绑定

Secrets：

- `ADMIN_PASS`：管理页面登录口令
- `SESSION_SECRET`：会话签名密钥（HMAC）

可选变量：

- `MAX_UPLOAD_BYTES`：单次上传总大小限制（默认 10MB）

## 上传限制与建议

- KV 适合小文件/少量资源。
- 默认限制 10MB，可以通过 `MAX_UPLOAD_BYTES` 调整。
- 不建议放大体积多媒体或大站点。

## 常见问题

**1. secret 写入失败，提示 Worker 不存在？**  
在提示时输入 `y` 创建 Worker，或先 `npx wrangler deploy`。

**2. 上传成功但访问 404？**  
请确认 `slug` 是否正确、资源路径是否与上传结构一致。

**3. 登录后仍被重定向到登录页？**  
检查 `SESSION_SECRET` 是否设置，浏览器是否禁用 Cookie。

## 安全建议

- 使用强口令作为 `ADMIN_PASS`
- 不在公共网络暴露 `/admin`（可使用 Cloudflare Access 保护）
- 定期轮换 `SESSION_SECRET`

## 清理数据

如需删除某个站点的数据，可用 `wrangler kv:key delete` 删除对应 key：  
例如删除清单与全部文件（需要自行遍历路径）：

```
npx wrangler kv:key delete "site:your-slug:manifest" --binding SITE_KV
```


# cf-worker-html 
 
 
