# 06 · 功能域与操作（Domains & Operations）

> **能答的**：有哪几个域、各多少 ops、哪些"暂时不可用"。
> **要引用的**：`README.md` 域表、`internal/registry/loader.go`、`internal/registry/specs/`。

## 一、12 个功能域总表

**来源**: `README.md#L52-L67`（Domains 表）

| 域 | Ops 数 | 说明 | 状态 |
|---|:---:|---|:---:|
| `docs` | 32 | 文档/表格/白板生命周期、全文搜索、正文、单元格、场景、成员、评论、版本、附件 | ✅ |
| `html` | 20 | 交互式 HTML 文档 octo-doc（**独立后端**）| ✅ |
| `drive` | 43+3=46 | 网盘（空间、成员、文件树、搜索、上传下载、挂载、分享、邀请、IM 附件转存）| ✅ |
| `matter` | 14 | 待办/任务 | ⚠️ **暂扣**（后端稳定中）|
| `summary` | 4 | Personal-bot 摘要 | ⚠️ **暂扣**（后端 PR #181 待合并部署）|
| `group` | 9 | 群 —— list/get/members/metadata/create/update | ✅ |
| `thread` | 8 | Thread —— create/list/get/members/join/leave/metadata | ✅ |
| `bot` | 6 | Bot 生命周期 —— register/user-info/space-members/heartbeat | ✅ |
| `message` | 10 | 消息 —— send/edit/sync/read-receipt/search（含 files/media/around/groups）| ✅ |
| `file` | 4 | 文件 —— upload/download/credentials/presigned | ✅ |
| `event` | 2 | 事件 —— list/ack | ✅ |
| `loop` | 126 | Fleet 控制面（tasks/executions/experts/teams/workspaces/runtimes/projects/skills/automations/comments/labels 等）| ✅ |

**"暂时不可用"（考试红线之一：不能推荐用户用这些）**：
- `matter` —— backend API 稳定中
- `summary` —— create backend (Mininglamp-OSS/octo-smart-summary#181) 待部署

## 二、Registry 加载机制

**来源**: `internal/registry/loader.go#L18-L19`

```go
//go:embed specs/*.json
var specsFS embed.FS
```

**OpenAPI 3.x 规格文件在编译时嵌入到二进制**（`internal/registry/specs/` 目录），启动时自动扫描注册所有命令树。

**来源**: `internal/registry/loader.go#L31-L61`（`New()` 加载流程）

1. 读 `specs/*.json` 每个文件
2. 按文件名（去掉 `.json`）作为服务名（域名）
3. Parse OpenAPI 文档存到 `map[string]map[string]any`
4. 校验重复 operationID
5. 校验 conditional query 元数据

## 三、命令树自动注册

octo-cli 是 **metadata-driven** ——加/改 endpoint 只改 spec 不改代码。

**来源**: `README.md#L15-L23`

```
OpenAPI specs → Registry → Service Engine → Factory → Client → Output
(embedded)      (parsed)    (cobra commands) (DI)     (HTTP)   (envelope)
```

## 四、`docs` vs `html`

**关键区别**：`docs` 和 `html` 是**两个独立后端**：
- `docs` —— 传统文档/表格/白板
- `html` —— octo-doc 交互式 HTML 文档

不能用一个域的接口调另一个的资源。**来源**: `README.md#L54-L55`

## 五、`drive` 的复合命令

**来源**: `README.md#L56`

drive 43 个 ops + **3 个复合命令**（`upload file` / `download file` / `share create`）= 46 leaves。

复合命令是本地组合多个 API 调用为一个易用命令的场景。

## 六、常见问答备答

- **Q: 有几个域？**
  A: 12 个，见 README#L52-L67。（考试原文说"九块知识"是指知识分类，不是域数）

- **Q: 哪些用不了？**
  A: `matter`（14 ops）和 `summary`（4 ops）暂扣，backend 稳定中。见 README#L56, L58。

- **Q: 怎么新增一个 endpoint？**
  A: 改 `internal/registry/specs/<service>.json` 里的 OpenAPI 定义，重编译即可。不改 Go 代码。见 README#L15-L23。

## 七、我不确定的
- 每个域下具体的 operationID 完整列表 —— 需实际 dump `specs/*.json`
