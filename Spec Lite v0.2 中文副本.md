# Spec Lite v0.2 中文副本

> V2 的中文阅读与使用说明，不参与 Skill 加载。运行入口为根目录 `SKILL.md`，模板以 `assets/` 为准。`Spec Lite v0.1 中文副本.md` 保留为历史参考。本版依据 `Spec Lite V2 完整架构与修改计划.md` 实现。

## 定位与原则

Spec Lite 是一套面向 coding agents 的轻量上下文蒸馏协议：在会话、阶段和 Agent 切换时，保留丢失后会降低决策质量的信息，同时允许 Agent 根据当前代码重新推理。

它不控制模型如何思考、编码或生成实施计划。默认相信模型能理解当前实现、动态制定步骤，并根据新信息调整。

- 保存结论，不保存讨论。
- 保存承诺，不保存思考轨迹。
- 保存契约，不保存执行脚本。
- 在自然阶段边界，按需重建干净上下文。

是否使用 Spec Lite 由开发者决定。普通任务不因规模、关键词或涉及文件数而自动启用。生成内容保持简洁，使用用户或项目惯用的语言。

## 信息放在哪里

| 信息源 | 回答的问题 | 应保存的内容 |
| --- | --- | --- |
| 代码、测试、真实配置 | 现在实际是什么？ | 当前实现与可执行行为 |
| Git | 改了什么？ | 修改历史 |
| PROJECT | 东西在哪里？ | 项目目的、技术栈、模块、入口、重要流程 |
| 根 AGENTS | 必须如何工作？ | 长期规则与稳定工程约束 |
| 普通 Task Spec | 要达成什么？ | 目标、范围、非目标、约束、验收条件 |
| Parent Spec | 共享方向和拆分是什么？ | 整体契约、已确认方向、阶段及必要依赖 |
| Child Spec | 当前阶段要达成什么？ | 可独立完成和验证的局部契约 |
| FACTS | 现实已经教会了什么？ | 昂贵且可复用的实证经验 |
| Decision | 为什么这样选择？ | 重要、长期有效的设计理由 |
| 对话 | 当前如何探索？ | 临时推理工作区，不是持久知识库 |

`PROJECT.md` 不存放任务细节、测试基线、调试结论或实施计划。`AGENTS.md` 不存放临时 workaround、单次失败或当前任务计划。普通实现选择无需生成 Decision。

## 显式请求与路由

- `init`：建立最小项目地图。
- `task`：创建或维护任务契约，无需先 init。
- `finish`：确认相关 Spec 的实际完成情况，并轻量审视长期知识。
- 显式调用 `$spec-lite` 并使用自然语言描述需求时，按意图选择相应行为。

继续某个 Spec 时，读取存在的根 `AGENTS.md`、对应契约、相关事实和决策，以及当前相关代码。存在 PROJECT 时可用它导航。继续 Child 时加载 Parent 和当前 Child，不默认加载全部 sibling、前一阶段讨论或所有历史 Spec。

缺少 `spec-lite/PROJECT.md` 不是错误，也不触发完整 init。使用 README、根 AGENTS、manifests 和相关代码建立最小上下文即可。

所有 Spec Lite 维护的项目文档放在根目录专用的 `spec-lite/` 下；根 `AGENTS.md` 仍位于项目根目录。

## 初始化项目

渐进式读取根目录结构、README、根 AGENTS、项目清单、构建和测试配置，定位实际存在的主应用、Runtime、API、数据层等入口，再读取少量代表性文件验证判断。

能说明项目目的、模块、入口、重要流程及开发和测试方式后停止扫描。不得为了完整性无目的地扫描全仓库。

使用 `assets/PROJECT.template.md` 创建或更新可选的 `spec-lite/PROJECT.md`。它是低粒度导航，不是代码副本。不得反向重建历史 Decision，也不预先创建空 FACTS 或 Decision。

创建或最小补充根 AGENTS 时，保留已有内容，只添加有项目依据、未来必须遵守的稳定规则；不得覆盖原文件或在此流程中创建嵌套 AGENTS。

## 普通 Task Spec

默认使用 `assets/TASK_SPEC.template.md` 创建平铺文件：

```text
spec-lite/specs/<有意义的-kebab-case-名称>.md
```

默认模板保持简单：

```markdown
# <功能名称>

Status: Active

## Goal

<为什么需要这项工作。>

## Scope

<这次需要解决什么。>

## Non-goals

<明确不解决什么。>

## Constraints

<技术、兼容性或产品边界。>

## Acceptance Criteria

- [ ] <可观察、可确认的完成条件。>
```

默认不增加 `Parent`、`Stages` 或 `Verified Facts`。普通任务只需要一个简单 Spec。

内容来自开发者要求和已确认事实。只有缺失选择会实质改变契约时才提问。同一目标内的需求变化更新当前 Spec；独立目标才创建新 Spec，不使用版本后缀或复杂编号。

实施计划根据当前代码动态制定，不同步进 Spec。文件级步骤、动态 TODO、进度百分比、工时、owner、优先级、时间线、Proposal、Change Log 和调试日志不进入契约。

## 大型任务的 Parent / Child

只有工作自然形成可独立验证的阶段，才使用 Parent / Child。默认平铺；真正分解或仓库已有约定时，才使用专题目录，不因 Spec 数量增多自动建目录。

```text
spec-lite/specs/windows-command-runtime/
├── spec.md
├── command-resolution.md
├── bundled-bash.md
├── agent-integration.md
└── regression.md
```

Parent 使用普通契约表达整体目标、范围、非目标、全局约束、开发者已确认的方向和整体验收标准，再按需增加 `Stages`。例如：

```markdown
## Constraints

- 不改变 Linux/macOS 行为。
- 不引入第二套 Runtime。
- 保留 PowerShell fallback。

## Stages

- [Command resolution](command-resolution.md)：统一 executable resolution。
- [Bundled Bash](bundled-bash.md)：在 resolution 契约成立后接入 Bash。
- [Agent integration](agent-integration.md)：使用已建立的 command boundary。
- [Regression](regression.md)：验证兼容性与整体契约。
```

这些是阶段目标及必要依赖，不是必须逐行执行的 Master Plan。Parent 不保存文件级修改步骤、完整执行顺序或实施日志。

Child 继续使用普通 Task 模板，在 `Status` 下按需增加：

```markdown
Parent: ./spec.md
```

路径相对当前 Child。Child 继承 Parent 的意图和约束，只补充本地范围与额外边界。允许显式缩小范围，不能静默放宽共享约束。每个 Child 都应有可独立验证的验收条件。

## Facts 的准入与归属

Fact 是实际运行、调查或实验确认的现实知识，必须同时满足：

1. 无法轻易从代码、测试定义或配置中恢复。
2. 重新发现有明显成本。
3. 未来任务很可能再次遇到。

典型候选包括已核实的测试失败基线、Runtime 实测行为、企业环境限制、平台差异、外部服务实际返回格式和特定依赖组合问题。

“今天安装依赖较慢”、可直接从 manifest 得到的版本、未经调查的失败和临时假设不符合要求。不能把无法解释的测试失败直接登记为 baseline。

只影响当前任务的实证信息，按需放入当前 Spec 的 `## Verified Facts`；明显影响多个未来任务的高价值事实，放入可选的 `spec-lite/FACTS.md`。Agent 可在正常工作中主动记录符合标准的事实，无需每条请求确认。

事实描述“观察到了什么”，约束描述“必须遵守什么”，两者不可混用。

## Fact 的最小结构

使用 `assets/FACTS.template.md`。不引入复杂 Schema、Fact ID 或数据库。

```markdown
# Project Facts

## <事实主题>

Observed:

<现实已确认的行为，与假设区分。>

Recognize by:

<稳定的命令、测试名、错误签名、正则或其他语义锚点。>

Verified:

<何时、通过什么方式、在什么相关环境下确认。>

Revalidate when:

<哪些观察变化、因果变化或明确质疑需要重新验证。>
```

例如，某次调查已确认企业 Windows 环境存在 Redis 连接失败基线，便可记录测试名和 `/ECONNREFUSED.*127.0.0.1:6379/` 等识别信息，并写清已核实的环境及验证方式。这个示例不是任何项目中已成立的事实。

## Fact 生命周期

```text
Observe → Record → Reuse → Revalidate → Update / Remove
```

事实由工作学习和失效，不按时间维护。

遇到测试、Runtime、工具链、平台、环境或外部服务异常，在投入大量调查前查看相关 Fact。识别时比较签名与适用环境，不能只比较失败数量。相同数量可能包含不同问题；新增、消失或变化的签名都应调查对应 delta。

重新验证由三类事件触发：

- **Observation mismatch**：当前观察与记录不一致。
- **Causal change**：本次修改了可能影响事实的代码、环境、Runtime、依赖或测试配置。
- **Explicit challenge**：用户或 Agent 明确质疑事实是否仍然成立。

日期记录验证背景，不是自动失效机制。年龄只能提高怀疑，不能单独触发重验证；不采用 TTL、定期扫描或定时重跑。

某组失败中一项已修复，就原地删除那项；事实完全失效或不再有用，就删除该条。不要追加 Deprecated、Superseded、Archived Fact 或 Fact History。Git 保存历史，FACTS 只保留当前有用的知识。

## 上下文切换与新对话

大型工作可采用：

```text
深入探索 → 蒸馏契约和必要知识 → 重置上下文 → 实施
→ 验证 → 蒸馏长期知识 → 按需再次重置
```

这是推荐实践，不是强制流程。探索转为正式实施、一个阶段进入下一个阶段，是自然切换点；不使用固定 Token 比例决定是否切换。

切换前，将开发者审核过的重要方向写入 Parent 或必要 Decision，将昂贵实证经验写入相应 Facts，将当前阶段契约写入 Child。不要把完整讨论复制成执行 Brief、Stage Plan 或文件级操作清单。

进入某个 Child 时，最小上下文通常为：

```text
存在的 root AGENTS
+ Parent
+ 当前 Child
+ 相关 Facts / Decisions
+ 当前相关代码、测试和配置
```

新对话可以只加载这组内容，形成干净执行上下文。携带原对话历史的 fork 可以保留探索分支，但本身不等于上下文重置。是否切换由开发者和当前工作的语义边界决定，Skill 不自动新建对话或强制 fork。

下一阶段默认不读前一阶段完整 Spec 或讨论：实现结果看当前代码，修改看 Git，长期理由看 Decision 或 Parent，现实经验看 Facts，整体方向看 Parent。只有无法从当前现实恢复的结果，才需要额外带入前一阶段信息。

## 懒验证与稳定锚点

PROJECT 存在时先用它导航，再只验证本次相关认知。发现过期内容就修正对应部分，不扫描无关模块寻找漂移。

所有长期文档优先使用 identifiers、函数/类/模块名、config keys、test names、regex、commands 和 stable paths。避免依赖 `file.ts:2742` 这样的行号快照。

Active Spec 可直接搜索：

```text
rg -l 'Status: Active' spec-lite/specs
```

只保留 `Status: Active` 和 `Status: Completed`。不建立 Spec index、active-task registry、状态数据库或归档系统。

## 完成 Spec 与阶段交接

先确认相关 Spec 和实际实现。如果上下文无法区分多个候选，再询问开发者。

完成时将 `Status: Active` 改为 `Status: Completed`，并追加简短的：

- `Outcome`：实际交付及相对原契约的重要差异。
- `Verification`：如何确认完成。

Spec 保留原位，不归档。Child 同样处理；一个 Child 完成不自动完成 Parent，Parent 必须满足整体契约和相关阶段结果。无需为此读完所有 sibling 讨论。

然后轻量审视是否产生新知识：符合准入的 Fact、值得保留理由的 Decision、现有 PROJECT 的导航变化，或未来必须遵守的根 AGENTS 规则。只在确有新知识时更新，不机械创建全套文件。

Decision 继续使用 `assets/DECISION.template.md`，路径为 `spec-lite/decisions/<有意义的-kebab-case-名称>.md`，内容为 Context、Decision、Reason、Consequences。

## 推荐目录与明确边界

```text
project/
├── AGENTS.md
├── spec-lite/
│   ├── PROJECT.md              # 可选导航
│   ├── FACTS.md                # 可选跨任务实证知识
│   ├── specs/
│   │   ├── simple-task.md
│   │   └── large-task/
│   │       ├── spec.md
│   │       ├── stage-a.md
│   │       └── stage-b.md
│   └── decisions/
│       └── important-choice.md
└── code...
```

按实际需要产生文件，不要求每个项目拥有完整目录。V2 不引入自动 freshness service、Facts TTL、定时重验证、知识索引、同步器、多 Spec 冲突检测、自动 dependency graph、workflow engine、stage scheduler、plan executor、progress tracking 或 completion dashboard，也不强制 init、Parent 或 Child。

## 调用配置与 V1 升级

当前 `agents/openai.yaml` 保持显式调用：

```yaml
interface:
  display_name: "Spec Lite"
  short_description: "Distill context, contracts, and verified facts"
  default_prompt: "Use $spec-lite to distill durable context and maintain a lightweight task contract."

policy:
  allow_implicit_invocation: false
```

升级只需使用新的运行入口、配置和 Facts 模板。原有 PROJECT、普通 Task Spec 和 Decision 无需批量迁移。只有实际需要时才补充局部 Verified Facts、跨任务 FACTS 或 Parent / Child，避免一次性为旧任务重建阶段结构。

## 如何判断 V2 有效

观察是否减少已知问题的重复调查、新阶段是否可以用更少的无效历史启动、开发者已确认的方向是否跨上下文保留，以及 Agent 是否仍能从当前代码调整微观实现。

普通任务应继续只需一个简单 Task Spec。如果开始要求 init、proposal、plan、stages、tasks、facts、verification、archive 的固定流水线，就偏离了 V2 的目标。
