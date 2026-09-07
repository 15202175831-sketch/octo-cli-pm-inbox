# 09 · Agent Skills（内嵌 Skill）

> **能答的**：内嵌哪几个 skill、怎么用、Skill 文档结构。
> **要引用的**：`skills/` `skills/skills.go` 各 skill 子目录 `SKILL.md`。

## 一、11 个内嵌 skill

**来源**: `skills/skills.go#L1-L13`

```go
//go:embed */*.md
var FS embed.FS
```

**关键设计**：**每个 skill 的 markdown 文档都在编译时嵌入到 CLI 二进制**，运行时无需附带外部文件。

### 内嵌 skill 清单

**来源**: `skills/` 目录

| Skill | 说明 |
|---|---|
| `octo-docs` | 文档域使用指南 |
| `octo-drive` | 网盘域使用指南 |
| `octo-files` | 文件上传下载指南 |
| `octo-html` | HTML 交互文档指南 |
| `octo-loop` | Loop（Fleet 控制面）指南 |
| `octo-mail` | 邮件相关能力指南 |
| `octo-marketplace` | Skill Marketplace 指南 |
| `octo-matter` | 待办任务指南（**跟着 matter 域一起暂扣**）|
| `octo-messaging` | 消息发送/搜索指南 |
| `octo-shared` | 跨 skill 共享参考文档 |
| `octo-summary` | Summary 指南（**跟着 summary 域一起暂扣**）|

## 二、Skill 文档结构

**来源**: 每个 skill 目录下的 `SKILL.md` + 若干参考文档（渐进披露）

以 `octo-docs/` 为例：
- `SKILL.md` —— skill 主入口
- `sheet.md` —— 表格能力的详细参考（渐进披露，按需加载）

## 三、`octo-cli skills` 子命令

**来源**: `cmd/skills.go` (需查看具体行数)

Agent 可通过 `octo-cli skills` 子命令：
- 列出所有内嵌 skill
- 读取指定 skill 的 SKILL.md
- 读取渐进披露的参考文档

## 四、加载模型

**来源**: `skills/skills.go` 头部注释

- 列表键（enumeration key）**只列 `*/SKILL.md`**（避免把参考文档误认为独立 skill）
- 参考文档（如 `octo-docs/sheet.md`）**只在明确请求时加载**（progressive disclosure）

## 五、为啥要内嵌 skill

- **零外部依赖** —— Agent 只装 CLI 就能用到官方推荐用法
- **版本一致** —— skill 文档随 CLI 版本一起分发，避免文档漂移
- **可离线** —— 内网/离线场景可用

## 六、常见问答备答

- **Q: 有几个内嵌 skill？**
  A: 11 个，见 `skills/` 目录。其中 `octo-matter` 和 `octo-summary` 跟着对应域暂扣。

- **Q: skill 文档在哪？**
  A: 编译时嵌入到二进制。用 `octo-cli skills` 子命令读取。见 `skills/skills.go`。

- **Q: 能加自定义 skill 吗？**
  A: 内嵌的不能改。外部 skill 通常由 Agent runtime（OpenClaw 等）管理，与 CLI 无关。

## 七、我不确定的
- `octo-cli skills` 子命令的完整子命令名（list / get / cat 等具体命名需查 `cmd/skills.go`）
- 内嵌 skill 是否有版本号
