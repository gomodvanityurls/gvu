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
> - **自定义域名** — 使用你自己的域名（如 `go.yourcompany.com`）。需要 [Verified](#配额与套餐) 或 Pro 账户，并配置 DNS CNAME 记录。

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

```bash
# 示例：Linux x86_64
wget https://github.com/gomodvanityurls/gvu/releases/latest/download/gvu-linux-amd64
chmod +x gvu-linux-amd64
sudo mv gvu-linux-amd64 /usr/local/bin/gvu
```

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

**使用自定义域名（需要 Verified 账户）：**

```bash
$ gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git

✔ Route created successfully!
═══════════════════════════════════════════════════════════
┃ Route ID:     m_def456
┃ Vanity Path:  go.mycompany.com/mylib
┃ Target Repo:  https://github.com/myorg/mylib.git
┃ TLS:          ✔ Certificate provisioned
═══════════════════════════════════════════════════════════
```

首次使用时会自动创建匿名账户，无需手动注册。如需使用自定义域名，请 [绑定邮箱](#认证) 升级为 Verified 用户。

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

# 自定义域名路由（需要 Verified 或 Pro 账户）
gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git
gvu add go.mycompany.com/api   ssh://git@gitlab.example.com:2222/group/api.git

# SSH URL 同时支持 gomod.io 和自定义域名
gvu add gomod.io/mylib git@github.com:myorg/mylib.git
gvu add gomod.io/mylib ssh://git@github.com/myorg/mylib.git
gvu add gomod.io/mylib ssh://git@gitlab.example.com:2222/group/mylib.git
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
- **Pro（专业版）**：订阅制。50 条路由 + 50 个自定义域名。*（即将推出）*

#### 多设备使用

在新设备上使用已验证账户：

```bash
gvu auth login user@example.com
# 输入邮箱收到的 6 位验证码
```

### 其他命令

| 命令 | 说明 |
|------|------|
| `gvu donate` | 显示捐赠二维码（微信 / 支付宝） |
| `gvu issue` | 在浏览器中创建 GitHub Issue |
| `gvu --version` | 显示版本信息 |

---

## 配额与套餐

| | Anonymous | Verified | Pro |
|---|:---:|:---:|:---:|
| **`gomod.io` 路由** | 5 | 10 | 50 |
| **自定义域名路由** | 0 | 1 | 50 |
| **合计路由** | 5 | 10 | 50 |
| **短路径（< 4 字符）** | 不支持 | 支持 | 支持 |
| **多设备** | 不支持 | 支持 | 支持 |

所有路由类型共享同一个配额池。例如，Verified 用户如果有 1 条自定义域名路由，则还剩 9 条 `gomod.io` 路由可用。

> `gomod.io` 是平台默认域名，无需任何 DNS 配置即可使用。自定义域名需要你将 CNAME 记录指向服务端。

---

## 自定义域名

Verified 和 Pro 用户可以使用自己的域名作为 vanity URL 前缀：

```bash
# 1. 升级为 Verified（如尚未升级）
gvu auth bind user@example.com

# 2. 添加自定义域名路由
gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git

# 3. 配置 DNS，将域名指向服务端
#    go.mycompany.com  CNAME  gomodvanityurls.com

# 4. TLS 证书自动签发（Caddy On-Demand TLS）
```

CLI 会在路由创建后自动探测 TLS 状态，如证书仍在签发中会给出提示。

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
      },
      {
        "id": "m_def456",
        "vanity_path": "go.mycompany.com/api",
        "target_repo": "https://github.com/myorg/api.git",
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

### 私有仓库可以使用 SSH URL 吗？

可以。`gvu` 支持所有标准 SSH URL 格式：

```bash
gvu add gomod.io/mylib git@github.com:myorg/mylib.git
gvu add gomod.io/mylib ssh://git@github.com/myorg/mylib.git
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

---

## 支持

- **源码仓库**：[github.com/bigwhite/gomodvanityurls](https://github.com/bigwhite/gomodvanityurls)
- **问题反馈**：`gvu issue` 或 [GitHub Issues](https://github.com/bigwhite/gomodvanityurls/issues)
- **捐赠支持**：`gvu donate`

## 许可证

本项目为开源项目，详见 [源码仓库](https://github.com/bigwhite/gomodvanityurls) 中的许可证文件。
