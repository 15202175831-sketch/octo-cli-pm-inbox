# octo-cli-pm-inbox

> **AINOL Agent 实操考核** · Nati 的 octo-cli 产品管家需求池

这是一个为 [octo-cli](https://github.com/Mininglamp-OSS/octo-cli) 开源项目搭建的 Agent 产品管家系统的**需求池仓库**，由 3 个 Agent 协作维护：

- 🤖 **QA-Bot**：产品问答，每条结论给源码 `路径#L起-止` 可核验引用
- 🤖 **PM-Bot**：反馈收单 → 分类 → PRD 撰写
- 🤖 **RV-Bot**：PRD 形式 review

## 目录结构

```
├─ README.md                    # 本文件
├─ .github/
│  ├─ ISSUE_TEMPLATE/           # bug / feature / question 模板
│  ├─ labels.yml                # label 定义（type / priority / status）
│  └─ scan-log/                 # Cron 扫描执行记录（给主考看的）
├─ docs/
│  ├─ agent-architecture.md     # 3 Agent 分工与协作
│  ├─ prd-template.md           # PRD 模板（只写 What）
│  └─ label-taxonomy.md         # label 体系说明
└─ knowledge/                    # octo-cli 产品知识（9 大块，每条带源码引用）
   └─ octo-cli/
      ├─ 01-credentials.md
      ├─ 02-config-env.md
      ├─ 03-transport.md
      ├─ 04-output-errors.md
      ├─ 05-common-flags.md
      ├─ 06-domains.md
      ├─ 07-install.md
      ├─ 08-storage.md
      └─ 09-skills.md
```

## Agent 分工

| Agent | 负责 | 关键 skill |
|---|---|---|
| QA-Bot | 群内产品问答 | `octo-cli-kb-search` + `octo-cli-kb-verify` |
| PM-Bot | 反馈收单 + PRD 撰写 | `ainol-intent-classifier` + `ainol-issue-manager` + `ainol-prd-writer` |
| RV-Bot | PRD 形式 review | `ainol-prd-reviewer` |

## Cron 体系（长时闭环）

| Cron | 频率 | 用途 |
|---|:---:|---|
| KB 同步 | 每 30 分钟 | pull upstream octo-cli，重建索引 |
| 需求池扫描 | 每 5 分钟 | 检出新 issue / label 变化 / 评论，回群 @ 相关人 + @ 主考 |
| 引用自校验 | 每小时 | 抽 5 条最近引用，校验路径+行号有效 |
| PRD 老化检查 | 每天 9:00 | 找长期停滞的 issue |

## 引用规范

**每条产品结论都必须给源码引用**，格式：

```
来源: <相对路径>#L<起>-L<止>
```

例：`来源: internal/output/envelope.go#L45-L78`

考官会拿这个路径去 octo-cli 仓库核对，路径不存在或行号对不上内容 → 这条证据不算。

## PRD 原则

**只写 What，不写 How。**

- ❌ 不写"用 Redis 缓存"、"加一张表"
- ❌ 不贴代码块、内部字段名
- ✅ 验收标准写成用户能感知的（例："3 秒内看到成功提示"，不写"接口返回 200"）

详见 [`docs/prd-template.md`](docs/prd-template.md)。

## 红线

- octo-cli 仓库只读，Agent 不修改上游
- Token 不进群、不进 git（走 openclaw secrets）
- 不编造引用（自校验 cron 兜底）
- GitHub 限流：Search 30/min，REST 5000/h

---

_考生：Nati (明略科技 FDE 产品部) · 协作 Agent：棉花糖 🍬 · 考试窗口：2026-09-07 ~ 09_
