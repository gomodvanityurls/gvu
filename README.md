# gvu

English | **[简体中文](README.zh-CN.md)**

---

**Effortless vanity URLs for your Go modules.**

`gvu` lets you register and manage custom import paths for your Go modules — so your users get clean, stable import paths instead of raw GitHub/GitLab URLs.

```go
// Before
import "github.com/yourorg/yourrepo/pkg"

// After — using the default gomod.io domain
import "gomod.io/your/pkg"

// After — using your own domain
import "go.yourcompany.com/your/pkg"
```

> **Two types of vanity URLs:**
> - **`gomod.io`** — the default domain provided by the platform. Works out of the box, no DNS setup needed. Free for all users.
> - **Custom domain** — use your own domain (e.g., `go.yourcompany.com`). Requires a [Verified](#quota--plans) or Pro account and a DNS CNAME record.

> **How it works:** You register a vanity path and point it to your Git repository. When a Go developer runs `go get` on that path, the server returns the correct `go-import` meta tag, and the Go toolchain fetches from the actual repo. If you ever move your code, your users' import paths stay the same.

---

## Installation

### Quick install (Linux / macOS / WSL)

```bash
curl -fsSL https://gomodvanityurls.com/install.sh | bash
```

Install a specific version:

```bash
curl -fsSL https://gomodvanityurls.com/install.sh | VERSION=v0.0.1 bash
```

### Manual install

Download the binary for your platform from the [latest release](https://github.com/gomodvanityurls/gvu/releases/latest):

| Platform              | Binary                    |
|-----------------------|---------------------------|
| Linux (x86_64)       | `gvu-linux-amd64`         |
| Linux (ARM64)        | `gvu-linux-arm64`         |
| Linux (ARM)          | `gvu-linux-arm`           |
| macOS (Intel)        | `gvu-darwin-amd64`        |
| macOS (Apple Silicon)| `gvu-darwin-arm64`        |
| Windows (x86_64)     | `gvu-windows-amd64.exe`   |
| Windows (ARM64)      | `gvu-windows-arm64.exe`   |

```bash
# Example: Linux x86_64
wget https://github.com/gomodvanityurls/gvu/releases/latest/download/gvu-linux-amd64
chmod +x gvu-linux-amd64
sudo mv gvu-linux-amd64 /usr/local/bin/gvu
```

Verify checksums:

```bash
sha256sum -c checksums.txt
```

---

## Quick start

### 1. Add a route

**Using `gomod.io` (default domain, no setup required):**

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

**Using a custom domain (requires Verified account):**

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

An anonymous account is created automatically on first run — no signup required. To use custom domains, [bind your email](#authentication) to upgrade to Verified.

### 2. Configure your Go environment

Tell the Go toolchain to resolve your vanity paths directly (bypass public proxies):

```bash
# For gomod.io routes
export GOPRIVATE=gomod.io

# For custom domain routes (add your domain too)
export GOPRIVATE=gomod.io,go.mycompany.com

# Recommended: keep a public proxy as fallback
export GOPROXY=https://proxy.golang.org,direct
```

Add to your `~/.bashrc` or `~/.zshrc` to make it persistent.

### 3. Use it

```go
// gomod.io (default domain)
import "gomod.io/mylib"

// Custom domain
import "go.mycompany.com/mylib"
```

That's it. The Go toolchain resolves the vanity path to your Git repo automatically.

---

## Commands

### Route management

| Command | Description |
|---------|-------------|
| `gvu add <vanity-path> <repo-url>` | Add a new vanity URL route |
| `gvu list` | List all your registered routes |
| `gvu remove <route-id> [...]` | Remove one or more routes |

#### `gvu add`

The hosting platform is **auto-detected** from the repo URL — no `-t` flag needed.

```bash
# gomod.io routes (default domain, available to all users)
gvu add gomod.io/mylib https://github.com/myorg/mylib.git
gvu add gomod.io/mylib https://gitlab.com/myorg/mylib.git
gvu add gomod.io/mylib https://user@bitbucket.org/workspace/mylib.git
gvu add gomod.io/mylib https://gitlab.example.com/group/mylib.git

# Custom domain routes (requires Verified or Pro account)
gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git
gvu add go.mycompany.com/api   ssh://git@gitlab.example.com:2222/group/api.git

# SSH URLs work with both gomod.io and custom domains
gvu add gomod.io/mylib git@github.com:myorg/mylib.git
gvu add gomod.io/mylib ssh://git@github.com/myorg/mylib.git
gvu add gomod.io/mylib ssh://git@gitlab.example.com:2222/group/mylib.git
```

#### `gvu remove`

Supports bulk deletion:

```bash
gvu remove m_abc123                    # single route
gvu remove m_abc123 m_def456           # multiple routes
gvu remove m_abc123 m_def456 -y        # skip confirmation
```

### Authentication

| Command | Description |
|---------|-------------|
| `gvu auth signup` | Create an anonymous account |
| `gvu auth bind <email>` | Bind email to upgrade to Verified Member |
| `gvu auth login <email>` | Login on a new device via email OTP |
| `gvu auth logout` | Clear local credentials (verified accounts only) |
| `gvu auth whoami` | Show current account info and quota |

#### Account lifecycle

```
  Anonymous (5 routes)         Verified (10 routes)         Pro (50 routes)
  ┌──────────────┐  bind email  ┌──────────────────┐  subscribe  ┌──────────────┐
  │ Auto-created │─────────────►│ Email verified   │────────────►│ Paid plan    │
  │ on first use │              │ Multi-device     │             │ 50 routes    │
  └──────────────┘              └──────────────────┘             │ 50 domains   │
                                                                 └──────────────┘
```

- **Anonymous**: Account is created automatically. 5 `gomod.io` routes. No email required.
- **Verified**: Run `gvu auth bind <email>` and verify with a 6-digit OTP. 10 routes + 1 custom domain.
- **Pro**: Subscription-based. 50 routes + 50 custom domains. *(Coming soon)*

#### Multi-device access

To use your verified account on a new device:

```bash
gvu auth login user@example.com
# Enter the 6-digit code sent to your email
```

### Other commands

| Command | Description |
|---------|-------------|
| `gvu donate` | Show donation QR codes (WeChat Pay / Alipay) |
| `gvu issue` | Open a GitHub issue in your browser |
| `gvu --version` | Show version info |

---

## Quota & plans

| | Anonymous | Verified | Pro |
|---|:---:|:---:|:---:|
| **`gomod.io` routes** | 5 | 10 | 50 |
| **Custom domain routes** | 0 | 1 | 50 |
| **Total routes** | 5 | 10 | 50 |
| **Short paths (< 4 chars)** | No | Yes | Yes |
| **Multi-device** | No | Yes | Yes |

All route types share a single quota pool. For example, a Verified user with 1 custom domain route has 9 remaining slots for `gomod.io` routes.

> `gomod.io` is the platform's default domain — it works immediately without any DNS configuration. Custom domains require you to point a CNAME record to the service.

---

## Custom domains

Verified and Pro users can use their own domain as the vanity URL prefix:

```bash
# 1. Upgrade to Verified (if not already)
gvu auth bind user@example.com

# 2. Add a custom domain route
gvu add go.mycompany.com/mylib https://github.com/myorg/mylib.git

# 3. Point your domain's DNS to the service
#    go.mycompany.com  CNAME  gomodvanityurls.com

# 4. TLS certificate is provisioned automatically (Caddy On-Demand TLS)
```

The CLI probes the route after creation and warns you if TLS is still provisioning.

---

## Supported Git platforms

`gvu` auto-detects the hosting platform from the repo URL:

| Platform | HTTPS | SSH | Self-hosted |
|----------|:-----:|:---:|:-----------:|
| **GitHub** | Yes | Yes | — |
| **GitLab** | Yes | Yes | Yes |
| **Bitbucket** | Yes | Yes | — |
| **Codeberg** | Yes | Yes | — |
| **Custom** | Yes | Yes | Yes |

SSH URLs with non-standard ports use the `ssh://git@host:port/path` format.

---

## Global flags

| Flag | Short | Description |
|------|-------|-------------|
| `--json` | `-j` | Structured JSON output (for scripting) |
| `--yes` | `-y` | Skip confirmation prompts |
| `--version` | `-v` | Show version |
| `--help` | `-h` | Show help |

### JSON output

All commands support `--json` for machine-readable output:

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

## Configuration

`gvu` stores credentials in `~/.config/gvu/config.yaml`. This file is created automatically and contains your access token.

No manual configuration is required for normal use.

---

## FAQ

### Why doesn't `go get` work with my vanity URL?

Make sure you've added the domain to `GOPRIVATE`:

```bash
export GOPRIVATE=gomod.io,go.mycompany.com
```

### Can I use SSH URLs for private repos?

Yes. `gvu` supports all standard SSH URL formats:

```bash
gvu add gomod.io/mylib git@github.com:myorg/mylib.git
gvu add gomod.io/mylib ssh://git@github.com/myorg/mylib.git
gvu add go.mycompany.com/mylib ssh://git@gitlab.example.com:2222/group/mylib.git
```

Users of your module will need their own SSH keys configured for the Git host.

### How do I move a module to a different host?

Remove the old route and add a new one pointing to the new repo:

```bash
gvu remove m_abc123
gvu add gomod.io/mylib https://newhost.com/myorg/mylib.git
```

Your users' import paths don't change.

### Can I use `gvu` in CI/CD?

Yes. Use `--json` for structured output and `--yes` to skip prompts:

```bash
gvu add gomod.io/mylib $REPO_URL --json --yes
```

---

## Support

- **Source code**: [github.com/bigwhite/gomodvanityurls](https://github.com/bigwhite/gomodvanityurls)
- **Issues**: `gvu issue` or [GitHub Issues](https://github.com/bigwhite/gomodvanityurls/issues)
- **Donate**: `gvu donate`

## License

This project is open source. See the [source repository](https://github.com/bigwhite/gomodvanityurls) for license details.
