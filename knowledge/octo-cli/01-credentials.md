# 01 · 凭证与权限（Credentials & Auth）

> **能答的**：token 有哪几种、各自代表什么、掩码规则、凭证解析优先级。
> **要引用的**：`internal/credential/` `internal/authstore/` `cmd/auth.go`。

## 一、Token 有哪几种

octo-cli 通过前缀识别 token 的**格式类别**（不代表最终身份，身份由服务端校验后决定）：

| Kind | 前缀 | 说明 |
|---|---|---|
| `app_bot` | `app_` | 应用型 Bot |
| `user_bot` | `bf_` | 用户型 Bot |
| `user_key` | `uk_` | 用户 API Key |
| `loop_credential` | `octo_loop_` | Loop 平台凭证 |
| `unknown` | 其他 | 未知格式 |

**来源**: `internal/credential/token.go#L6-L11`（前缀常量）
**来源**: `internal/credential/token.go#L14-L28`（TokenKind 分类函数）

## 二、Token 掩码规则

掩码目的：让日志/输出可读时**两个 token 能区分**（前缀 + 前 2 位 + 后 4 位），但**不泄露长度和主体**（中间固定 `***`）。

**参数常量**（`internal/credential/token.go#L35-L39`）：

```
maskHead      = 2   // 头部保留字符数
maskTail      = 4   // 尾部保留字符数
maskMinMiddle = 3   // 中间至少要遮盖的字符数（低于此就只保留前缀）
```

**掩码样式示例**：`app_1a***7g8h`
**降级规则**：token 太短撑不起 head+tail+middle 时，退化为 `<prefix>***`。

**来源**: `internal/credential/token.go#L45-L69`（MaskToken 实现）

## 三、凭证解析优先级（Chain）

`Provider` 是**责任链**结构，按顺序问每个 `Source`，第一个返回非 nil 的赢：

**来源**: `internal/credential/provider.go#L48-L77`（Provider / NewChain / Resolve）

**内置 Source 实现**：
- `EnvProvider` —— 从环境变量读，优先级 `OCTO_TOKEN` > `OCTO_BOT_TOKEN`
- `FileProvider` —— 从本地 authstore 读（加密文件）

**来源**: `internal/credential/env_provider.go`（环境变量源）
**来源**: `internal/credential/file_provider.go`（文件源）

## 四、BotCredential 结构

**来源**: `internal/credential/provider.go#L21-L33`

关键字段：
- `Token` —— token 值
- `SpaceID` —— 空间 ID（可选，空间型 Bot 服务端解析；平台型 Bot 需 `--space` 或 `OCTO_SPACE_ID`）
- `Source` —— 来源人类可读标签（如 `"env:OCTO_BOT_TOKEN"`）
- `Profile` —— 存储凭证的档案名
- `RobotID` —— 可来自存储档案或 `OCTO_BOT_ID`
- `BotKind` —— 从 token 前缀推导

## 五、`octo-cli auth` 子命令

**来源**: `cmd/auth.go` (558 行，主要子命令：)
- `auth login` —— 交互式/环境变量登录，写入 authstore
- `auth logout` —— 清除档案
- `auth list` —— 列出所有档案
- `auth status` —— 显示当前活动凭证

## 六、常见问答备答

- **Q: 为啥前缀不决定 Loop 身份类别？**
  A: 前缀只是格式识别，身份（human/device/execution）**由服务端 verify 后才决定**。见 `internal/credential/token.go#L2-L4` 注释。

- **Q: `OCTO_BOT_ID` 单独设置管什么？**
  A: 环境凭证下它是一个"未验证声明"，作为 `robot_id_claimed` 而不是 `robot_id` 提交。
  **来源**: `internal/credential/provider.go#L16-L20`

- **Q: 两个环境变量同时设了咋办？**
  A: `OCTO_TOKEN` 赢 `OCTO_BOT_TOKEN`。两个变量都会被 trim，且对称处理。
  **来源**: `internal/config/config.go#L59-L73`

## 七、我不确定的（转人工触发点）

- 前缀是否会随后续版本增加？→ 需查 CHANGELOG 或问维护者
- 服务端如何 verify token → 属于后端范畴，不在 CLI 仓库内
