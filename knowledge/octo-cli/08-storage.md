# 08 · 安全与本地存储（Security & Local Storage）

> **能答的**：token 存哪、怎么加密、机器绑定、多档案管理。
> **要引用的**：`internal/authstore/` `SECURITY.md`。

## 一、存储位置

**来源**: `internal/authstore/authstore.go#L22`（`EnvConfigDir = "OCTO_CONFIG_DIR"`）
**来源**: `internal/authstore/authstore.go#L81-L90`（`resolveDir()`）

默认目录：**`$HOME/.octo/`**（可被 `OCTO_CONFIG_DIR` 覆盖）。

三个关键文件（`internal/authstore/authstore.go#L92-L94`）：
- `configPath()` —— 档案元信息（明文 JSON）
- `credPath()` —— **加密后的 token 密文**
- `saltPath()` —— 加密盐

## 二、加密方案

**来源**: `internal/authstore/crypto.go` (165 行完整实现)

- **算法**：AES-256-GCM（`newGCM` 见 `#L155-L164`）
- **KDF**：从"机器 ID + salt"派生密钥（`deriveKey` 见 `#L79-L98`）
- **Salt**：32 字节，首次运行时随机生成并持久化（`saltLen = 32` 见 `#L15`）
- **Seal / Open**：GCM 密封与解密（`#L129-L153`）

## 三、机器绑定（machineid）

**来源**: `internal/authstore/machineid_*.go`

按平台读取机器唯一 ID：
- `machineid_darwin.go` —— 用 `IOPlatformUUID`
- `machineid_linux.go` —— 用 `/etc/machine-id` 或 `/var/lib/dbus/machine-id`
- `machineid_windows.go` —— 用注册表
- `machineid_other.go` —— fallback

**为啥要绑定机器**：让加密文件**只能在同一台机器解开**。拷贝走 `.octo/` 到别的机器解不开 → 防止误 copy 泄露。

## 四、多档案（Profile）

**来源**: `internal/authstore/authstore.go#L49-L56`（`ProfileMeta`）

一个用户可以存多个 Bot 档案（`ProfileMeta` map），支持：
- **来源**: `#L163-L206`（`SaveProfile`）
- **来源**: `#L238-L276`（`RemoveProfile`）
- **来源**: `#L288-L306`（`GetToken`）
- **来源**: `#L308-L347`（`ActiveProfile` —— 决定当前活动档案）

## 五、原子写

**来源**: `internal/authstore/authstore.go#L377-L400`（`atomicWrite`）

`write to tmp + rename` 原子替换，防止半写入损坏文件。

## 六、Mail 绑定

**来源**: `internal/authstore/mail.go` (370 行)

**Mail Credential** 是一个附加维度：邮箱凭证与本地档案绑定，切换 token 时可能会失效。

**来源**: `internal/credential/provider.go#L18-L20`（"only select a Mail credential whose encrypted local binding matches the active token"）

## 七、Status（活动档案状态）

**来源**: `internal/authstore/authstore.go#L35-L48`（`Status` 枚举）

有几个状态位表明当前活动档案是清晰的、有歧义的、还是不匹配等。

## 八、SECURITY.md

**来源**: 项目根 `SECURITY.md` (2951 bytes)

- 漏洞报告流程
- 支持的版本
- 已知安全策略

## 九、常见问答备答

- **Q: token 存哪？**
  A: `$HOME/.octo/`（可由 `OCTO_CONFIG_DIR` 覆盖），密文文件由 AES-256-GCM 加密，密钥从"机器 ID + 32 字节 salt"派生。见 `authstore/crypto.go`。

- **Q: 把 `.octo/` 拷贝到别的机器能用吗？**
  A: 不能。密钥绑机器 ID。见 `authstore/machineid_*.go`。

- **Q: 支持几个 Bot 档案？**
  A: 多档案，通过 profile 名区分，见 `authstore.go` 的 `ProfileMeta` map。

## 十、我不确定的
- 具体的 machine ID 采集在容器/沙箱环境的兼容性
