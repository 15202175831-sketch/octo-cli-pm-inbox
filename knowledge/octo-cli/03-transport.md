# 03 · 传输与重试（Transport & Retry）

> **能答的**：默认超时、重试次数、退避策略、`Retry-After` 处理、密钥屏蔽。
> **要引用的**：`internal/client/client.go`（1362 行核心）。

## 一、默认参数

**来源**: `internal/client/client.go#L30-L45`

```go
defaultTimeout = 30 * time.Second
```

设计文档：**`docs/architecture-design.md §6.2`** 定义了完整重试规则。

## 二、请求级选项

**来源**: `internal/client/client.go#L38-L90`

- `NoRetry` —— 全局禁用重试的 flag
- `Timeout` —— 超时时长（构造时解析一次）
- `DisableRetry` —— 单请求级禁用重试（独立于全局开关）

## 三、退避策略

**来源**: 见 `client.go` 头部注释 `#L1-L5`

> "module-qualified paths, retry with exponential backoff + jitter, Retry-After, ..."

即：
- **指数退避 + 抖动**（Exponential backoff + jitter）
- 尊重服务端 `Retry-After` 响应头
- 特定错误码不重试（详见 `output/errors.go`）

## 四、密钥屏蔽（防泄漏）

`client.go` 里有一大块专门做**日志中的密钥屏蔽**：

**来源**: `internal/client/client.go#L125-L410`（redactSecrets / maskOrSuppressValue 等一系列函数）

关键设计：
- 请求/响应 body 都过 `redactBodyForLog` 屏蔽
- 不仅屏蔽完整 token，还屏蔽变形（比如 URL-escape 版本）
- `secretForms()` 生成一个 token 的所有变形，逐一替换

**为啥这么重要**：合规——**任何日志、错误、调试输出都不能出现 token 原文**。
**辅助测试**: `internal/client/secretmask_test.go` (1430 行) `secretcensus_test.go` (278 行)

## 五、Search 路由

**来源**: `internal/client/search_route.go` (88 行)

Search API 有独立路由逻辑（限流敏感，GitHub Search 30 次/分钟同类约束）。

## 六、限流

- **GitHub Search API**: 30 次/分钟
- **GitHub REST API**: 5000 次/小时

Agent 侧要遵守这个约束（**这是考试红线之一**）。

## 七、常见问答备答

- **Q: 默认超时多久？**
  A: 30 秒。见 `internal/client/client.go#L35`。

- **Q: 重试策略是什么？**
  A: 指数退避 + 抖动 + 尊重 `Retry-After`。见 `internal/client/client.go#L1-L5` 头部注释；完整规则在 `docs/architecture-design.md §6.2`。

- **Q: 怎么禁用重试？**
  A: 全局 `--no-retry` flag 或单请求 `DisableRetry`。见 `client.go#L43,L84-L89`。

## 八、我不确定的
- 具体重试次数上限（需查 `architecture-design.md §6.2`）
- 每个 status 是否重试的完整分类表
