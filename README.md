# 🎮 Runox 自动唤醒

使用 SeleniumBase + GitHub Actions 每 2 小时自动唤醒并保持 Runox 免费服务器运行。

---

## ⚙️ 配置步骤

### 1️⃣ 准备仓库

- **公库**：存放 `.github/workflows/runox-awaken.yml`
- **私库**（`Runox-Awaken-Private`）：存放 `awaken.py`

### 2️⃣ 添加 Secrets

进入公库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**：

| 🔑 Secret 名称 | 📝 格式 | ✅ 必填 |
|---|---|---|
| `RUNOX_ACCOUNT` | `email,password` | ✅ |
| `PRIVATE_REPO_TOKEN` | GitHub PAT（有私库读取权限） | ✅ |
| `TG_BOT` | `chat_id,bot_token` | 可选 |
| `GOST_PROXY` | `socks5://user:pass@host:port` | 可选 |

### 3️⃣ 启用 Actions

进入公库 **Actions** 标签页，点击 **Enable GitHub Actions**。

### 4️⃣ 手动触发测试

**Actions** → **Runox-Awaken** → **Run workflow**

---

## 🕐 运行时间

每 **2 小时**自动运行一次（UTC `0 */2 * * *`）。

可在 `.github/workflows/runox-awaken.yml` 的 `cron` 表达式中修改。

---

## 📊 运行逻辑

脚本登录后读取服务器状态，根据不同状态执行不同操作：

### 状态一：HIBERNATED（服务器已休眠）

服务器进入休眠状态，需要先通过 Cloudflare Turnstile 人机验证才能唤醒。

```
登录 → 进入服务器详情页
→ Cloudflare Turnstile 验证
→ 点击 Start / Restore（唤醒休眠）
→ 等待 Start 按钮出现
→ 点击 Start（正式启动）
→ 等待状态变为 Online
→ 📩 TG 推送：✅ 唤醒成功 + 使用期限
```

### 状态二：Offline（已唤醒，待启动）

服务器已从休眠恢复，但尚未启动，页面无 CF 验证。

```
登录 → 进入服务器详情页
→ 直接等待 Start 按钮出现
→ 点击 Start
→ 等待状态变为 Online
→ 📩 TG 推送：✅ 唤醒成功 + 使用期限
```

### 状态三：Active（运行中，期限充足）

服务器正在运行，且剩余使用时间 ≥ 1 小时，无需任何操作。

```
登录 → 进入服务器详情页
→ 读取使用期限
→ 期限 ≥ 1 小时：无操作，静默退出
```

### 状态四：Active（运行中，期限不足）

服务器正在运行，但剩余使用时间 < 1 小时，需要重启续期。

```
登录 → 进入服务器详情页
→ 读取使用期限
→ 期限 < 1 小时：点击 Stop
→ 等待 Start 按钮出现
→ 点击 Start
→ 等待状态变为 Online
→ 静默退出（不推送 TG）
```

---

## 📩 TG 推送说明

| 情况 | 是否推送 | 推送内容 |
|---|---|---|
| HIBERNATED 唤醒成功 | ✅ | `✅ 唤醒成功` + 使用期限 |
| Offline 启动成功 | ✅ | `✅ 唤醒成功` + 使用期限 |
| Active 期限充足 | ❌ | 静默，不推送 |
| Active 重启成功 | ❌ | 静默，不推送 |
| Start 按钮超时 | ✅ | `❌ Start 按钮未出现` |
| Turnstile 验证失败 | ✅ | `❌ Turnstile验证失败` |
| 启动超时 | ✅ | `⚠️ 启动超时，状态：xxx` |
| 登录失败 | ✅ | `❌ 登录超时` 等 |

**推送格式示例：**
```
🎮 Runox 唤醒通知
🕐 运行时间: 2026-03-16 11:07:58
🖥 服务器: Runox-NL
📊 唤醒结果: ✅ 唤醒成功
⏱️ 使用期限：2 hours y 59 minutes
```

---

## 🖼️ 调试截图

每次运行后，截图会上传至 Actions Artifacts（`debug-screenshots`），方便排查问题：

| 截图文件 | 说明 |
|---|---|
| `dashboard.png` | 登录后 dashboard 页面 |
| `server_detail.png` | 进入服务器详情页时的状态 |
| `turnstile_click_pos.png` | CF 验证点击位置（红点标记） |
| `turnstile_fail.png` | CF 验证失败时的页面 |
| `final_status.png` | 启动后的最终状态页面 |
