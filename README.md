# Cloudflare Workers - Claude AI 反向代理加速

使用 Cloudflare Workers 为 Claude AI 中转站（https://anyrouter.top）提供全球加速服务。

## 功能特性

✅ **智能缓存** - 对 GET 请求和静态资源进行缓存优化
✅ **流式传输支持** - 完美支持 Claude AI 的 SSE 流式响应
✅ **自动重试机制** - 网络异常时自动重试，提高稳定性
✅ **性能优化** - Keep-Alive 连接、请求超时控制
✅ **完整 CORS 支持** - 跨域请求无障碍
✅ **详细监控** - 响应时间、缓存状态等性能指标

---

## 部署步骤

### 方法一：GitHub 自动部署（推荐）⭐

将项目推送到 GitHub，每次代码更新时自动部署到 Cloudflare。

#### 1. 获取 Cloudflare API Token 和 Account ID

**获取 API Token：**
1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 点击右上角头像 → **My Profile** → **API Tokens**
3. 点击 **Create Token**
4. 选择 **Edit Cloudflare Workers** 模板
5. 点击 **Continue to summary** → **Create Token**
6. **复制并保存** Token（只显示一次）

**获取 Account ID：**
1. 在 Cloudflare Dashboard 点击左侧 **Workers & Pages**
2. 右侧会显示你的 **Account ID**，复制它

#### 2. 配置 GitHub Secrets

1. 在 GitHub 仓库页面，点击 **Settings** → **Secrets and variables** → **Actions**
2. 点击 **New repository secret**，添加两个密钥：
   - **Name**: `CLOUDFLARE_API_TOKEN`
     **Value**: 粘贴你的 API Token
   - **Name**: `CLOUDFLARE_ACCOUNT_ID`
     **Value**: 粘贴你的 Account ID

#### 3. 推送代码到 GitHub

```bash
# 初始化 Git 仓库（如果还没有）
git init

# 添加远程仓库
git remote add origin https://github.com/你的用户名/cloudflare-claude-proxy.git

# 添加文件
git add .

# 提交
git commit -m "feat: 初始化 Cloudflare Workers 代理项目"

# 推送到 GitHub
git push -u origin master
```

#### 4. 自动部署

推送成功后，GitHub Actions 会自动触发部署：

1. 在 GitHub 仓库页面，点击 **Actions** 标签
2. 查看部署进度
3. 部署成功后，会显示 Worker 访问地址

以后每次推送代码到 `master` 分支，都会自动部署到 Cloudflare！

---

### 方法二：使用 Wrangler CLI

#### 1. 安装 Wrangler CLI

```powershell
npm install -g wrangler
```

#### 2. 登录 Cloudflare 账号

```powershell
wrangler login
```

这会打开浏览器，授权 Wrangler 访问你的 Cloudflare 账号。

#### 3. 修改配置（可选）

编辑 `wrangler.toml` 文件，修改 Worker 名称：

```toml
name = "claude-proxy"  # 改成你想要的名称（全局唯一）
```

#### 4. 部署到 Cloudflare

```powershell
# 在项目目录下执行
cd c:\Users\YXL\Desktop\work\cloudflare-claude-proxy

# 部署
wrangler deploy
```

部署成功后，会显示 Worker 的访问地址：

```
✨ Deployed claude-proxy
   https://claude-proxy.你的用户名.workers.dev
```

---

### 方法三：使用 Cloudflare Dashboard

#### 1. 登录 Cloudflare Dashboard

访问 https://dash.cloudflare.com/ 并登录

#### 2. 创建 Worker

1. 点击左侧菜单 **Workers & Pages**
2. 点击 **Create Application**
3. 选择 **Create Worker**
4. 输入 Worker 名称（如 `claude-proxy`）
5. 点击 **Deploy**

#### 3. 编辑代码

1. 在创建的 Worker 页面，点击 **Quick Edit**
2. 删除默认代码
3. 复制 `worker.js` 的全部内容粘贴进去
4. 点击 **Save and Deploy**

#### 4. 获取访问地址

部署成功后，会显示 Worker 的访问地址，格式如下：

```
https://claude-proxy.你的用户名.workers.dev
```

---

## 自定义域名（可选）

如果你有自己的域名，可以绑定到 Worker：

### 1. 添加域名到 Cloudflare

确保你的域名已经添加到 Cloudflare 并使用 Cloudflare 的 DNS 服务。

### 2. 绑定自定义域名

在 Worker 页面：

1. 点击 **Settings** 标签
2. 找到 **Domains & Routes** 部分
3. 点击 **Add Custom Domain**
4. 输入域名（如 `claude-api.yourdomain.com`）
5. 点击 **Add Domain**

Cloudflare 会自动配置 DNS 记录和 SSL 证书。

---

## 使用方法

部署完成后，将原来的 API 地址替换为 Worker 地址即可。

### 示例

**原始地址：**
```
https://anyrouter.top/v1/chat/completions
```

**替换为 Worker 地址：**
```
https://claude-proxy.你的用户名.workers.dev/v1/chat/completions
```

或者自定义域名：
```
https://claude-api.yourdomain.com/v1/chat/completions
```

---

## 配置说明

### 修改目标地址

如果需要更改代理目标，编辑 `worker.js` 文件的第 17 行：

```javascript
const CONFIG = {
  targetUrl: 'https://anyrouter.top',  // 修改为其他地址
  // ...
};
```

### 调整缓存策略

编辑 `worker.js` 文件的第 23-30 行：

```javascript
cache: {
  defaultTtl: 300,      // GET 请求缓存时间（秒）
  staticTtl: 86400,     // 静态资源缓存时间（秒）
  staticExtensions: [   // 静态资源扩展名
    '.jpg', '.jpeg', '.png', '.gif',
    '.svg', '.css', '.js', '.woff',
    '.woff2', '.ttf', '.ico'
  ],
},
```

### 调整重试策略

编辑 `worker.js` 文件的第 33-37 行：

```javascript
retry: {
  maxRetries: 2,                    // 最大重试次数
  retryDelay: 1000,                 // 重试延迟（毫秒）
  retryableStatuses: [502, 503, 504], // 可重试的状态码
},
```

### 调整请求超时

编辑 `worker.js` 文件的第 40 行：

```javascript
timeout: 60000,  // 请求超时时间（毫秒），默认 60 秒
```

---

## 性能监控

Worker 会在响应头中添加性能指标：

```
X-Cache-Status: HIT/MISS       # 缓存状态
X-Response-Time: 123ms         # 响应时间
X-Proxy-By: Cloudflare-Workers # 代理标识
```

可以在浏览器开发者工具的 Network 标签中查看这些头信息。

---

## 查看日志

### 实时日志

```powershell
wrangler tail claude-proxy
```

### Dashboard 查看

1. 进入 Worker 页面
2. 点击 **Logs** 标签
3. 开启实时日志流

---

## 常见问题

### Q1: 部署后访问 403/404 错误

**解决方案：**
- 检查 Worker 是否部署成功
- 确认访问的 URL 是否正确
- 在 Dashboard 检查 Worker 是否启用

### Q2: 请求超时

**解决方案：**
- Workers 免费版有 10ms CPU 时间限制，如果请求过大可能超时
- 考虑升级到付费版（$5/月）获得 50ms CPU 时间
- 调整 `worker.js` 中的超时配置

### Q3: 缓存未生效

**解决方案：**
- 确认是 GET 请求（POST 等不会缓存）
- 检查目标服务器是否返回了 `Cache-Control: no-store`
- 查看响应头中的 `X-Cache-Status` 判断缓存状态

### Q4: CORS 错误

**解决方案：**
- 代码已包含完整 CORS 支持
- 如果还有问题，检查浏览器控制台的详细错误信息
- 确认预检请求（OPTIONS）是否正常

---

## 免费版限制

Cloudflare Workers 免费版提供：

- ✅ 每天 100,000 次请求
- ✅ 10ms CPU 时间/请求
- ✅ 全球 CDN 加速
- ✅ 无限流量

如果需要更高配额，可以升级到付费版：

- 💰 $5/月
- ✅ 1000 万次请求/月
- ✅ 50ms CPU 时间/请求
- ✅ 更高优先级

---

## 更新 Worker

修改代码后重新部署：

```powershell
wrangler deploy
```

或在 Dashboard 中编辑代码并保存。

---

## 安全建议

1. **不要在代码中硬编码敏感信息**（API Key 等）
2. **考虑添加速率限制**，防止滥用
3. **定期检查日志**，监控异常访问
4. **使用自定义域名 + HTTPS**，增强安全性

---

## 技术支持

如有问题，可以：

1. 查看 [Cloudflare Workers 官方文档](https://developers.cloudflare.com/workers/)
2. 访问 [Cloudflare 社区论坛](https://community.cloudflare.com/)
3. 检查本项目的 `worker.js` 注释说明

---

## 许可证

MIT License
