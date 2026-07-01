# gvu（简体中文）

**[English](README.md)** | 简体中文

---

**毫不费力地为 Go 模块创建自定义导入路径。**

`gvu` 让你为 Go 模块注册和管理自定义导入路径（Vanity URL），让用户使用简洁、稳定的导入路径，而不是冗长的 GitHub/GitLab 原始 URL。

```go
// 之前
import "github.com/yourorg/yourrepo/pkg"

// 之后 — 使用平台默认的 gomod.io 域名
import "gomod.io/your/pkg"

// 之后 — 使用你自己的域名
import "go.yourcompany.com/your/pkg"
```

> **两种 Vanity URL 类型：**
> - **`gomod.io`** — 平台默认提供的域名。开箱即用，无需配置 DNS。所有用户免费使用。
> - **自定义域名** — 使用你自己的域名（如 `go.yourcompany.com`）。需要 [Verified](#配额与套餐) 或 Pro 账户，并提前配置 DNS CNAME 记录。

> **工作原理：** 你注册一个 vanity 路径并指向你的 Git 仓库。当开发者对该路径执行 `go get` 时，服务端返回正确的 `go-import` meta 标签，Go 工具链再从实际仓库拉取代码。如果你迁移代码，用户的导入路径不受影响。

---

## 安装

### 一键安装（Linux / macOS / WSL）

```bash
curl -fsSL https://gomodvanityurls.com/install.sh | bash
```

安装指定版本：

```bash
curl -fsSL https://gomodvanityurls.com/install.sh | VERSION=v0.0.1 bash
```

### Windows

**方式 A — Scoop（PowerShell / CMD）：**

```powershell
scoop bucket add gomodvanityurls https://github.com/gomodvanityurls/scoop-bucket
scoop install gvu
```

**方式 B — Git Bash / MSYS2 / WSL：**

```bash
curl -fsSL https://gomodvanityurls.com/install.sh | bash
```

### 手动安装

从 [最新 Release](https://github.com/gomodvanityurls/gvu/releases/latest) 下载对应平台的二进制文件：

| 平台 | 文件名 |
|------|--------|
| Linux (x86_64) | `gvu-linux-amd64` |
| Linux (ARM64) | `gvu-linux-arm64` |
| Linux (ARM) | `gvu-linux-arm` |
| macOS (Intel) | `gvu-darwin-amd64` |
| macOS (Apple Silicon) | `gvu-darwin-arm64` |
| Windows (x86_64) | `gvu-windows-amd64.exe` |
| Windows (ARM64) | `gvu-windows-arm64.exe` |

校验文件完整性：

```bash
sha256sum -c checksums.txt
```

---

## 快速开始

### 1. 添加路由

**使用 `gomod.io`（平台默认域名，无需任何配置）：**

```bash
$ gvu add gomod.io/mylib https://github.com/myorg/mylib.git

✔ Route created successfully!
═══════════════════════════════════════════════════════════
┃ Route ID:     m_abc123
┃ Vanity Path:  gomod.io/mylib
┃ Target Repo:  https://github.com/myorg/mylib.git
┃ Quota:        1 / 5 used
═══════════════════════════════════════════════════════════
```

首次使用时会自动创建匿名账户，无需手动注册。

**Monorepo 子目录支持（Go 1.25+）：**

如果你的仓库包含多个子目录下的 Go 模块，使用 `--subdir`：

```bash
gvu add gomod.io/module-a https://github.com/myorg/monorepo.git --subdir go/module-a
```

Go 1.25+ 工具链会 clone 仓库并读取 `go/module-a/go.mod`。Go < 1.25 版本会静默忽略 `subdir` 字段（向后兼容）。

**使用自定义域名（需要 Verified 账户）：**

```bash
# 第一步：创建匿名账户（快速，无需邮箱）
gvu auth signup

# 第二步：绑定邮箱，升级为 Verified 账户
gvu auth bind user@example.com
# 输入发送到邮箱的 6 位验证码

# 第三步：配置 DNS 解析（必须在添加路由前完成）
# go.mycompany.com  CNAME  gomodvanityurls.com

# 第四步：添加自定义域名路由
gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git
```

> **DNS 必须先配置完成。** 服务端在创建路由时会验证 CNAME 记录。如果 DNS 尚未配置，命令将以 `DNS_NOT_CONFIGURED` 错误失败。

### 2. 配置 Go 环境

告诉 Go 工具链直接解析你的 vanity 路径（绕过公共代理）：

```bash
# gomod.io 路由
export GOPRIVATE=gomod.io

# 自定义域名路由（同时添加你的域名）
export GOPRIVATE=gomod.io,go.mycompany.com

# 建议保留公共代理作为兜底
export GOPROXY=https://goproxy.cn,direct
```

添加到 `~/.bashrc` 或 `~/.zshrc` 以持久化。

### 3. 开始使用

```go
// gomod.io（平台默认域名）
import "gomod.io/mylib"

// 自定义域名
import "go.mycompany.com/mylib"
```

就这么简单。Go 工具链会自动将 vanity 路径解析到你的 Git 仓库。

> **重要提示：** 你的模块 `go.mod` 中的 module path 必须声明为 vanity URL，而**不是**原始仓库地址。
>
> ```go
> // go.mod — 正确
> module gomod.io/mylib
>
> // go.mod — 错误（会导致导入解析失败）
> module github.com/myorg/mylib
> ```

---

## 命令参考

### 路由管理

| 命令 | 说明 |
|------|------|
| `gvu add <vanity路径> <仓库URL>` | 添加新的 vanity URL 路由 |
| `gvu list` | 列出所有已注册的路由 |
| `gvu remove <路由ID> [...]` | 删除一个或多个路由 |

#### `gvu add`

托管平台会根据仓库 URL **自动识别**，无需指定 `-t` 参数。

```bash
# gomod.io 路由（平台默认域名，所有用户可用）
gvu add gomod.io/mylib https://github.com/myorg/mylib.git
gvu add gomod.io/mylib https://gitlab.com/myorg/mylib.git
gvu add gomod.io/mylib https://user@bitbucket.org/workspace/mylib.git
gvu add gomod.io/mylib https://gitlab.example.com/group/mylib.git

# 自定义域名路由（需要 Verified 或 Pro 账户；DNS 必须先配置完成）
gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git
gvu add go.mycompany.com/api   ssh://git@gitlab.example.com:2222/group/api.git

# SSH URL 同时支持 gomod.io 和自定义域名
gvu add gomod.io/mylib git@github.com:myorg/mylib.git
gvu add gomod.io/mylib ssh://git@github.com/myorg/mylib.git

# Monorepo 子目录（Go 1.25+）
gvu add gomod.io/module-a https://github.com/myorg/monorepo.git --subdir go/module-a
```

#### `gvu remove`

支持批量删除：

```bash
gvu remove m_abc123                    # 删除单个路由
gvu remove m_abc123 m_def456           # 删除多个路由
gvu remove m_abc123 m_def456 -y        # 跳过确认提示
```

### 认证

| 命令 | 说明 |
|------|------|
| `gvu auth signup` | 创建匿名账户 |
| `gvu auth bind <邮箱>` | 绑定邮箱，升级为 Verified 用户 |
| `gvu auth login <邮箱>` | 通过邮箱 OTP 在新设备登录 |
| `gvu auth logout` | 清除本地凭证（仅 Verified 用户） |
| `gvu auth whoami` | 查看当前账户信息和配额 |

#### 账户升级路径

```
  Anonymous (5 条路由)        Verified (10 条路由)        Pro (50 条路由)
  ┌──────────────┐  绑定邮箱   ┌──────────────────┐  订阅   ┌──────────────┐
  │ 首次使用时    │───────────►│ 邮箱已验证        │────────►│ 付费套餐      │
  │ 自动创建      │            │ 支持多设备        │        │ 50 条路由     │
  └──────────────┘            └──────────────────┘        │ 50 个域名     │
                                                          └──────────────┘
```

- **Anonymous（匿名）**：自动创建。5 条 `gomod.io` 路由，无需邮箱。
- **Verified（已验证）**：运行 `gvu auth bind <邮箱>` 并通过 6 位 OTP 验证。10 条路由 + 1 个自定义域名。
- **Pro（专业版）**：运行 `gvu subscribe` 订阅。50 条路由 + 50 个自定义域名 + 域名独占绑定。

#### 多设备使用

在新设备上使用已验证账户：

```bash
gvu auth login user@example.com
# 输入邮箱收到的 6 位验证码
```

### 订阅

| 命令 | 说明 |
|------|------|
| `gvu subscribe` | 订阅 Pro 套餐 |

Pro 套餐价格：

| 周期 | 人民币 | 美元 |
|------|--------|------|
| 月付 | ¥9.9 | $3.99 |
| 年付 | ¥79 | $29.99 |

支付方式：微信支付、支付宝、PayPal。

> **Windows 用户：** `gvu subscribe` 会自动用系统图片查看器打开支付二维码。

### 其他命令

| 命令 | 说明 |
|------|------|
| `gvu donate` | 显示捐赠二维码（微信 / 支付宝 / PayPal） |
| `gvu issue` | 在浏览器中创建 GitHub Issue |
| `gvu --version` | 显示版本信息 |

---

## 配额与套餐

| | Anonymous | Verified | Pro |
|---|:---:|:---:|:---:|
| **`gomod.io` 路由** | 5 | 10 | 50 |
| **自定义域名路由** | 0 | 1 | 50 |
| **合计路由** | 5 | 10 | 50 |
| **短路径（< 4 字符）** | 不支持 | 不支持 | 支持 |
| **域名绑定** | 0 | 0 | 50 |
| **多设备** | 不支持 | 支持 | 支持 |

- **`gomod.io`**：平台默认域名，开箱即用，无需 DNS 配置
- **自定义域名**：用户自有域名（如 `go.mycompany.com`），需在 `gvu add` 前完成 DNS CNAME 配置
- **短路径**：路径第一段 < 4 个字符（自定义域名不受此限制）
- **域名绑定**：Pro 用户独占绑定某个域名后，其他用户无法在该域名下添加路由
- **配额**：所有路由类型共享同一配额池，各类型有独立上限

---

## 自定义域名

Verified 和 Pro 用户可以使用自己的域名作为 vanity URL 前缀。

### 配置步骤

```bash
# 1. 升级为 Verified（如尚未升级）
gvu auth bind user@example.com

# 2. 先配置 DNS（必须在添加路由前完成）
#    go.mycompany.com  CNAME  gomodvanityurls.com

# 3. 添加自定义域名路由
gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git
```

> **DNS 必须在 `gvu add` 之前配置完成。** 服务端在创建路由时会验证 CNAME 记录。

### 域名绑定（仅 Pro）

Pro 用户享有**域名独占绑定**：当 Pro 用户在新自定义域名下添加路由时，该域名会自动绑定到该用户。绑定后，其他用户无法在该域名下添加路由。

- Pro 用户最多可绑定 50 个域名
- Verified 用户可使用 1 个自定义域名，但不参与绑定机制
- Pro 订阅过期且路由被清理后，域名绑定会自动解除

---

## 支持的 Git 平台

`gvu` 根据仓库 URL 自动识别托管平台：

| 平台 | HTTPS | SSH | 自托管 |
|------|:-----:|:---:|:------:|
| **GitHub** | ✅ | ✅ | — |
| **GitLab** | ✅ | ✅ | ✅ |
| **Bitbucket** | ✅ | ✅ | — |
| **Codeberg** | ✅ | ✅ | — |
| **自定义** | ✅ | ✅ | ✅ |

非标准端口的 SSH URL 请使用 `ssh://git@host:port/path` 格式。

---

## 全局参数

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--json` | `-j` | 结构化 JSON 输出（适合脚本调用） |
| `--yes` | `-y` | 跳过确认提示 |
| `--version` | `-v` | 显示版本 |
| `--help` | `-h` | 显示帮助 |

### JSON 输出

所有命令支持 `--json` 输出机器可读的结果：

```bash
$ gvu list --json
{
  "status": "success",
  "data": {
    "routes": [
      {
        "id": "m_abc123",
        "vanity_path": "gomod.io/mylib",
        "target_repo": "https://github.com/myorg/mylib.git",
        "repo_type": "github"
      }
    ]
  }
}
```

---

## 配置文件

`gvu` 将凭证存储在 `~/.config/gvu/config.yaml`，首次使用时自动创建。

正常使用无需手动配置。

---

## 常见问题

### 为什么 `go get` 无法使用我的 vanity URL？

请确保已将域名添加到 `GOPRIVATE`：

```bash
export GOPRIVATE=gomod.io,go.mycompany.com
```

### 为什么 `go.mod` 必须使用 vanity URL 而不是 `github.com/...`？

Go 工具链使用**导入路径**来定位模块。`go.mod` 中的 `module` 指令必须与 vanity URL 完全一致，否则 `go get` 和 `go build` 会失败。

### 私有仓库可以使用 SSH URL 吗？

可以。`gvu` 支持所有标准 SSH URL 格式：

```bash
gvu add gomod.io/mylib git@github.com:myorg/mylib.git
gvu add go.mycompany.com/mylib ssh://git@gitlab.example.com:2222/group/mylib.git
```

使用你模块的开发者需要在 Git 托管平台配置自己的 SSH Key。

### 如何迁移模块到新的托管平台？

删除旧路由，添加指向新仓库的路由：

```bash
gvu remove m_abc123
gvu add gomod.io/mylib https://newhost.com/myorg/mylib.git
```

用户的导入路径无需任何变更。

### 可以在 CI/CD 中使用 `gvu` 吗？

可以。使用 `--json` 获取结构化输出，`--yes` 跳过交互确认：

```bash
gvu add gomod.io/mylib $REPO_URL --json --yes
```

### 如果先添加自定义域名路由再配置 DNS 会怎样？

命令会以 `DNS_NOT_CONFIGURED` 错误失败。你必须**先**配置 CNAME 记录，**再**运行 `gvu add`：

```
go.mycompany.com  CNAME  gomodvanityurls.com
```

等待 DNS 生效（通常几分钟），然后重试。

---

## 捐赠

如果 `gvu` 对你有帮助，请我喝杯咖啡吧！

**PayPal**：[![Donate with PayPal](https://img.shields.io/badge/Donate-PayPal-0070ba?logo=paypal&logoColor=white)](https://www.paypal.com/paypalme/tonybaicn)

**微信 / 支付宝**：

<p float="left">
  <img src="doc/donate/wechat-pay.png" alt="微信支付" width="200"/>
  <img src="doc/donate/alipay.png" alt="支付宝" width="200"/>
</p>

---

## 支持

- **问题反馈**：`gvu issue` 或 [GitHub Issues](https://github.com/gomodvanityurls/gvu/issues)
- **官方网站**：[gomodvanityurls.com](https://gomodvanityurls.com)
