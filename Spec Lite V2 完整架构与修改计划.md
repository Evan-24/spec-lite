# Spec Lite V2 完整架构与修改计划

## 1. V2 的核心问题

Spec Lite V1 已经解决了一个基础问题：

> 如何用尽可能少的持久上下文，让开发者和 Agent 跨会话继续完成同一个任务。

V1 的基本模型是：

- `PROJECT.md` 保存项目级导航。
- `Task Spec` 保存任务目标、范围、边界和验收标准。
- `AGENTS.md` 保存长期开发规则。
- `Decision` 保存重要且长期有效的设计理由。
- 代码、测试和真实配置仍然是主要事实源。
- 实施计划、进度日志和执行过程不进入 Spec。

这一设计能够避免传统 Spec Framework 中常见的重流程问题。

但真实使用后暴露出了三个更深层的问题。

### 1.1 代码之外存在高价值现实事实

例如：

- 项目全量测试长期存在一批与当前修改无关的失败。
- 某个企业环境下 PowerShell 存在固定限制。
- 某个 Runtime 和依赖版本组合存在稳定行为。
- 某个外部系统返回格式与文档描述不一致。

这些信息：

- 无法轻易从代码恢复；
- 重新调查非常消耗时间和 Token；
- 又很可能被未来 Agent 重复遇到。

V1 没有合适的位置保存它们。

### 1.2 大型任务存在阶段上下文问题

复杂任务往往经历：

```
需求讨论
→ 架构探索
→ 多轮方案比较
→ 形成宏观计划
→ 开发者审核
→ 实施
```

到真正开始实施时，原对话可能已经包含大量：

- 被否决的方案；
- 临时假设；
- 重复解释；
- 已过时的分析；
- 详细规划过程。

继续直接在原对话执行，会让 Agent 背负大量低价值历史上下文。

但如果直接新建对话，又可能丢失已经审核过的重要实施方向。

### 1.3 完整执行计划不适合长期保存

另一种极端是把详细 Plan 保存下来，让后续 Agent 严格执行。

例如：

```
Step 1 修改 A 文件
Step 2 创建 helper
Step 3 修改 B 文件
Step 4 执行测试
...
```

这种计划高度依赖当前代码结构，很快腐化。

它还会削弱强模型根据真实代码重新推理的能力。

因此 V2 不能走向：

> 将详细实施计划持久化并要求 Agent 严格执行。

V2 应解决的是：

> 如何保留真正不能丢失的信息，同时允许 Agent 在当前现实基础上重新推理。

# 2. V2 的总体定位

Spec Lite V2 不定位为：

> 一个 lightweight spec framework。

更准确的定位是：

> **A lightweight context distillation protocol for coding agents.**

即：

> 一套帮助开发者和 Agent 判断“哪些信息值得跨上下文保留”的轻量协议。

Spec Lite 不负责控制模型如何思考、如何编码、如何生成详细 Plan。

它负责的是：

> 在阶段、会话和 Agent 切换时，尽量减少高价值信息损失。

# 3. V2 的核心哲学

V2 延续并强化以下原则。

## 3.1 强模型，轻框架

默认相信模型能够：

- 阅读当前代码；
- 理解当前结构；
- 动态生成实施步骤；
- 根据实现过程中出现的新信息调整计划。

Spec Lite 不重复管理模型已经能够动态完成的工作。

## 3.2 保存结论，不保存讨论

核心原则：

> **Preserve decisions, not conversations.**

长对话本身不应该成为下一阶段必须携带的上下文。

应从讨论中蒸馏真正重要的信息。

## 3.3 保存承诺，不保存思考轨迹

> **Preserve commitments, not reasoning history.**

如果开发者已经明确审核并确认：

```
不要修改 runtime core
保持 Linux/macOS 行为不变
Windows compatibility 放在 command boundary
```

这些应该被保留。

但模型如何经过五轮推理得出这些结论，不需要保存。

## 3.4 保存契约，不保存执行脚本

> **Preserve contracts, not execution scripts.**

Spec 描述：

- 要实现什么；
- 不允许破坏什么；
- 什么算完成。

而不是：

- 第一步打开哪个文件；
- 第二步改哪个函数；
- 第三步运行哪个命令。

## 3.5 上下文应在阶段边界重置

> **Reset context at phase boundaries.**

从：

```
探索 / 讨论
```

进入：

```
正式实施
```

是天然的上下文切换点。

从一个大型阶段进入另一个阶段，同样可以重新建立干净执行上下文。

是否切换不由固定 Token 比例决定，而由语义阶段决定。

# 4. V2 完整知识模型

V2 将信息分成六类。

```
PROJECT
AGENTS
FACTS
DECISIONS
SPEC
CODE / TESTS / CONFIG
```

它们分别承担不同职责。

# 5. PROJECT.md

回答：

> Where is everything?

保存项目级稳定导航：

- 项目目的；
- 技术栈；
- 核心模块；
- 主要入口；
- 重要流程；
- 长期有效的项目级结构信息。

不保存：

- 当前任务细节；
- 测试基线；
- 调试结论；
- 详细架构讨论；
- 实施计划。

`PROJECT.md` 仍然是可选导航能力，而不是任务执行前置条件。

# 6. AGENTS.md

回答：

> How must work be done?

保存：

- Agent 必须长期遵守的项目规则；

- 项目明确要求的开发约束；
- 稳定工程惯例。

例如：

```
所有新增 Python 工具必须通过 uv 管理依赖。
Windows 命令执行不得直接假设 PowerShell 7。
```

不保存：

- 当前任务计划；
- 临时 workaround；
- 某次测试失败；
- 单任务经验。

# 7. DECISIONS

回答：

> Why was this chosen?

保存重要且长期有效的设计决策。

例如：

```
为什么采用 bundled Bash，而不是继续扩展 PowerShell compatibility。
```

Decision 只有当其理由未来仍然值得知道时才创建。

普通实现选择不应形成 Decision。

# 8. FACTS.md

V2 新增的可选长期知识层。

回答：

> What has reality taught us?

它保存：

> 无法轻易从代码重新获得，但通过实际运行、调查或实验得到，并且未来很可能再次有用的现实事实。

典型内容包括：

- 已知测试失败基线；
- Runtime 实测行为；
- 企业环境限制；
- 平台兼容性；
- 外部服务行为；
- 特定依赖组合的问题；
- 重复出现的环境噪声。

# 9. Fact 的准入标准

一条事实只有同时满足以下条件，才值得进入 `FACTS.md`：

1. 无法轻易从代码、测试定义或配置中直接获得；
2. 重新发现存在明显成本；
3. 未来任务很可能再次遇到。

例如：

值得记录：

```
Windows 企业环境下，全量测试长期存在一组与普通 feature 修改无关的 sandbox 失败。
```

不值得记录：

```
今天 npm install 比较慢。
```

# 10. Fact 的最小结构

Fact 不采用复杂 Schema。

只需要回答：

```
What was observed?
How can it be recognized?
When should it be revalidated?
```

推荐结构：

```
## Windows full-test baseline

Observed:

`pnpm test` contains known pre-existing failures related to the
corporate Windows environment.

Recognize by:

- `sandbox.test.ts > PowerShell policy`
- `/ECONNREFUSED.*127.0.0.1:6379/`
- `windows-ca.test.ts > certificate store`

Verified:

2026-09-29 on Windows 10 / Node 24

Revalidate when:

- Node/runtime changes
- test configuration changes
- Windows execution handling changes
- known signatures disappear or change
- new failure signatures appear
```

# 11. Fact 的生命周期

Facts 不由时间驱动维护。

原则：

> **Facts are learned and invalidated by work, not maintained by time.**

生命周期：

```
Observe
  ↓
Record
  ↓
Reuse
  ↓
Revalidate
  ↓
Update / Remove
```

# 12. Observe

Agent 在正常工作过程中遇到现实行为。

例如：

```
运行测试
→ 41 failures
→ 调查发现其中 38 个是项目既有问题
```

此时产生 Fact 候选。

# 13. Record

Agent 可以主动记录事实，不必每次询问用户。

但必须满足 Fact 准入标准。

如果只影响当前任务，则记录到当前 Spec 的可选：

```
## Verified Facts
```

如果明显会影响多个未来任务，则进入：

```
spec-lite/FACTS.md
```

# 14. Reuse

当 Agent 遇到：

- 测试失败；
- Runtime 异常；
- 工具链问题；
- 平台差异；
- 环境异常；
- 外部服务异常；

在投入大量调查前，应优先查看相关 Fact。

例如：

```
Current:
38 failures

FACTS:
38 known signatures

Result:
0 new relevant failures
```

则无需重复调查全部失败。

# 15. Revalidate

Fact 通过事件驱动重新验证。

主要有三类触发。

## Observation mismatch

当前现实与记录不一致。

例如：

```
Fact: 38 known failure signatures
Current: 41 failures
```

Agent 只调查 delta。

## Causal change

当前任务修改了可能影响 Fact 的代码、环境或依赖。

例如：

```
Fact 基于 Electron 41
当前升级 Electron 43
```

则重新验证相关事实。

## Explicit challenge

用户或 Agent 主动质疑某条事实。

例如：

```
这些 baseline failures 是否已经修复？
```

重新验证。

# 16. Fact 不采用 TTL

日期不是自动失效机制。

原则：

> **Age is evidence for suspicion, not a trigger for revalidation.**

一条 Fact 很旧，只说明：

> 如果当前环境已经发生相关变化，应提高警惕。

不意味着：

> 每隔 N 天自动重新运行一遍验证。

# 17. Fact 更新与删除

`FACTS.md` 只保存当前仍然有用的信息。

如果原来：

```
Known failures:
A
B
C
```

B 已经修复，则改成：

```
Known failures:
A
C
```

不追加历史日志。

如果 Fact 已完全失效，则删除。

不引入：

- Deprecated
- Superseded
- Archived Fact
- Fact History

历史由 Git 保存。

# 18. Spec 的新角色

V2 的 Spec 继续表示：

> 一个可验证的任务契约。

普通 Spec 结构仍然保持：

```
Goal
Scope
Non-goals
Constraints
Acceptance Criteria
```

必要时可以增加：

```
Verified Facts
```

但该区块不进入默认 Task 模板。

# 19. 大型任务采用 Parent / Child Spec

V2 不引入独立的：

- Master Plan
- Execution Plan
- Stage Plan
- Execution Brief

而是让 Spec 本身支持自然分解。

形式：

```
Standalone Spec
```

或者：

```
Parent Spec
├── Child Spec
├── Child Spec
└── Child Spec
```

# 20. Parent Spec 的职责

Parent Spec 保存整个大型目标共享的信息。

包括：

- 整体目标；
- 整体范围；
- 全局非目标；
- 全局约束；
- 经过开发者确认的关键方向；
- 阶段拆分；
- 必要的阶段依赖关系。

Parent Spec 不应该成为 Master Plan。

不保存：

- 文件级修改步骤；
- 详细 TODO；
- 每阶段完整执行顺序；
- 进度百分比；
- 工时；
- ownership；
- 实施日志。

# 21. Child Spec 的职责

Child Spec 表示：

> 一个可以独立完成和验证的局部任务契约。

例如：

```
Parent: Windows command runtime

Child 1:
Executable resolution

Child 2:
Bundled Bash integration

Child 3:
Agent command integration

Child 4:
Regression and compatibility
```

每个 Child 仍然使用普通 Spec 结构。

# 22. Parent / Child 示例

目录可以是：

```
spec-lite/specs/windows-command-runtime/
├── spec.md
├── command-resolution.md
├── bundled-bash.md
├── agent-integration.md
└── regression.md
```

Parent：

```
# Windows Command Runtime

Status: Active

## Goal

建立稳定、统一的 Windows command execution 基础。

## Constraints

- 不改变 Linux/macOS 行为。
- 不引入第二套 runtime。
- 保留 PowerShell fallback。

## Stages

- [Command resolution](command-resolution.md)
- [Bundled Bash](bundled-bash.md)
- [Agent integration](agent-integration.md)
- [Regression](regression.md)
```

Child：

```
# Bundled Bash

Status: Active

Parent: ./spec.md

## Goal

在 Windows command execution 中支持 bundled Bash。

## Scope

...

## Non-goals

...

## Constraints

...

## Acceptance Criteria

- [ ] ...
```

# 23. Child 继承 Parent 上下文

原则：

> **Child specs inherit parent intent and constraints unless explicitly narrowed.**

因此：

Parent 已经说明：

```
Do not change Linux/macOS behavior.
```

每个 Child 不需要重复写一遍。

只有当前阶段存在额外限制时，Child 才增加本地 Constraint。

# 24. 不把 Parent 变成完整计划

这是 V2 必须明确守住的边界。

Parent 描述：

> decomposition。

而不是：

> execution script。

正确：

```
阶段 1：统一 executable resolution
阶段 2：接入 bundled Bash
阶段 3：接入 agent runtime
```

不正确：

```
阶段 1：
1. 打开 foo.ts
2. 修改 resolvePath()
3. 创建 helper.ts
4. 改 import
5. 跑 xxx test
...
```

微观执行计划仍由当前 Agent 根据当前代码动态生成。

# 25. 为什么 Parent / Child 能解决长规划问题

开发者可以在一个长对话中：

```
探索方案
→ 讨论架构
→ 审核总体计划
→ 明确阶段拆分
```

然后将真正需要跨上下文保存的内容蒸馏成：

```
Parent Spec
+
Child Specs
+
必要 Decision / Facts
```

之后可以结束原讨论上下文。

# 26. 实施阶段使用干净上下文

正式执行某个 Child 时，推荐加载：

```
root AGENTS.md
+
Parent Spec
+
Current Child Spec
+
Relevant FACTS
+
Relevant Decisions
+
Current Code
```

而不是加载：

```
所有规划讨论
+
所有 sibling Child Specs
+
所有历史 Stage
```

原则：

> **Load the parent and the active child, not every sibling.**

# 27. 前一阶段的信息如何进入下一阶段

默认情况下，下一阶段不需要读取前一阶段完整 Spec 或讨论。

前一阶段的信息分别由不同事实源承担。

## 实现结果

看当前代码。

## 修改历史

看 Git。

## 长期设计理由

看 Decision 或 Parent。

## 环境和现实经验

看 FACTS。

## 当前大型任务整体方向

看 Parent Spec。

因此：

> Previous stages are context only when their outcomes cannot be recovered from current reality.

# 28. 阶段完成后的行为

Child 完成时：

```
Status: Completed
```

追加：

```
Outcome
Verification
```

然后进行轻量知识审视：

- 是否产生新的项目级 Fact？
- 是否产生新的长期 Decision？
- 是否修改 PROJECT？
- 是否产生新的 AGENTS 规则？

不机械更新任何文件。

# 29. 下一阶段如何开始

例如：

```
command-resolution
```

完成后进入：

```
bundled-bash
```

推荐新建干净执行上下文。

加载：

```
Parent
+
bundled-bash Child
+
相关 FACTS
+
当前代码
```

Agent 根据当前代码重新制定该阶段的微观执行计划。

不继承上一阶段的完整思考历史。

# 32. Context-reset Development

V2 推荐形成一种工作范式：

```
Think deeply
    ↓
Distill
    ↓
Reset context
    ↓
Execute
    ↓
Verify
    ↓
Distill durable knowledge
    ↓
Reset when useful
```

这不是强制流程。

Spec Lite 只是支持这种方式，而不是要求每个任务都如此执行。

# 34. Init 与 Task 解耦

V2 明确：

```
init != prerequisite for task
```

即使不存在：

```
spec-lite/PROJECT.md
```

也允许：

```
$spec-lite task ...
```

Agent 使用：

- README；
- root `AGENTS.md`；
- manifests；
- 当前任务相关代码；

建立最小必要上下文。

不得因为缺少 `PROJECT.md` 自动执行完整 init。

# 35. Spec 目录策略

默认仍然：

```
spec-lite/specs/<name>.md
```

大型 Parent / Child 可以使用：

```
spec-lite/specs/<topic>/
```

但不根据 Spec 数量自动创建目录。

原则：

> Flat by default; use a topic directory when the work itself forms a real parent/child decomposition or when the repository already follows that convention.

# 36. 稳定语义锚点

所有长期文档优先使用：

- identifiers；
- function / class / module names；
- config keys；
- test names；
- regex；
- commands；
- stable paths。

避免依赖：

```
file.ts:2742
```

这种位置型快照。

原则：

> **Prefer durable semantic anchors over positional snapshots.**

# 37. Active Spec 不建立状态系统

继续使用：

```
Status: Active
Status: Completed
```

不建立：

- Spec index；
- lifecycle service；
- active-task registry；
- status database；
- archive system。

Agent 可以使用自身搜索能力发现 Active Spec。

# 38. V2 推荐目录模型

```
project/
├── AGENTS.md
│
├── spec-lite/
│   ├── PROJECT.md              # optional
│   ├── FACTS.md                # optional
│   │
│   ├── specs/
│   │   ├── simple-task.md
│   │   │
│   │   └── large-task/
│   │       ├── spec.md
│   │       ├── stage-a.md
│   │       ├── stage-b.md
│   │       └── stage-c.md
│   │
│   └── decisions/
│       └── important-choice.md
│
└── code...
```

所有文件均按实际需要产生。

不是每个项目都必须拥有完整结构。

# 39. V2 信息职责总表

```
Code / tests / config
→ What is true now?

Git
→ What changed?

PROJECT
→ Where is everything?

AGENTS
→ How must work be done?

Task Spec
→ What are we trying to achieve?

Parent Spec
→ What is the shared goal and decomposition?

Child Spec
→ What must this stage achieve?

FACTS
→ What has reality already taught us?

Decision
→ Why was an important lasting choice made?

Conversation
→ Temporary reasoning workspace
```

# 40. Spec Lite 不保存什么

V2 明确不保存：

- 完整思维过程；
- 对话历史；
- 普通模型 Plan；
- 文件级实施清单；
- 动态 TODO；
- 开发进度百分比；
- 工时；
- owner；
- 自动生成的任务状态；
- 可以从代码直接恢复的信息；
- 普通 Debug 日志。

# 41. V2 明确不引入的系统能力

为了防止逐步演化成重型 Spec Framework，V2 明确排除：

- 自动 freshness service；
- Facts TTL；
- 定时 Fact 重验证；
- Fact 数据库；
- Fact ID；
- 全仓库知识索引；
- 自动同步器；
- 多 Spec 冲突检测；
- 自动项目 dependency graph；
- Spec workflow engine；
- 自动 stage scheduler；
- plan executor；
- progress tracking；
- completion dashboard；
- 强制 init；
- 强制 Parent Spec；
- 强制 Child Spec。

# 42. SKILL.md 修改计划

V2 的运行规则应继续保持短小。

`SKILL.md` 只加入必要行为，不复制本设计文档全部解释。

预计修改如下。

## Route

补充：

- `task` 不依赖 `init`。
- continuing a child spec 时加载 Parent + active Child，而不是所有 sibling。
- Parent/Child 仅用于自然可独立验证的阶段。

## Task Spec

补充：

- 支持 standalone 和 parent/child 两种形态。
- Parent 保存共享目标、约束和阶段索引。
- Child 保存局部契约。
- Child 继承 Parent 上下文。
- 不把详细 Plan 写入 Spec。

## Verified Facts

加入：

- Task-local Verified Facts 为可选区块。
- 跨任务高价值事实进入 `FACTS.md`。
- Fact 需要满足三项准入标准。

## Reuse / Revalidation

加入：

- 调查环境、测试和 Runtime 异常前，优先检查相关 Fact。
- observation mismatch / causal change / explicit challenge 时重新验证。
- 不通过时间自动刷新。

## Stable Anchors

加入：

- durable semantic anchors 优于 file。

## Finish

补充：

- 完成 Child 时正常记录 Outcome / Verification。
- 轻量审视是否产生新的 Fact / Decision / PROJECT / AGENTS 知识。
- 不保存详细执行过程。

# 43. TASK_SPEC.template.md

保持默认模板尽量不变。

仍然：

```
# <Feature Name>

Status: Active

## Goal

...

## Scope

...

## Non-goals

...

## Constraints

...

## Acceptance Criteria

- [ ] ...
```

不默认增加：

```
Verified Facts
Parent
Stages
```

这些只在任务真正需要时动态增加。

目的：

> 普通任务不应为大型任务能力支付模板复杂度。

# 44. 新增 FACTS.template.md

建议新增：

```
# Project Facts

<!--
Record only empirically verified project knowledge that:
- cannot be easily rediscovered from code,
- is meaningfully expensive to rediscover,
- is likely to affect future work.

Keep only facts that are still useful now.
Git preserves history.
-->

## <Fact topic>

Observed:

<What reality has established.>

Recognize by:

<Stable signatures or semantic anchors when useful.>

Verified:

<When and under what relevant environment this was established.>

Revalidate when:

<Events that make this fact reasonably questionable.>
```

# 45. 中文副本

新增：

```
Spec Lite v0.2 中文副本.md
```

完整说明：

- Facts；
- Parent / Child；
- 上下文切换；
- 新对话 vs fork；
- context-reset development；
- 信息生命周期；
- 不做什么。

V1 中文副本保留作为历史版本参考。

# 46. V2 推荐实践示例

大型任务：

```
Windows Command Runtime
```

经过多轮讨论后：

```
Parent Spec:
Windows Command Runtime

Children:
1. Command Resolution
2. Bundled Bash
3. Agent Integration
4. Regression
```

正式进入 Stage 1 时，新上下文读取：

```
AGENTS
Parent
Command Resolution Child
Relevant FACTS
Relevant current code
```

Agent 自己制定当前微观 Plan。

Stage 1 完成后：

```
Child → Completed
Outcome / Verification
必要 Fact → FACTS
必要 architecture rationale → Decision
```

进入 Stage 2：

```
新上下文
AGENTS
Parent
Bundled Bash Child
Relevant FACTS
Current code
```

不带 Stage 1 长讨论。

# 47. 判断 V2 是否成功的标准

V2 不以“功能更多”为成功。

而应观察：

## 上下文效率

同一项目反复工作时：

- Agent 是否少重复调查已知问题？
- 是否减少无价值测试分析？
- 新阶段是否能在更干净上下文启动？

## 计划稳定性

经过开发者审核的大方向：

- 是否能够跨上下文保留？
- Agent 是否仍然能根据当前代码动态调整微观实现？

## 框架重量

普通任务是否仍然：

```
一个简单 Task Spec
```

而不是被迫进入：

```
init
→ proposal
→ plan
→ stages
→ tasks
→ facts
→ verification
→ archive
```

如果普通任务开始需要复杂生命周期，说明 V2 设计失败。

# 48. 最终设计原则

Spec Lite V2 最终可以浓缩成以下几句话。

> **Code is reality.**

> **Specs preserve intent and contracts.**

> **Facts preserve expensive lessons from reality.**

> **Decisions preserve lasting rationale.**

> **Parent specs preserve shared direction; child specs preserve local stage contracts.**

> **Plans remain dynamic.**

> **Preserve decisions, not conversations.**

> **Preserve commitments, not reasoning history.**

> **Preserve contracts, not execution scripts.**

> **Load only the context required by the current stage.**

> **Reset context at natural phase boundaries.**

最终，Spec Lite 的目标不是让 Agent 记住更多。

而是：

> **让 Agent 在每一次新的上下文中，只携带那些如果丢失就会真正降低决策质量的信息。**