# 🔧 GitHub Actions 部署问题修复指南

## 问题分析

您遇到的错误：
```
✘ [ERROR] A request to the Cloudflare API (/accounts/***/pages/projects/cfvless-admin) failed.
Project not found. The specified project name does not match any of your existing projects. [code: 8000007]
```

**根本原因**：GitHub Actions 尝试部署到一个不存在的 Cloudflare Pages 项目。

## 🚀 解决方案（两种方式）

### 方案一：手动创建项目（推荐，最稳定）

#### 步骤 1：在 Cloudflare Dashboard 创建项目

1. **登录 Cloudflare Dashboard**
   - 访问：https://dash.cloudflare.com/
   - 进入 **Workers 和 Pages** → **Pages**

2. **创建新项目**
   - 点击 **创建应用程序**
   - 选择 **连接到 Git**
   - 选择您 fork 的 `cfvless-admin` 仓库

3. **配置构建设置**
   ```
   框架预设: None
   构建命令: (留空)
   构建输出目录: (留空)
   根目录: (留空)
   ```

4. **保存并部署**
   - 点击 **保存并部署**
   - 等待首次部署完成（可能会失败，这是正常的）

#### 步骤 2：触发 GitHub Actions

1. **推送代码触发部署**
   ```bash
   git add .
   git commit -m "修复部署配置"
   git push origin main
   ```

2. **或手动触发**
   - 进入 GitHub 仓库 → Actions
   - 选择 "Deploy to Cloudflare Pages (Fixed)"
   - 点击 "Run workflow"

### 方案二：自动创建项目（已优化）

我已经更新了 GitHub Actions 工作流，现在它会：

1. **尝试部署到现有项目**
2. **如果项目不存在，自动创建新项目**
3. **重新部署到新创建的项目**

直接推送代码即可：
```bash
git add .
git commit -m "使用自动创建项目的部署流程"
git push origin main
```

## 📋 已完成的配置修复

### 1. 修复了 wrangler.toml 配置

**之前的问题**：
- 使用了 Workers 的配置格式
- 缺少 `pages_build_output_dir` 字段

**现在的配置**：
```toml
name = "cfvless-admin"
compatibility_date = "2024-01-01"

[pages_build]
pages_build_output_dir = "./"

[vars]
ENVIRONMENT = "production"

[[d1_databases]]
binding = "DB"
database_name = "subscription-db"

[[kv_namespaces]]
binding = "subscription"
```

### 2. 优化了 GitHub Actions 工作流

**新增功能**：
- ✅ 自动创建不存在的 Pages 项目
- ✅ 智能重试机制
- ✅ 详细的错误处理和日志
- ✅ 环境验证

## ⚠️ 部署后必须完成的配置

无论使用哪种方案，部署成功后都需要在 Cloudflare Dashboard 中手动绑定资源：

### 1. 绑定 D1 数据库

1. **进入 Pages 项目设置**
   - Cloudflare Dashboard → Workers 和 Pages → Pages → cfvless-admin
   - 点击 **设置** → **函数**

2. **添加 D1 绑定**
   - 在 **D1 数据库绑定** 部分点击 **添加绑定**
   - 变量名：`DB`
   - D1 数据库：选择 `subscription-db`

### 2. 绑定 KV 命名空间

1. **添加 KV 绑定**
   - 在 **KV 命名空间绑定** 部分点击 **添加绑定**
   - 变量名：`subscription`
   - KV 命名空间：选择对应的命名空间

### 3. 保存并重新部署

- 点击 **保存**
- Pages 会自动触发重新部署
- 等待部署完成

## 🔍 验证部署成功

1. **访问应用**
   - URL: https://cfvless-admin.pages.dev
   - 检查页面是否正常加载

2. **检查功能**
   - 尝试注册/登录功能
   - 检查数据库连接是否正常

3. **查看日志**
   - Cloudflare Dashboard → Pages → cfvless-admin → 函数
   - 查看实时日志确认无错误

## 🎯 推荐流程

**对于新用户，推荐使用方案一**：
1. 手动在 Dashboard 创建项目（5分钟）
2. 推送代码触发自动部署
3. 手动绑定 D1 和 KV 资源
4. 验证功能正常

这样可以确保最高的成功率和最少的问题。

## 📞 如果仍有问题

如果按照以上步骤仍然遇到问题，请提供：

1. **完整的 GitHub Actions 日志**
2. **Cloudflare Dashboard 中的错误信息**
3. **具体的错误步骤**

我会进一步协助您解决问题。