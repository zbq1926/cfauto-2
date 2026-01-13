# 🚀 Cloudflare Worker 多项目部署中控 (Traffic Monitor Edition)

这是一个基于 Cloudflare Worker 的**多项目集中部署与管理工具**。它允许你在一个统一的 Dashboard 中管理、配置并自动更新多个不同的 Worker 项目（目前支持 **CMliu (EdgeTunnel)** 和 **Joey (CFNew)**）。

> **✨ V3.2 版本特性**
> * **📊 精准流量监控**：内置 **Billing 级流量统计**，精准核算每日 10w 免费请求额度（包含 Workers 和 Pages 用量），每日 UTC 0点自动重置。
> * **⚡️ 零报错内核**：采用兼容性最强的 GraphQL Billing 接口，彻底解决 "unknown field" 报错问题。
> * **⏰ 自动更新**：支持后台定时检测 GitHub 上游版本，发现新版本自动拉取并重新部署。
> * **⚙️ 独立配置**：不同项目（CMliu/Joey）拥有独立的自动更新开关和变量配置。
> * **🧩 代码补丁**：针对 Joey 项目自动注入 `window` 环境补丁。

---

## 🛠️ 功能列表

1. **可视化仪表盘**：
   - **经典分栏布局**：左侧展示实时流量统计与账号列表，右侧为控制台与日志。
   - **进度条监控**：直观展示每日 10w 请求额度的使用进度（绿 -> 黄 -> 红 变色警示）。

2. **集中式账号管理**：
   - 支持添加多个 Cloudflare 账号（ID + Token）。
   - 支持将不同的 Worker 分配给不同账号，支持备注名 (Alias)。
   - **隐私保护**：账号列表默认折叠，直播或截图时更安全。

3. **智能更新系统**：
   - **手动更新**：一键检测 GitHub API，对比本地与远程 SHA，手动触发部署。
   - **自动更新**：配合 Cron Triggers，后台静默检查并更新。

4. **变量管理**：
   - 可视化增删改查 Worker 环境变量。
   - 支持 `UUID` 一键刷新生成。

---

## 📥 部署指南

### 1. 准备工作

* 一个 Cloudflare 账号。
* (可选) 一个 GitHub Token (用于提高 API 请求限额)。

### 2. 创建 Worker

1. 登录 Cloudflare Dashboard。
2. 进入 **Workers & Pages** -> **Create Application** -> **Create Worker**。
3. 命名为 `deploy-manager` (或你喜欢的名字)，点击 Deploy。
4. 点击 **Edit code**，将本项目提供的 `worker.js` (V3.2) 完整代码复制粘贴进去，保存。

### 3. 配置 KV 存储 (必须)

此项目依赖 KV 来存储账号数据、变量配置和版本信息。

1. 在 Cloudflare 侧边栏选择 **Storage & Databases** -> **KV**。
2. 点击 **Create a Namespace**，命名为 `CONFIG_KV`（建议）。
3. 回到你创建的 Worker 的 **Settings** -> **Variables** 页面。
4. 在 **KV Namespace Bindings** 区域：
   * Variable name 填写: `CONFIG_KV` (**必须完全一致**)
   * KV Namespace 选择你刚才创建的那个。

### 4. 设置环境变量

在 Worker 的 **Settings** -> **Variables** -> **Environment Variables** 区域添加：

| 变量名 | 示例值 | 说明 |
| --- | --- | --- |
| `ACCESS_CODE` | `password123` | **(强烈推荐)** 访问控制台的密码。如果不填，任何人都能访问你的后台！ |
| `GITHUB_TOKEN` | `ghp_xxxxxx` | **(推荐)** GitHub Personal Access Token。配置后可大幅提高更新检测的稳定性。 |

### 5. 配置自动更新 (Cron Triggers)

为了让“自动更新”功能生效，你需要设置一个唤醒触发器。

1. 进入 Worker 的 **Settings** -> **Triggers**。
2. 点击 **Add Cron Trigger**。
3. **CRON Expression** 建议填写：`0 */1 * * *` (每小时) 或 `*/30 * * * *` (每30分钟)。
   * *注意：脚本内部会根据你在前端页面设置的“间隔时间（小时）”来决定是否真正执行更新，这里只是唤醒频率。*

---

## 🔑 关键：API Token 权限配置

为了确保**流量统计**和**部署功能**都能正常工作，您填入的 Cloudflare API Token 必须具备以下权限：

1. **Permissions (权限)**:
   * `Account` -> `Account Analytics` -> **Read** (用于读取流量统计)
   * `Account` -> `Workers Scripts` -> **Edit** (用于部署 Worker)
   * *(可选)* `Account` -> `Account Settings` -> **Read**

2. **Account Resources (账户资源) [重要!]**:
   * ❌ **错误**：选择 `All zones`。
   * ✅ **正确**：必须选择 **`Include` -> `All accounts`** (所有账户) 或者指定具体的 Account。
   * *原因：流量统计是基于 Account 维度的，如果只授权 Zone 维度，查询接口会报错。*

---

## 💻 使用说明

### 1. 访问控制台
访问 `https://你的worker域名.workers.dev/`。如果设置了 `ACCESS_CODE`，需要输入密码登录。

### 2. 添加目标账号
在左侧“账号管理”面板点击“显示列表” -> 填写信息 -> 保存。
* **CMliu Workers**: 填写该账号下部署 EdgeTunnel 的 Worker 名（逗号分隔）。
* **Joey Workers**: 填写该账号下部署 CFNew 的 Worker 名（逗号分隔）。

### 3. 查看流量
添加账号后，点击左上角的 **“🔄 刷新”** 按钮。
* 面板会显示该账号下 Workers 和 Pages 的今日总请求数。
* 进度条会根据 100,000 次免费额度的消耗情况自动变色。

### 4. 变量与部署
* 切换顶部的下拉菜单选择项目（CMliu 或 Joey）。
* 修改变量后，点击底部的 **“立即执行更新”** 即可手动强制部署。

---

## 🧩 支持的项目模板

1. **🔴 CMliu - EdgeTunnel**
   * **Source**: `cmliu/edgetunnel` (beta2.0)
   * **Default Vars**: `UUID`, `PROXYIP`, `PATH` 等。

2. **🔵 Joey - 少年你相信光吗**
   * **Source**: `byJoey/cfnew`
   * **Special**: 自动注入 `var window = globalThis;` 补丁。
   * **Default Vars**: `u` (UUID), `d`。

---

## ⚠️ 免责声明

本项目仅供学习和技术研究使用。请勿用于非法用途。使用者需自行承担因使用本项目而产生的任何后果。
