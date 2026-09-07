# 04 · 输出与错误（Envelope & Errors）

> **能答的**：JSON envelope 结构、错误分类、退出码。
> **要引用的**：`internal/output/envelope.go` `internal/output/errors.go`。

## 一、Envelope 结构

**Envelope 是 octo-cli 的稳定 JSON 输出契约**，所有子命令都输出这个 envelope 到 stdout。

**来源**: `internal/output/envelope.go#L16-L26`（`EnvelopeMeta`）

**来源**: `internal/output/envelope.go#L40-L79`（`WriteSuccess`）

成功 envelope 包含：
- 身份（identity，固定 `type: "bot"`，见 `#L28`）
- data（业务数据）
- pagination（分页元数据，可选）
- rate-limit（限流元数据，可选）

## 二、错误 envelope

**来源**: `internal/output/envelope.go#L98-L127`（`WriteError`）

**错误类型枚举** `internal/output/errors.go#L12-L26`（`ExitError`）：
- `Type` —— 错误大类（auth / validation / api / network / ...）
- `Code` —— 具体错误码
- `Message` —— 用户可读消息
- `Hint` —— 修复建议
- `Status` —— HTTP 状态码（如果有）
- `OutcomeUnknown` —— 是否是"结果不确定"型（见 `#L27-L34`）

## 三、错误工厂函数

**来源**: `internal/output/errors.go#L77-L100`

| 函数 | 用途 |
|---|---|
| `ErrWithHint(typ, code, msg, hint)` | 通用错误 |
| `ErrAuth(msg, hint)` | 认证类错误 |
| `ErrValidation(msg, hint)` | 参数校验错 |
| `ErrAPI(code, msg, hint)` | 后端 API 错 |
| `ErrNetwork(msg, hint)` | 网络类错误 |

## 四、退出码（Exit Code）

**来源**: `internal/output/errors.go#L48-L60`（`ExitCode()`）

具体退出码映射规则见该函数实现。原则：
- 0 成功
- 非 0 分类不同错误类型（认证/验证/API/网络等）

## 五、错误码映射（后端错误 → CLI 错误）

**来源**: `internal/output/errors.go#L113-L160`（`backendErrorMapping` map）
**来源**: `internal/output/errors.go#L162-L182`（`ParseBackendError` / `ParsePublicAPIError`）

后端返回的错误 body 经过 `parseBackendError` 转成本地 `ExitError`，保持消息一致。

## 六、错误码是否已知

**来源**: `internal/output/errors.go#L275-L294`

- `IsErrorCodeShaped(s)` —— 判断字符串是否"看起来像错误码"格式
- `IsKnownErrorCode(s)` —— 是否在已知列表内

## 七、Hint / Type / Code from status

**来源**: `internal/output/errors.go#L331-L400`（`typeFromStatus` / `codeFromStatus` / `hintFromStatus`）

HTTP 状态码兜底映射（比如 401 → `auth` / `unauthenticated` / "请检查 token"）。

## 八、JQ 过滤器

**来源**: `internal/output/jq.go`

支持 `--jq` 参数在输出上做 jq 表达式过滤。

## 九、Normalize（数据规范化）

**来源**: `internal/output/normalize.go`

对后端返回数据做规范化处理（比如字段命名统一）。

## 十、常见问答备答

- **Q: envelope 长啥样？**
  A: 顶层含 identity / data / pagination（可选）/ rate-limit（可选）/ error（错误时）。见 `envelope.go#L40-L127`。

- **Q: 错误分几大类？**
  A: auth / validation / api / network 等，见 `errors.go` 里的 `ErrXxx` 工厂函数 `#L77-L100`。

- **Q: 退出码怎么定？**
  A: 见 `errors.go#L48-L60` 的 `ExitCode()`。

## 十一、我不确定的
- 完整 exit code 数值表（需精读 `ExitCode` 函数实现）
