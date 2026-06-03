# gvu

**Go Module Vanity URL** management tool.

`gvu` lets you register and manage custom import paths (vanity URLs) for your Go modules — so your users can `go get gomod.io/your/pkg` or `go get <yourorg-domain>/your/pkg` instead of a raw GitHub/GitLab URL.

```go
// Before
import "github.com/yourorg/yourrepo/pkg"

// After
import "gomod.io/your/pkg"

or 

import "<yourorg-domain>/your/pkg"
```

## How it works

1. You register a vanity path (e.g. `gomod.io/your/pkg`) and point it to your Git repository
2. When a Go developer runs `go get gomod.io/your/pkg`, the server returns the correct `go-import` meta tag
3. The Go toolchain fetches the module from the actual Git repository
4. If you move your code to a different host, your users' import paths stay the same

## Installation

### Quick install (Linux / macOS / WSL)

```bash
curl -fsSL https://gomodvanityurls.com/install.sh | bash
```

### Manual install

Download the binary for your platform from the [latest release](https://github.com/gomodvanityurls/gvu/releases/latest):

| Platform | Binary |
|----------|--------|
| Linux (x86_64) | `gvu-linux-amd64` |
| Linux (ARM64) | `gvu-linux-arm64` |
| Linux (ARM) | `gvu-linux-arm` |
| macOS (Intel) | `gvu-darwin-amd64` |
| macOS (Apple Silicon) | `gvu-darwin-arm64` |
| Windows (x86_64) | `gvu-windows-amd64.exe` |
| Windows (ARM64) | `gvu-windows-arm64.exe` |

```bash
# Example: Linux x86_64
wget https://github.com/gomodvanityurls/gvu/releases/latest/download/gvu-linux-amd64
chmod +x gvu-linux-amd64
sudo mv gvu-linux-amd64 /usr/local/bin/gvu
```

## Quick start

//TBD
