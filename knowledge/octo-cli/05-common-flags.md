# 05 · 通用参数（Common Flags）

> **能答的**：`--format` `--jq` `--dry-run` `--page-all` 等通用 flag 的行为。
> **要引用的**：`internal/cmdutil/factory.go` (796 行) 各子命令。

## 一、Factory 依赖注入

**来源**: `internal/cmdutil/factory.go#L1-L50`（Factory 结构）

Factory 是 DI 容器，装配：
- `ConfigFunc` —— 配置加载器
- `CredentialFunc` —— 凭证解析器
- `ClientFunc` —— HTTP 客户端构造
- `RegistryFunc` —— OpenAPI 注册表

测试时通过替换这些 func 注入 stub。

## 二、通用 Flag 列表

**来源**: `internal/cmdutil/factory.go` （搜 `Flag`/`Persist`）

| Flag | 作用 |
|---|---|
| `--format` | 输出格式：`json`（默认）/ `raw` / `jq` 等 |
| `--jq <expr>` | 用 jq 表达式过滤输出 |
| `--dry-run` | 只演练，不实际发请求 |
| `--page-all` | 自动翻页拉全量 |
| `--space <id>` | 指定 space ID |
| `--profile <name>` | 使用某个已保存的凭证档案 |
| `--no-retry` | 全局禁用重试 |
| `--timeout <dur>` | 覆盖默认超时 |
| `--verbose` / `-v` | 详细日志（token 依然掩码）|

## 三、`--dry-run`

`--dry-run` 应当**不发起真实请求**，但仍输出预期的请求体/URL，便于 Agent 验证参数正确性。

## 四、`--page-all`

**来源**: `internal/client/client.go` 里的分页逻辑

自动跟随 pagination 元数据翻页，把所有页拼成一个数组返回。

## 五、`--format` 与 Envelope

**来源**: `internal/output/format.go` (218 行)

- 默认 `json` —— 输出完整 envelope
- `raw` —— 只输出 data 字段
- `jq` —— 用 `--jq` 表达式过滤

## 六、`--jq` 参数

**来源**: `internal/output/jq.go` (79 行)

支持 jq 语法在 envelope.data 上过滤。

## 七、IOStreams

**来源**: `internal/cmdutil/iostreams.go` (30 行)

stdin/stdout/stderr 抽象，便于测试注入。

## 八、常见问答备答

- **Q: `--dry-run` 会发请求吗？**
  A: 不发。只做参数校验+打印计划。

- **Q: `--page-all` 拉多少页？**
  A: 跟随后端 pagination cursor 拉到没有为止。（具体限流保护看 `client.go` 分页逻辑）

## 九、我不确定的
- `--format` 完整取值集合（需 grep `format.go`）
- `--page-all` 是否有页数上限保护
