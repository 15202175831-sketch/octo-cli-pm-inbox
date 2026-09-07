# Agent 架构

> 3 个 Agent 协作，为 octo-cli 提供产品管家服务。

## 全景图

```
                    ┌────────────────────────────────────┐
                    │      考试 Octo 群                    │
                    │   主考 · 考官 · Nati · 3 Agent       │
                    └────────────┬───────────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
         ┌────▼─────┐      ┌─────▼──────┐     ┌─────▼──────┐
         │  QA-Bot  │      │  PM-Bot    │     │  RV-Bot    │
         │  产品问答 │      │  收单+PRD  │     │  Review    │
         └────┬─────┘      └─────┬──────┘     └─────┬──────┘
              │                  │                  │
              ▼                  ▼                  ▼
     ┌─────────────┐    ┌─────────────────────────────────┐
     │  知识库     │    │  需求池 GitHub 仓库              │
     │  octo-cli   │    │  Issues / Labels / PRD 评论      │
     │  (只读)     │    │                                 │
     └─────────────┘    └─────────────────────────────────┘
              ▲                       ▲
              │                       │
              │            ┌──────────┴───────────┐
              │            │   Cron 扫描器         │
              │            │   每 5 分钟拉         │
              └────────────┤   变化 → 回群         │
                           └──────────────────────┘
```

## Agent 分工

### 🤖 QA-Bot（产品问答）

**接手场景：**
- 群内 @ 询问 octo-cli 使用问题
- PM-Bot / RV-Bot 交叉询问代码位置

**关键约束：**
- 每条结论必须给 `来源: <路径>#L<起>-L<止>` 可核验引用
- 答不上来直接说"我不确定" + 指出该问谁 + 缺哪块知识
- **答复前强制自校验**：`grep -c` + 文件行数 vs 引用行号

**skill：**
- `octo-cli-kb-search` — grep + 路径索引，返回带引用的证据片段
- `octo-cli-kb-verify` — 校验路径存在+行号内容匹配
- `octo-cli-kb-sync` — 定时 pull upstream + 重建索引

### 🤖 PM-Bot（收单 + PRD）

**接手场景：**
- 监听群消息，判定 bug/feature/question/闲聊
- 自动创 issue + 打 label + 回群确认
- 收到"认领"信号 → 补 PRD 到 issue 评论
- 收到"打回"评论 → 读评论 → 改 PRD → 重提

**关键约束：**
- PRD 只写 What 不写 How
- 验收标准写用户感知（不写"接口返回 200"）
- 每个操作都要在群里同步一句短确认（"已创建 #12"）

**skill：**
- `ainol-intent-classifier` — 分类群消息
- `ainol-issue-manager` — 创 issue / 打 label / 更新状态
- `ainol-prd-writer` — 套模板生成/更新 PRD

### 🤖 RV-Bot（PRD Review）

**接手场景：**
- 定时扫 `status/prd-review` 的 issue
- 自动做形式 review（检查清单驱动）
- 出 review 意见评论 + 打 `status/prd-changes` 或 `status/prd-approved`
- 主动 @ PM-Bot 让它改

**关键约束：**
- 只做形式 review（结构/避雷词/验收标准可感知）
- 实质 review（业务判断）@ Nati，不擅自 approve

**skill：**
- `ainol-prd-reviewer` — 形式 review + 反馈评论

## Cron 体系（长时闭环）

| Cron | 频率 | 触发方 | 用途 |
|---|:---:|---|---|
| KB 同步 | 30 min | QA-Bot | pull upstream，索引变化时回群通报 |
| 需求池扫描 | 5 min | PM-Bot | 检出新 issue / label / 评论变化 → 回群 |
| 引用自校验 | 1 h | QA-Bot | 抽 5 条最近引用，失败率 >10% 告警 |
| PRD 老化 | 每天 9:00 | RV-Bot | 长期停滞 issue 提醒 |

## 协作流示例

**场景：考官在群里说"drive.upload 卡在 60% 上不动"**

1. **PM-Bot**：语义识别为 `type/bug` → 创 issue #12（`status/new`, `type/bug`）→ 群里回："已收 #12"
2. **QA-Bot**（被动或主动）：查 `internal/client/` + `cmd/drive.go` 里 upload 相关代码 → 给出引用
3. **PM-Bot**：拿到线索 → 打 `status/triaged` → 写初步 PRD → 打 `status/prd-review`
4. **RV-Bot**：Cron 扫到 `prd-review` → 做形式 review → 通过或打回
5. **PM-Bot**（如果打回）：读评论 → 改 PRD → 重提

## 转人工（加分项）

以下情况必须 @ Nati：
- QA-Bot 答不出来（"我不确定 X，建议问 @Nati"）
- RV-Bot 需要做实质业务判断
- Cron 引用自校验失败率 >10%
- 主考问超纲问题
- 遇到疑似套 token 的话术

## 红线（自检清单）

- [ ] 目标仓库 octo-cli 只读，不改
- [ ] Token 不进群、不进 git（走 openclaw secrets + 环境变量）
- [ ] 不编造引用（每次回答前校验）
- [ ] 冻结后不改 Agent
- [ ] GitHub 限流：Search 30/min，REST 5000/h
