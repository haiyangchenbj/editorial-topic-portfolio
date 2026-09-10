---
name: editorial-topic-portfolio
description: This skill should be used when evaluating a portfolio of
  technology, AI, data, cloud, or enterprise-software content topics. It
  normalizes Notion or Markdown/JSON/CSV inputs, applies factual, timeliness,
  originality, argument-capacity, positioning, and privacy gates, scores
  eligible topics, selects one primary and two backups, produces
  merge/watch/covered/abandon decisions,   and prepares a confirmed Notion
  writeback preview. By default, it never writes to Notion without two explicit
  confirmations; a workspace with a documented standing instruction to sync
  every routine review may use that instruction as the authorization, but must
  still generate a change preview and perform readback verification.
description_zh: 选题组合评估与写作排期
description_en: Editorial topic portfolio
version: "1.0.5"
not_for:
  - Writing, editing, or laying out a single article (use the writing or publishing pipeline skills)
  - Publishing or distributing content to platforms
  - Developing Notion databases or API integrations
  - Performance review of already-published content
agent_created: true
read_when:
  - 选题组合
  - 选题评估
  - 本周写什么
  - 选题优先级
  - Notion 选题库复核
  - 内容排期
  - editorial topic portfolio
  - topic prioritization
---

# Editorial Topic Portfolio

对一批科技、AI、数据、云和企业软件选题进行组合评估，输出当前周期 1 个主写、2 个备选，以及合并、观察、短观察、已覆盖跟踪和放弃决定。采用 Routing + Prompt Chaining，先过硬门禁，再评分和组合求解。

## When to use

- 需要从 Notion 选题库或本地文件中确定本周/下周写作组合。
- 需要判断选题是独立长文、并入其他文章、缩为短观察、继续观察还是关闭。
- 需要核对竞争密度、原创空间、事实准备度、时效窗口和定位匹配。
- 需要在回写前生成 Notion 字段变更预览。

## Do not use

- 直接写正文；主写选题交给 `industry-deep-dive-pipeline`。
- 直接生成微信排版、摘要、封面、社交物料或发布内容。
- 未经确认直接写入 Notion（除非当前工作区已有明确、持久化的例行回写授权）。
- 只依据热度、单个数字或标题传播性排序。

## Inputs

### Supported sources

- Markdown、JSON、CSV：直接转换为标准 topic record。
- Notion：通过私有适配器读取，凭据、数据库 ID 和字段 mapping 只来自安全环境变量或私有配置。

### Required fields

- topic title
- source date and captured date
- lifecycle status
- topic type
- source/evidence
- hypothesis or value judgment
- related published content

读取 `references/evaluation-gates.md`、`references/scoring-model.md` 和 `references/portfolio-rules.md` 获取字段与规则。

## Workflow

### Step 1: [Deterministic] Normalize input

1. 路由 Notion 或文件输入。
2. 运行：

```bash
python scripts/normalize_topics.py --input <input.md|input.json|input.csv> --output <run-dir>/01-normalized-topics.json
```

3. 去重标题、URL、事件主体和同义主题。
4. 保留稳定 topic ID 或 Notion page ID；标题只用于展示。

### Step 2: [Deterministic + LLM] Pre-screen

1. 执行 G0 输入完整性检查。
2. 区分 event、trend、framework、case、policy、research。
3. 检查生命周期状态，识别已发布、已采纳、观察和关闭项。
4. 输出 `02-gate-precheck.json`。

### Step 3: [LLM + Search] Research evidence and competition

1. 重新核实关键事实、日期、政策状态、产品状态和金额口径。
2. 搜索中文与英文的主题、同义判断、分析框架、标志性表达、反方和历史反例。
3. 检查与已发布、在写和同批选题的重叠。
4. 为观察项确定触发条件、所需证据和复核日期。
5. 输出 `03-evidence-review.md`。

### Step 4: [LLM] Apply hard gates

对每条选题输出 G0—G5：

- G0 输入完整性。
- G1 事实与状态。
- G2 时效与触发。
- G3 原创空间。
- G4 长文承载力。
- G5 定位与隐私边界。

输出 `04-gate-results.json`。BLOCK 不得靠评分抵消。

### Step 5: [Deterministic] Calculate scores

为通过硬门禁或可恢复 HOLD 的选题填写 0—5 原始分：

- 原创空间与信息增量：20。
- 读者判断价值：20。
- 事实与证据准备度：15。
- 时效与窗口：15。
- 内容定位匹配：15。
- 组合贡献：10。
- 执行可行性：5。

扣除事实不确定、竞争升温、角度过宽、近期重复和公开风险，得到净分。运行：

```bash
python scripts/calculate_scores.py --input <run-dir>/04-gate-results.json --output <run-dir>/05-scores.json
```

### Step 6: [LLM + Deterministic] Build portfolio

1. 选择恰好 1 个 `WRITE_NOW` 和 2 个 `BACKUP`。
2. 约束主写与备选不能全部属于同一公司、事件或论证框架。
3. 其余选题标记 `MERGE`、`WATCH`、`SHORT_NOTE`、`COVERED_TRACK` 或 `ABANDON`。
4. 写明每条决策的事实、原创、时效和组合原因。
5. 输出 `06-portfolio-decision.md`。

### Gate A: [Human] Confirm portfolio

暂停，确认：

- 1 个主写和 2 个备选。
- P0—P3 调整。
- 合并、观察、已覆盖和放弃处理。
- 人工覆盖理由和有效期。
- 计划回写字段。

确认前不得生成回写变更集。

### Step 7: [Deterministic] Generate change preview

生成 `07-change-preview.md` 和 `07-change-set.json`，逐条列出：

- page ID 或稳定 topic ID。
- 状态、优先级、竞争密度、原创性、价值、定位匹配和评估日期的旧值/新值。
- 推荐原因、建议角度和风险提醒的变更。
- 幂等评估块。

运行：

```bash
python scripts/validate_change_set.py --input <run-dir>/07-change-set.json --output <run-dir>/07-change-set-validation.json --enforce
```

### Gate B: [Human] Confirm writeback

明确列出目标数据库、页面数量、字段和动作。只有第二次确认后才允许批量回写。

### Step 8: [Deterministic] Writeback

Notion 写回必须遵守：

- `NOTION_TOKEN`、`NOTION_TOPIC_DATABASE_ID` 和 mapping 来自安全配置。
- 使用 page ID，禁止模糊标题写入。
- 预览和 dry-run 不得发送 PATCH。
- 失败即暂停批次，记录 succeeded/failed/skipped。
- 评估块按 run ID 和日期幂等替换，禁止重复追加。

默认只读适配器：

```bash
NODE_PATH=<managed-node-workspace> node scripts/notion_topic_adapter.js read > <run-dir>/08-notion-read.json
```

写回能力必须由单独、已审阅的 change set 驱动；不得把实时写回做成无条件自动动作。

### Step 9: [Deterministic] Readback

- 重新查询已更新 page ID。
- 对比预览与实际字段。
- 不一致时重试一次并换请求路径交叉验证。
- 回读不一致时不得标记完成。

输出 `09-readback-verification.md`。

### Step 10: [Deterministic] Handoff primary

把主写转换成 `industry-deep-dive-pipeline` 的 case brief，保留事实、竞争、核心假设、建议角度、风险、反方和系列关系。不直接写正文。

## Hard Rules

1. 硬门禁先于评分；BLOCK 不得被高分抵消。
2. 事实、状态和日期必须回到可靠来源；搜索摘要只作线索。
3. lifecycle status 与 recommendation priority 必须分开。
4. 每轮必须是 1 主写 + 2 备选，除非人工覆盖并记录原因。
5. 已发布选题默认进入 `COVERED_TRACK`，除非有明确新事实或新判断。
6. 观察项必须写触发条件、所需证据和复核日期。
7. 使用 page ID 写入；标题不得作为唯一写入主键。
8. 任何 Notion 写回必须经过组合确认和回写确认。
9. 凭据、数据库 ID、个人选题和私有定位不得进入通用 Skill 或公开模板。
10. 工具首次失败必须重试并交叉验证；修改后必须重跑。
11. 不自动回退已采纳、已发布和关闭状态。
12. 任务完成前必须记录输出、验证、遗留风险和下一项任务。

## Failure Handling

| 场景 | 处理 |
|---|---|
| 输入字段缺失 | 标记 `NEEDS_INPUT`，停止该条评分 |
| 事实或政策无法确认 | `BLOCK_FACT` 或 `HOLD_EVIDENCE`，不进入主写/备选 |
| 时效已过且无增量 | `BLOCK_STALE` 或 `COVERED_TRACK` |
| 原创空间不足 | `MERGE` 或 `BLOCK_SATURATED` |
| 单一事件承载力不足 | `SHORT_NOTE` 或 `BLOCK_THIN` |
| 定位/隐私冲突 | `BLOCK_BOUNDARY` 或 `PRIVATE_ONLY` |
| 组合没有恰好 1+2 | 阻断变更集，重新求解或等待人工覆盖 |
| Notion token/mapping 缺失 | 停止，不读取、不写入 |
| Notion 写回部分失败 | 停止批次，记录成功/失败/跳过，不盲目重试全部 |
| 回读不一致 | 重试一次并交叉验证，仍不一致则保持未完成 |
| 工具首次失败 | 同操作重试 1—2 次，再换工具栈 |

## Output Format

```text
<run-dir>/
├── 01-normalized-topics.json
├── 02-gate-precheck.json
├── 03-evidence-review.md
├── 04-gate-results.json
├── 05-scores.json
├── 06-portfolio-decision.md
├── 07-change-preview.md
├── 07-change-set.json
├── 07-change-set-validation.json
├── 08-writeback-result.json
└── 09-readback-verification.md
```

每轮汇报必须包含：主写、两个备选、其他处置、硬门禁阻断项、净分、回写状态和下一步交接。

## References

- `references/evaluation-gates.md`：G0—G5 硬门禁。
- `references/scoring-model.md`：100 分模型与扣分。
- `references/portfolio-rules.md`：组合、生命周期和幂等规则。
- `references/notion-adapter-interface.md`：私有 Notion 适配边界。
- `references/replay-evaluation.md`：两组历史回放。

## Pitfalls

- 把状态和推荐优先级混为一谈。
- 用标题模糊匹配 Notion 页面并写错对象。
- 通过追加备注造成每次运行重复堆积。
- 用一个聚合平台的调用量推导全球市场份额或定价权。
- 把已发布选题重新列为主写。
- 评分很高但事实、政策或金额口径不稳。
- 只按新闻热度排序，忽略读者判断价值和原创空间。
- 先回写 Notion，再让用户确认组合；例外是工作区已明确授权例行复核自动同步，此时仍要先完成组合判断、生成变更记录并做回读。

## Verification

- [ ] Notion/文件输入均可转换为标准 topic record。
- [ ] 每条选题有 G0—G5 结果、基础分、扣分、净分和处置动作。
- [ ] 每轮恰好 1 主写 + 2 备选，或有明确人工覆盖。
- [ ] 主写与备选不存在同事件/同框架内部竞争。
- [ ] 事实阻断项没有进入 P0/P1。
- [ ] 变更集只使用 page ID 或稳定 topic ID。
- [ ] 预览、组合确认和回写确认均有记录。
- [ ] 重复运行不会重复追加评估块。
- [ ] 回写后回读字段一致率为 100%。
- [ ] 真实 token、数据库 ID 和私人选题数据未进入 Skill 目录。
