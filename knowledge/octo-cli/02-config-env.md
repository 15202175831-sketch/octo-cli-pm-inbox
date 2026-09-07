# 02 · 配置与环境变量（Config & Env）

> **能答的**：哪些环境变量必填、各自管什么、优先级、Validate 规则。
> **要引用的**：`internal/config/config.go` `cmd/config.go`。

## 一、环境变量总表

**来源**: `internal/config/config.go#L14-L38`（Env* 常量定义）

| 变量 | 必需 | 管什么 |
|---|:---:|---|
| `OCTO_API_BASE_URL` | ✅ | 统一网关根 URL |
| `OCTO_TOKEN` | ⚪ | 凭证 token（优先级最高）|
| `OCTO_BOT_TOKEN` | ⚪ | 凭证 token（备用）|
| `OCTO_CREDENTIAL_MODE` | ⚪ | 凭证解析模式 |
| `OCTO_SPACE_ID` | ⚪ | 空间 ID（平台型 Bot 用）|
| `OCTO_FORMAT` | ⚪ | 输出格式默认值 |
| `OCTO_BOT_ID` | ⚪ | 未验证的 robot_id 声明 |
| `OCTO_CONFIG_DIR` | ⚪ | authstore 存储目录（默认 `~/.octo`）|

**来源**: `internal/authstore/authstore.go#L22`（`OCTO_CONFIG_DIR`）

## 二、Config 结构

**来源**: `internal/config/config.go#L41-L57`

```go
type Config struct {
    APIBaseURL string  // OCTO_API_BASE_URL
    BotToken   string  // OCTO_TOKEN 或 OCTO_BOT_TOKEN
    SpaceID    string  // OCTO_SPACE_ID
    Format     string  // OCTO_FORMAT
    // ...
}
```

## 三、加载流程

**来源**: `internal/config/config.go#L61-L83`（`Load()`）

1. 读所有 `OCTO_*` 环境变量
2. Token 走 `envToken()` 内部函数处理优先级
3. 返回 `*Config`

## 四、Validate 规则

**来源**: `internal/config/config.go#L85-L102`

必需项校验：
- `APIBaseURL` 不能为空
- token 不能为空（除非某些子命令允许空 token）

## 五、`NormalizeAPIBaseURL`

**来源**: `internal/config/config.go#L104-L125`

- 去掉尾部 `/`
- 校验 scheme 必须是 `http`/`https`
- 校验能 parse 通过

## 六、`octo-cli config` 子命令

**来源**: `cmd/config.go` (128 行)

- `config show` —— 打印当前生效配置（token 掩码）
- `config get <key>` —— 读特定项
- `config validate` —— 手动跑 Validate

## 七、常见问答备答

- **Q: `OCTO_CREDENTIAL_MODE` 有哪些值？**
  A: 需查 `internal/credential/provider.go` 的模式常量定义。（暂未查到明确常量）

- **Q: `OCTO_FORMAT` 支持哪些值？**
  A: 见 `internal/output/format.go`，默认支持 `json` / `raw` / `jq` 等。

## 八、我不确定的
- `OCTO_CREDENTIAL_MODE` 的完整取值集
