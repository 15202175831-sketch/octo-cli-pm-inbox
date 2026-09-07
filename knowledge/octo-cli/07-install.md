# 07 · 安装与发布（Install & Release）

> **能答的**：npm / go install / GitHub Releases / brew 各种装法、包命名。
> **要引用的**：`README.md#L74-L110` `install.sh` `npm/` `.goreleaser.yaml`。

## 一、四种安装方式

### 1. npm（推荐给 Node Agent Runtime）

**来源**: `README.md#L76-L84`

```bash
npm install -g @mininglamp-oss/octo-cli
```

**要点**：
- npm 包会自动 resolve 到对应平台的**子包**（预编译二进制在子包里）
- **不从 GitHub 下载二进制**（安全 / 网络 / 企业内网友好）

**平台子包**在 `npm/` 目录（`npm/darwin-arm64` / `npm/linux-x64` 等）。

### 2. go install

**来源**: `README.md#L86-L88`

```bash
go install github.com/Mininglamp-OSS/octo-cli/cmd/octo-cli@latest
```

需要本机装 Go，从源码构建。

### 3. Homebrew（即将支持）

**来源**: `README.md#L90-L94`

```bash
brew install Mininglamp-OSS/tap/octo-cli
```

（status: coming soon）

### 4. GitHub Releases

**来源**: `README.md#L96-L102`

直接下预编译二进制。

## 二、`install.sh`

**来源**: `install.sh`（2421 bytes）

自动化脚本：检测平台 → 下二进制 → 装到 `/usr/local/bin` 或 `$HOME/.local/bin`。

## 三、Release 工具链

**来源**: `.goreleaser.yaml` (1543 bytes)

- 用 [goreleaser](https://goreleaser.com/) 做多平台交叉编译
- 出 npm 平台子包 + GitHub Release archive
- CI 里由 GitHub Actions 触发

**来源**: `.github/workflows/`

## 四、包命名规范

- npm 主包：`@mininglamp-oss/octo-cli`
- npm 平台子包：`@mininglamp-oss/octo-cli-<os>-<arch>`（例 `octo-cli-darwin-arm64`）
- Go module：`github.com/Mininglamp-OSS/octo-cli`
- Homebrew tap：`Mininglamp-OSS/tap/octo-cli`
- GitHub Release archive：`octo-cli_<version>_<os>_<arch>.tar.gz`

## 五、CHANGELOG

**来源**: `CHANGELOG.md` (40701 bytes) —— 完整的版本变更记录。

## 六、常见问答备答

- **Q: npm 装完为啥不用下二进制？**
  A: 平台子包已经包含预编译二进制。见 `README.md#L82-L84`。

- **Q: 支持哪些平台？**
  A: darwin-arm64 / darwin-amd64 / linux-amd64 / linux-arm64 / windows-amd64（看 `npm/` 和 `.goreleaser.yaml`）。

- **Q: 怎么升级？**
  A: `npm update -g @mininglamp-oss/octo-cli` 或 `go install ...@latest`。

## 七、我不确定的
- Homebrew tap 的具体开放时间
