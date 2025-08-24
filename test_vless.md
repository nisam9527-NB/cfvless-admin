# 🧪 VLESS 代理功能测试指南

## 📋 测试步骤

### 1. 部署验证
```bash
# 部署到 Cloudflare Pages
wrangler pages deploy ./ --project-name=subscription-manager

# 初始化数据库
wrangler d1 execute subscription-db --remote --file=d1_init.sql
```

### 2. 注册用户并获取源节点
1. 访问您的 Pages URL（如：`https://fq8-2dz.pages.dev`）
2. 注册新用户
3. 登录后查看"源节点管理"标签页
4. 复制默认的 NAT64 或 ProxyIP 源节点

### 3. 测试 VLESS 连接

#### 示例源节点格式：
```
vless://f72cfff5-be11-4890-acb8-d27953d86472@fq8-2dz.pages.dev:443?encryption=none&security=tls&type=ws&host=fq8-2dz.pages.dev&path=%2F%3Fed%3D2560&sni=fq8-2dz.pages.dev&fp=random#NAT64_fq8-2dz.pages.dev
```

#### 关键参数说明：
- **UUID**: `f72cfff5-be11-4890-acb8-d27953d86472` - 用户的唯一标识符
- **域名**: `fq8-2dz.pages.dev` - 您的 Pages 域名
- **端口**: `443` - HTTPS 端口
- **路径**: `%2F%3Fed%3D2560` - 即 `/?ed=2560`
- **安全**: `tls` - 使用 TLS 加密
- **类型**: `ws` - WebSocket 协议

### 4. 客户端配置测试

#### 使用 v2rayN (Windows)：
1. 添加服务器
2. 选择 VLESS 协议
3. 粘贴完整的 vless:// 链接
4. 或手动填写各项参数

#### 使用 v2rayNG (Android)：
1. 点击 "+" 添加配置
2. 选择"从剪贴板导入"
3. 粘贴 vless:// 链接

#### 使用 Qv2ray/v2rayA (Linux)：
1. 导入 vless:// 链接
2. 或手动配置各项参数

### 5. 功能验证

#### 基本连接测试：
```bash
# 通过代理访问测试网站
curl --proxy socks5://127.0.0.1:1080 https://www.google.com
curl --proxy socks5://127.0.0.1:1080 https://ipinfo.io
```

#### NAT64 功能测试：
- 测试访问 IPv4 网站
- 验证 NAT64 自动转换功能
- 检查连接失败时的自动重试

#### ProxyIP 功能测试：
- 配置自定义 ProxyIP
- 测试优选 IP 功能

## 🔍 故障排除

### 常见问题：

#### 1. 连接失败
- **检查 UUID**：确保 UUID 与数据库中的用户 UUID 匹配
- **检查域名**：确保使用正确的 Pages 域名
- **检查路径**：确保路径为 `/?ed=2560`

#### 2. UUID 不匹配
```sql
-- 查询用户的 UUID
SELECT username, user_uuid FROM users;

-- 更新用户的 UUID（如果需要）
UPDATE users SET user_uuid = 'new-uuid' WHERE username = 'your-username';
```

#### 3. WebSocket 升级失败
- 检查 Cloudflare Pages 的 Functions 配置
- 确保 D1 和 KV 绑定正确
- 查看 Cloudflare 控制台的实时日志

#### 4. 代理不工作
- 检查客户端配置是否正确
- 验证防火墙设置
- 测试网络连接

## 📊 调试信息

### 查看日志：
```bash
# 查看 Pages 部署日志
wrangler pages deployment tail --project-name=subscription-manager

# 查看 D1 数据库内容
wrangler d1 execute subscription-db --remote --command="SELECT * FROM users LIMIT 5;"
```

### 测试数据库连接：
```bash
curl https://your-pages-url.pages.dev/api/debug
```

### 测试 UUID 验证：
在浏览器开发者工具中检查 WebSocket 连接和错误信息。

## ✅ 成功标志

如果一切正常，您应该能够：
1. ✅ 成功注册用户并看到默认源节点
2. ✅ 复制源节点到客户端
3. ✅ 建立 WebSocket 连接
4. ✅ 通过代理访问网站
5. ✅ NAT64 自动转换功能工作
6. ✅ 在节点池中看到源节点

## 🚀 下一步

测试成功后，您可以：
- 配置自定义源节点
- 使用节点生成器扩展更多节点
- 管理节点池和标签
- 导出订阅链接