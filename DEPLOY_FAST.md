# 快速部署指南（免费资源优先）

> **目标**：最快上线可用、尽量免费、可持续运行  
> **推荐优先级**：Railway（最省事）> 本地 / 云主机 Docker（最稳）

---

## 0. 重要说明

1. **Railway 可以部署本项目**，但其免费额度与策略可能随时间调整，请以 [Railway 官方定价页](https://railway.app/pricing) 为准。
2. 本项目包含浏览器自动化能力，请确保你的使用**符合目标平台服务条款与当地法律法规**。
3. 本文档只覆盖"尽快跑起来"的最短路径，不保证在所有环境下开箱即用。

---

## 1. 方案 A：Railway 免费部署（优先推荐）

### 1.1 前置条件

- GitHub 账号已 Fork 或拥有本仓库的访问权限
- Railway 账号（可用 GitHub 登录）：<https://railway.app/>

### 1.2 从 GitHub 导入仓库

1. 登录 Railway，点击 **New Project**
2. 选择 **Deploy from GitHub repo**
3. 授权 Railway 访问你的 GitHub 账号，然后选择仓库 `zhikanyeye/kiro-auto`（或你的 Fork）
4. Railway 会自动识别 Node.js 项目并触发首次构建

### 1.3 设置启动命令（Start Command）

本项目是**任务型脚本**而非常驻 HTTP 服务，建议在 Railway 的 **Settings → Deploy → Start Command** 中填写：

```bash
npm run register -- --count 1 --non-interactive
```

> 若只需验证部署流程，先用单次注册命令。后续可根据需求改为其他 npm scripts。

### 1.4 最小环境变量配置

在 Railway 项目的 **Variables** 面板中，按需填写以下变量：

| 变量名 | 说明 | 是否必填 |
|--------|------|----------|
| `HTTP_PROXY` | HTTP 代理地址，如 `http://127.0.0.1:7890` | 按需 |
| `HTTPS_PROXY` | HTTPS 代理地址 | 按需 |

> 如果你的网络可以直连 AWS，代理变量可以先不填；出现网络错误时再补齐。

### 1.5 查看日志与验证部署

1. Railway → 项目页面 → **Deployments**，点击最新一次部署
2. 查看 **Build Logs**：确认依赖安装、浏览器安装均成功
3. 查看 **Runtime Logs**：出现脚本执行完成的日志即表示部署可用

---

## 2. 免费数据库（如需持久化，可选）

当前项目默认将结果写入本地文件（`show/results.json`），Railway 的容器重新部署后文件会重置。  
若你需要持久化注册结果，可选以下免费层数据库：

| 服务 | 类型 | 免费层说明 |
|------|------|------------|
| [Supabase](https://supabase.com/) | PostgreSQL | 2 个免费项目，500 MB 存储 |
| [Neon](https://neon.tech/) | PostgreSQL | 1 个免费项目，512 MB 存储 |

**建议**：先不接数据库，把部署流程跑通后再考虑持久化。

---

## 3. 回退方案

### 方案 B：本地 Docker / Node.js 运行

当 Railway 免费额度不足或策略变化时，可以直接在本地或任意 Linux 主机上运行：

```bash
# 1. 克隆仓库
git clone https://github.com/zhikanyeye/kiro-auto.git
cd kiro-auto

# 2. 安装依赖
npm install

# 3. 安装 Playwright 浏览器（含系统依赖）
npm run install-browser

# 4. 运行一次注册任务
npm run register -- --count 1 --non-interactive

# 5. 或启动账号切换工具
npm run switch
```

### 方案 C：低价云主机 + 定时任务（长期稳定）

在任意 Linux 主机上完成上述步骤后，用 `cron` 或 `pm2` 定期触发任务：

```bash
# 用 pm2 启动（需先 npm install -g pm2）
pm2 start "npm run register -- --count 1 --non-interactive" --name kiro-register --cron "0 9 * * *"
pm2 save
```

---

## 4. 常见问题（部署视角）

**Q1：Railway 构建成功，但运行时报错？**

先查看 Runtime Logs，常见原因：
- 缺少必要的环境变量（如代理配置）
- 代理地址不可用，导致连接 AWS 超时
- 上游临时邮件服务访问失败

**Q2：Playwright 在云端无法启动浏览器？**

检查以下几点：
- 构建日志中 `npm run install-browser` 是否执行成功
- 是否缺少无头浏览器所需的系统库（Railway 基于 Debian，通常已预装）
- 确认使用无头模式（`headless: true`），云端无显示器环境不支持有头模式

**Q3：免费优先策略下如何保证稳定性？**

- **开发 / 验证阶段**：Railway 免费层足够
- **长期运行**：迁移到自有 Linux 主机，用定时任务触发脚本，成本更可控
- **注意**：不建议依赖"永久免费"承诺，免费策略可能随时变化

**Q4：results.json 在 Railway 部署后消失了？**

Railway 容器是无状态的，每次重新部署文件系统会重置。  
若需持久化，接入 Supabase / Neon 并修改写入逻辑，或使用 Railway 的 Volume（持久化磁盘，按量付费）。

---

## 5. 推荐执行顺序（最短落地路径）

1. **Fork 仓库** 到你的 GitHub 账号（如已有权限可跳过）
2. **Railway 连接 GitHub** → 选择仓库 → 触发首次部署
3. **查看构建日志**，确认依赖与浏览器安装成功
4. **设置 Start Command** 为 `npm run register -- --count 1 --non-interactive`，按需补齐环境变量
5. **查看 Runtime Logs**，确认脚本执行完成
6. 跑通后再根据需要接入数据库或迁移到自有主机

---

> 如需进一步优化，可以在仓库根目录添加 `railway.json` 指定构建/启动配置，让 Railway 部署更稳定。
