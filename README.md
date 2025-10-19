# FileCodeBox - Cloudflare Workers 版本

基于 Cloudflare Workers 的匿名文件快递柜，支持文件和文本的临时分享。

## ✨ 特性

- 📦 支持文件和文本分享
- 🔐 6位数字提取码
- ⏰ 灵活的过期时间设置（分钟/小时/天/永久）
- 🗄️ 使用 Cloudflare KV + R2 作为存储后端
- 🚀 基于 Cloudflare Workers 边缘计算
- 📱 响应式设计，支持移动端
- 🎯 支持拖拽上传文件
- 🔒 内置速率限制保护
- 🧹 自动清理过期文件

## 📋 前置要求

1. **Cloudflare 账号**
   - 免费账号即可
   - 需要获取 API Token 和 Account ID

2. **GitHub 账号**（用于 GitHub Actions 自动部署）

## 🚀 快速开始

### 1️⃣ Fork 本仓库

点击右上角的 Fork 按钮，将仓库复制到你的 GitHub 账号下。

### 2️⃣ 获取 Cloudflare 凭证

#### 获取 Account ID
1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
2. 在右侧找到你的 **Account ID**
3. 复制保存

#### 获取 API Token
1. 访问 [API Tokens 页面](https://dash.cloudflare.com/profile/api-tokens)
2. 点击 "Create Token"
3. 使用 "Edit Cloudflare Workers" 模板
4. 或自定义权限：
   - Account - Workers KV Storage - Edit
   - Account - Workers Scripts - Edit
   - Account - R2 Storage - Edit
5. 创建后复制保存（只显示一次）

### 3️⃣ 配置 GitHub Secrets

在你 Fork 的仓库中设置以下 Secrets：

1. 进入仓库的 `Settings` → `Secrets and variables` → `Actions`
2. 点击 `New repository secret` 添加以下变量：

#### 必需的 Secrets

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API Token | `xxxx-your-api-token-xxxx` |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare Account ID | `abc123def456` |

#### 可选的 Secrets

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `PERMANENT_PASSWORD` | 永久保存功能密码 | `123456` |

### 4️⃣ 触发部署

有两种方式触发部署：

#### 方法一：推送代码
```bash
git commit --allow-empty -m "Trigger deployment"
git push
```

#### 方法二：手动触发
1. 进入仓库的 `Actions` 标签
2. 选择 "Deploy to Cloudflare Workers"
3. 点击 "Run workflow"

### 5️⃣ 查看部署结果

1. 部署完成后，访问 [Cloudflare Workers Dashboard](https://dash.cloudflare.com/?to=/:account/workers)
2. 找到名为 `filecodebox` 的 Worker
3. 点击查看部署的 URL（如：`https://filecodebox.your-subdomain.workers.dev`）

## ✨ 自动化功能

GitHub Action 会自动完成以下操作：

- ✅ 自动创建 KV Namespace（如果不存在）
- ✅ 自动创建 R2 Bucket（如果不存在）
- ✅ 自动获取资源 ID 并配置
- ✅ 自动设置永久密码（默认：123456）
- ✅ 自动部署到 Cloudflare Workers

## ⚙️ 可选配置

### 自定义环境变量

在 `wrangler.toml` 文件中可以修改以下配置：

```toml
[vars]
# 文件大小限制（MB 或字节）
MAX_FILE_SIZE = "90"              # 默认 90MB
MAX_TEXT_SIZE = "1"               # 默认 1MB

# 二维码生成服务
QR_API = "https://api.qrserver.com/v1/create-qr-code/"

# 首次声明弹窗间隔（小时）
NOTICE_TTL_HOURS = "24"           # 默认 24 小时

# 速率限制（每分钟请求数）
UPLOAD_FILE_RPM = "10"            # 上传文件限制
UPLOAD_TEXT_RPM = "20"            # 上传文本限制
VERIFY_PERM_RPM = "20"            # 验证密码限制
GET_INFO_RPM = "120"              # 获取信息限制
DOWNLOAD_RPM = "60"               # 下载限制
```

### 自动清理任务

系统会自动清理过期文件，清理频率在 `wrangler.toml` 中配置：

```toml
[triggers]
crons = ["*/5 * * * *"]  # 每5分钟运行一次
```

可以修改为其他 Cron 表达式，例如：
- `0 * * * *` - 每小时运行一次
- `0 0 * * *` - 每天凌晨运行一次

## 🔧 本地开发

### 前置要求
- Node.js 18+
- npm 8+

### 安装依赖
```bash
npm install
```

### 配置本地环境
1. 复制 `wrangler.toml` 并替换占位符：
   ```bash
   cp wrangler.toml wrangler.local.toml
   ```

2. 在 Cloudflare Dashboard 中创建 KV 和 R2 资源：
   ```bash
   # 创建 KV Namespace
   wrangler kv namespace create FILECODEBOX_KV
   wrangler kv namespace create FILECODEBOX_KV --preview
   
   # 创建 R2 Bucket
   wrangler r2 bucket create filecodebox-storage
   ```

3. 将获取到的 ID 替换到 `wrangler.local.toml` 中

4. 设置永久密码：
   ```bash
   wrangler secret put PERMANENT_PASSWORD --config wrangler.local.toml
   ```

### 本地运行
```bash
npm run dev
```

### 部署到 Cloudflare
```bash
npm run deploy
```

## 📝 使用说明

### 发送文件
1. 选择"发文件"标签
2. 点击选择文件或**直接拖拽文件**到上传区域
3. 设置过期时间
4. 点击"生成提取码"
5. 保存6位数字提取码

### 发送文本
1. 选择"发文件"标签，然后点击"文本"
2. 输入要分享的文本内容
3. 设置过期时间
4. 点击"生成提取码"

### 接收文件
1. 选择"取文件"标签
2. 输入6位数字提取码
3. 查看或下载文件

### 永久保存
- 选择"永久"选项需要输入密码
- 默认密码：`123456`
- 可通过 `PERMANENT_PASSWORD` Secret 自定义

## 🛡️ 安全性

- ✅ 内置速率限制，防止滥用
- ✅ 自动清理过期文件
- ✅ 敏感信息通过 Secrets 加密存储
- ✅ 支持自定义永久保存密码
- ✅ 基于 Cloudflare 全球边缘网络
- ⚠️ 建议定期更换 API Token 和密码

## 🐛 常见问题

### 部署失败？
1. 检查 Secrets 是否正确填写
2. 确认 API Token 权限是否足够（需要包含 Workers、KV、R2 权限）
3. 查看 GitHub Actions 日志获取详细错误

### KV 或 R2 资源创建失败？
1. 确认 Cloudflare 账号已验证
2. 检查 API Token 是否包含相应权限
3. 手动创建资源后更新 `wrangler.toml` 配置

### 文件上传失败？
1. 检查文件大小是否超过限制（默认90MB）
2. 确认网络连接稳定
3. 查看浏览器控制台错误信息

### 拖拽上传不工作？
1. 确认使用现代浏览器（Chrome、Firefox、Safari、Edge）
2. 检查是否有 JavaScript 错误
3. 尝试刷新页面重新加载

### 永久保存密码错误？
1. 检查是否设置了 `PERMANENT_PASSWORD` Secret
2. 默认密码为 `123456`
3. 重新部署后生效

## 📊 资源配置

### Cloudflare 免费额度
- **Workers**: 100,000 请求/天
- **KV**: 100,000 读取/天，1,000 写入/天
- **R2**: 10GB 存储，1,000,000 A类操作/月

### 推荐配置
- 小型团队：默认配置即可
- 中型使用：考虑升级到 Workers Paid 计划
- 大型部署：建议使用 Cloudflare Pro 计划

## 📄 许可证

MIT License

## 🙏 致谢

- [Hono](https://hono.dev/) - Web 框架
- [Cloudflare Workers](https://workers.cloudflare.com/) - 边缘计算平台
- [FileCodeBox](https://github.com/vastsa/FileCodeBox) - 原始项目灵感

## 📮 反馈与贡献

欢迎提交 Issue 和 Pull Request！

---

**注意事项：**
- 请遵守当地法律法规，不要上传违法内容
- 建议定期备份重要文件
- 免费版 Cloudflare Workers 有请求限制，请合理使用
- 大文件建议使用专业的文件存储服务