# 修复恶意注册问题的完整指南 (无需数据库权限)

## 问题分析

通过代码分析，发现系统默认关闭注册验证，这就是导致恶意注册的原因：

1. **系统默认设置**：系统初始化时默认 `register_verify = 1` (关闭)
2. **COUNT 模式漏洞**：即使开启COUNT模式，前N次注册不需要验证

## 解决方案

### 步骤 1：更新代码（推荐）

如果可以重新部署代码，将以下文件修改后重新部署：

1. **修改初始化默认设置**：[init.js](file:///workspace/mail-worker/src/init/init.js#L483)
   - 将 `register_verify` 和 `add_email_verify` 从 1 改为 0

2. **修复 COUNT 模式验证逻辑**：[login-service.js](file:///workspace/mail-worker/src/service/login-service.js#L111-L114)
   - COUNT 模式下也始终要求验证

3. **添加 Turnstile 密钥检查**：[turnstile-service.js](file:///workspace/mail-worker/src/service/turnstile-service.js#L15-L21)
   - 确保密钥未配置时会报错

### 步骤 2：通过 API 更新现有设置（无需数据库权限）

如果你已经部署了代码，且没有数据库权限，可以通过 API 更新设置：

#### 2.1 获取管理员 Token

1. 登录管理员账号
2. 打开浏览器开发者工具（F12）
3. 查看本地存储（Local Storage）
4. 找到 `token` 字段，复制其值

#### 2.2 发送 API 请求更新设置

使用 curl 或 Postman 发送以下请求：

```bash
# 使用 curl 命令
curl -X PUT "https://your-mail-app.workers.dev/setting/set" \
  -H "Authorization: YOUR_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "registerVerify": 0,
    "addEmailVerify": 0
  }'

# 或者添加 Turnstile 密钥（如果还没配置）
curl -X PUT "https://your-mail-app.workers.dev/setting/set" \
  -H "Authorization: YOUR_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "registerVerify": 0,
    "addEmailVerify": 0,
    "siteKey": "YOUR_TURNSTILE_SITE_KEY",
    "secretKey": "YOUR_TURNSTILE_SECRET_KEY"
  }'
```

**参数说明**：
- `YOUR_ADMIN_TOKEN`：从本地存储复制的 token
- `YOUR_TURNSTILE_SITE_KEY`：从 Cloudflare Turnstile 获取的 Site Key
- `YOUR_TURNSTILE_SECRET_KEY`：从 Cloudflare Turnstile 获取的 Secret Key

#### 2.3 验证设置是否更新成功

发送查询请求验证：

```bash
curl -H "Authorization: YOUR_ADMIN_TOKEN" "https://your-mail-app.workers.dev/setting/query"
```

### 步骤 3：清理恶意用户

1. 登录管理员后台
2. 进入「用户管理」页面
3. 批量删除可疑的恶意注册账号
4. 检查这些账号的注册 IP，识别恶意来源

### 步骤 4：额外安全措施

1. **启用注册密钥（Reg Key）**：
   - 在系统设置中开启 Reg Key 功能
   - 生成注册密钥，只发给可信用户

2. **临时关闭注册**（如果恶意注册严重）：
   - 在系统设置中关闭注册功能
   - 只允许管理员通过后台添加用户

3. **配置 Turnstile 高级设置**：
   - 登录 Cloudflare 控制台
   - 调整 Turnstile 的难度设置为「Moderate」或「Difficult」

## 验证修复效果

1. 打开隐私/无痕浏览窗口
2. 尝试注册新账号
3. **应该会看到 Turnstile 人机验证**，不通过验证无法注册

## 常见问题

### Q: 没有 Turnstile 密钥怎么办？

**A:** 免费注册 Cloudflare 账号，然后：
1. 登录 Cloudflare 控制台
2. 进入 Turnstile 服务
3. 创建新的站点，获取 Site Key 和 Secret Key

### Q: API 请求返回 401 错误

**A:** 检查管理员 token 是否正确，确保账号有管理员权限

### Q: 还是有恶意注册

**A:** 检查是否还有其他注册入口，或者考虑启用注册密钥功能