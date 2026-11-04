# Matt SDLC × Herdr app：layout 与 mods 系统设计

v0.3 索引 · 2026-11-04 · 提议与可交互原型，尚未完成真实接入。

**当前实施方向以 [统一工作区基线](./2026-11-04-unified-workspace-baseline.md) 为准。** 该基线替代下文早期的只读范围与分散布局，明确受控执行、操作桥和恢复契约。下文 v0.2 内容保留为 Matt SDLC、平台事实和设计推导依据；与新基线冲突时采用新基线。

主原型：[E · 简洁工作区](https://github.com/zengjie/herdr-matt-workbench/blob/prototype/unified-workspace/docs/design/matt-workbench-prototype.html?variant=E)。A–D 仅保留为历史比较。

用户已确认：mods 是 Herdr app 自身的 mods。本文替代基于旧 Herdr TUI 和 Claude Code mod 的 v0.1；旧稿保留在同目录 `.v0.1.md`。

## 1. 结论与范围

建议做一个稳定的个人 layout `matt-workbench`，复用 app 原生 mods，加一个 `matt-flow` mod，以 props.role 呈现阶段条、任务图、工件和交接建议。

首版让人看到当前工作、可开始的工作、依据和下一步。Matt skills 继续拥有执行与编排职责，Herdr 提供展示、定位和人工操作入口。

本轮只交付设计。首版建议只读本机 tracker，不自动认领或关闭 ticket、不复制 implement-spec 调度器、不依赖 Claude Code 扩展、不改 Matt skills。暂不做多机编排、GitHub tracker、自动 token 遥测。当前仓库只有设计文档，没有应用代码。

## 2. Matt 倡导的 SDLC

### 主流程与目的

```text
想法 → grill-with-docs → 必要时 prototype 往返 → 判断工作规模
  单 session → implement：TDD → 双轴 review → commit
  多 session → to-spec → to-tickets
    → 每 ticket 新上下文 implement
    或 implement-spec：fresh workers/worktrees → merger → integration review
  → 按需 PR → retro → 改善下一轮 agent 工作环境
```

它把不同认知工作放进适合的上下文。先共同确定问题与领域语言，再保存决策，然后按行为切片实现与验证，最后改进 agent 环境。它不是每个任务必须走完的线性流水线。

| 工作 | 核心要求 | 完成依据 |
|---|---|---|
| grill-with-docs | 每轮询问整个决策 frontier；事实由 agent 查，决策由人作；保留术语和必要 ADR | 决策收敛，用户确认共同理解 |
| prototype | 回答一个逻辑或视觉问题，独立上下文，handoff 往返 | 原问题、结果、保留的 prototype 分支和来源指针 |
| to-spec | 综合已有讨论，不重做产品访谈；确认测试 seam | 已确认的 spec，发布到配置的 tracker |
| to-tickets | tracer-bullet 垂直行为切片，声明 blocking edges | 用户认可粒度与依赖，tickets 已发布 |
| implement | 一次一个 red → green 切片；review；commit | 验收证据、review 结果与提交 |
| implement-spec | 每 ticket 新上下文、独立 worktree/branch；merger 集成；最后整条分支 review | integration branch 完成，tracker 按自身约定收尾 |
| code-review | 固定比较范围，Standards 与 Spec 独立审阅 | 分别报告两个维度；缺 spec 时显式标明 |
| retro | 改环境：导航、检查、规范、steering、工具 | 机械错误进入确定性检查，判断问题进入 review 规范 |

TDD 测公共 interface 上的行为，不测内部实现；一次一个 red-green 切片，避免先批量写测试再批量实现。测试 seam 要事先与用户确认。

### 入口与旁路

- triage 只处理外来的原始 issue；to-tickets 输出已经 agent-ready，不重复 triage。
- diagnosing-bugs 先实际运行一条能捕捉当前症状的 red-capable 命令，再最小化复现、比较假设、修复、回归与清理；随后同 session retro。
- wayfinder 面向单 session 装不下的模糊工作，产出决策地图。区分 HITL 和 AFK；一般每 session 解决一张 decision ticket，research 有例外。路线清晰后进入 to-spec，不能直接把决策图当实现图。
- improve-codebase-architecture 找 deepening 候选，再回到主流程。codebase-design、domain-modeling 是贯穿流程的词汇层，调用它们不能覆盖顶层工作阶段。

### 上下文卫生

grill → spec → tickets 尽量保留一个不中断上下文；接近 smart zone 时允许在最近阶段交接处 compact。约 150k 是 Matt 文档的经验值，不是所有模型的容量或质量保证。

PHASE-BOUNDARIES 的问题按顺序判断，首个 yes 优先：

1. 能继续吗？下一工作需要当前原始推理，或有足够余量，优先 Continue。
2. 当前上下文与下一工作无关吗？是则 clear。
3. 换 harness、换目录、交给同事或分出旁路任务吗？是则 handoff。
4. 下一工作明确且可以 AFK 吗？是则 subagent。
5. 否则 compact，指明下一工作需要什么。

这些是人的判断。原稿把第一条的 OR 写成了 AND，已修正；smart zone 例外仍需结合实际容量判断。交接只在一段工作结束时发生，不能用 turn.complete 猜测。

retro 在 clear 前完成；若已 clear，需指向原 session 日志。handoff 和 subagent 报告不能代替其原始证据指针。

## 3. 平台事实与旧稿纠偏

本机核验：`herdr-app 0.1.12`，`hc 0.1.17`。旧命令 `herdr 0.9.5` 也存在，属于不同接口。

| 能力 | app 当前事实 | 设计约束 |
|---|---|---|
| layout | layouts/name.json；block、row、column、stack | 不使用旧 TUI 的 layout.apply/split 树 |
| mod | main.js 导出 View；gpui-kit 原生 UI | 无 DOM、HTML、Node API、setTimeout |
| live data | spaces、agents、terminals、panes、focused；变化自动重绘 | 不轮询已有 live data |
| actions | run、layout.show/select/resize、tab.new、agent.focus、terminal.peek | 呈现与真实工作结构分开 |
| core | hc 运行终端；有 parent 与 labels | 不能保证看到 harness 内部所有 subagent |
| 工件 | readTextFile、watchFolder、exec | 本机限目录 watch；失败后低频刷新 |
| shared | 跨 mod 持久化，last-write-wins | 保存 UI 绑定，不当任务锁或事务库 |
| 校验 | herdr-app check、shot | check 只覆盖初始渲染，仍须 app 实测 |

已读取运行状态：当前 main layout；Claude Code 与 Codex 终端均存在；存在离线机器错误。设计必须区分未知、失联、陈旧与成功。

本地已有 mission-control、herdr-t3、revenant-main layouts 和证据 mods，可借鉴原生 UI 模式。本设计不覆盖它们，也不依赖它们的私有领域模型。

## 4. 领域模型

- Effort：围绕一个目的组织的工作集合。不等于 space；一个 repo 可有多个 effort，一个 effort 可关联多个 worktree/space。
- Work Item：分 Implementation Ticket 与 Decision Ticket；完成语义不能混用。
- Session：agent 的一次上下文。Terminal 是载体；clear 后 terminal address 可以不变，但 Session 已变。
- Phase：一段认知工作，可有子工作。implement 内 review 后回到 implement，不能只存全局 phase 字符串。
- Phase Boundary：当前工作完成、下一工作开始前的交接点。
- Stage Profile：突出相关信息的工作视图；切换它不表示阶段完成。
- Frontier：开放、未认领、全部 blockers 已完成的工作项集合。
- Context Pointer：来源位置；验收与 review 还要固定 commit/revision。

建议身份：machine + canonical repo root + effort directory；work item 用 tracker 路径或原生 id；terminal/tab/space 用 app 明确 id。worktree 关系由 git 和显式绑定确认，不靠 space 名或模糊 cwd 前缀。

首版由人绑定 effort、tab/terminal 和当前工作。恢复时仅保留上次选择并要求核对；无法识别 session 变更时，不恢复上下文安全结论或完成结论。

## 5. Layout：稳定工作台

```text
┌────────────────┬─────────────────────────────────────────────────┐
│ 原生 spaces    │ 原生 tabs                                       │
│                ├─────────────────────────────────────────────────┤
│ 原生 agents    │ Matt 阶段条：effort / 工作 / 子工作 / 交接入口   │
│                ├───────────────────────────┬─────────────────────┤
│ blocked 优先   │ content: {tab: active}    │ 右侧 stack          │
│ 点击定位终端   │                           │ Frontier / 工件     │
│                │ 保留真实 terminal/split   │ 证据 / Files        │
│                │                           │ Changes             │
├────────────────┴───────────────────────────┴─────────────────────┤
│ 原生状态栏；流程信息标注来源、更新时间和未知项                    │
└──────────────────────────────────────────────────────────────────┘
```

初始尺寸提议：左侧沿用 main 的 268px，阶段条 40px，右侧 360px 可拖动和隐藏；中央占剩余空间。窄窗口收起右侧，任务详情用 dialog。比例待 P0 验证。

使用一个 layout 的 stack 和 show/hide 改变阅读重点，不为每个 phase 重建布局。layout.switch 会重新启动 layout 所属 mod，并关闭浮窗；流程变化不应频繁触发它。改变工作视图不能重建终端、结束进程或 clear。

| 工作视图 | 信息重点 | 操作 |
|---|---|---|
| Shape：grill/spec/tickets | 问题、GLOSSARY、ADR、spec、ticket 粒度 | 打开来源，手动标注当前工作 |
| Explore：prototype/research/wayfinder | 当前问题、地图、原始证据、返回主线指针 | 定位独立 session，区分 HITL/AFK |
| Build | frontier、blockers、当前 ticket、关联终端和 branch | 打开 ticket、定位 agent、显式新 session |
| Review | 固定 base/head、Standards/Spec 两栏、Changes | 打开 diff 和报告，缺项保持未知 |
| Diagnose | 症状、复现命令、最近 verdict 与输出 | 定位反馈终端，无 red 证据时提示先复现 |
| Retro | 来源 session/log、失败模式、环境改进位置 | 组织建议，不自动写规范 |

首版先做 Shape → Build 闭环。其余视图先保留为设计需求，避免空壳按钮。

## 6. Mod 与模块

新增一个 matt-flow mod，props.role 为 rail、board、artifact、boundary。它们是同一个流程能力在不同位置的呈现，静态模块在同目录共享。继续复用原生 spaces、agents、tabs、files、changes、pane、workspace-status、worktree。

```text
layouts/matt-workbench.json
mods/matt-flow/
  main.js       View 与各角色 UI
  mod.json      place: none，由 layout 放置
  model.js      状态到显示模型；工作阶段与建议
  tracker.js    Markdown 解析、图校验、frontier
  actions.js    绑定、读取、定位、打开、显式启动
RECIPE.md       实现后按真实行为整理
preview.png     实现后 app shot 获取
```

不提前引入 TS 编译、vendor 复制、通用 Tracker 框架或多适配器系统。以后确需独立反馈循环能力时再拆 mod。

数据流：工件/tracker + Herdr live data + 人工选择 → 显示模型 → 各角色 UI；用户动作 → app commands/打开来源/新 tab 初始任务。

事实分层：tracker 决定依赖和任务状态；git 与证据决定代码版本；Herdr 决定运行和注意力；shared 保存 UI 偏好和人工标注。不能拿一层的绿色补另一层的未知。

shared 使用 matt-flow 命名空间，按 effort/tab 保存绑定。board 读取选中本机 effort，发布带 revision/time/error 的缓存；rail 读缓存与 live data，artifact 只读选中文件，boundary 打开时才生成建议。缓存不写回 tracker。

异步读加 generation 检查，切 effort 后丢弃旧响应；同实例不重叠刷新；watch 只看 effort 目录，失败时 10 秒对账并提供刷新。隐藏实例仍可能运行，不能用高频定时器刷 UI。重启/热重载从来源重建；多窗口允许只读重复刷新，首版不引入 leader 协议。

## 7. Tracker 与状态

to-tickets 的真实模板有 `**Blocked by:**`、`**Status:** ready-for-agent`；wayfinder 的本地约定有 Type、Status、Blocked by。兼容两种已有字段写法并保留原文，不重写技能输出。

- Decision Ticket：claimed 为认领，resolved 为解决，用于 decision graph 的依赖完成判断。
- Implementation Ticket：ready-for-agent 是 triage 状态，不是验收完成。默认模板未统一规定实施完成/认领字段，需读取具体项目 tracker 约定。
- 约定未明确时显示原始状态及“依赖完成状态未知”；不能擅自把 done/closed/resolved 都映射为通过。
- 缺失或重复 blocker、模糊编号、环、冲突字段、未知状态都标为问题，相关下游不可判 ready。读取错误保留旧值并标陈旧，不显示空列表成功态。

首版不写 claim/status。新 session 前重新读取 ticket/blockers；已有绑定 live terminal 则优先定位。没有原子认领协议时不承诺独占执行，不做并行自动启动。

三套状态分开：

| 维度 | 示例 | 证明什么 |
|---|---|---|
| 运行/注意力 | working、blocked、done、idle、失联 | agent 是否需要人注意 |
| 工作 | tracker 原始状态、依赖、认领 | 工作是否可开始或已解决 |
| 证据 | 缺失、失败、通过、过期 | 某 revision 的验证结果 |

Herdr done 表示未被看到的完成通知，看过后变 idle；它不能关闭 ticket、触发 merge 或把验收转绿。review 必须绑定 base/head；head 改变则报告过期。测试通过不代表 Spec review 通过。

## 8. 阶段与交接顾问

本机 app live types 未提供跨 harness 的 skill.prompt 或当前上下文 token 接口。首版手动选择阶段并标注来源；工件变化只提示候选，不自动推进。不能承诺拦截 clear/compact。

模型区分顶层工作、子工作、交接状态；例如 implement → 子工作 review → 返回 implement。domain-modeling 不覆盖顶层阶段。

用户主动打开交接框并声明当前工作完成后，按第 2 节顺序判断。展示下一工作、需保留来源、依据和未知项；只给建议与可复制文本，不向已有终端自动发 clear/compact。

上下文缺失显示“未知：当前 harness 未提供”，不能显示 0 或推测余额。以后有遥测仍需区分当前窗口占用、累计计费 token、模型容量、来源时间。约 150k 只作带出处的经验提示。

## 9. 操作契约与场景

- 绑定：明确 repo/effort/tab/terminal，不根据 space 名静默猜测。
- 打开工件：artifact 只读展示来源；不存在、不可读、超限分别报告。
- 定位：显式 terminal id 调用 agent.focus/peek；退出后显示未连接。
- 新 session：复核 tracker，选择 installed agent，使用 tab.new 的 argv/folder/task；原 session 保留。未知 harness 的 skill 调用方式只提供任务文本，不拼造命令。
- 切面板：layout.show/select/resize，不创建或关闭终端。
- 初始任务主要传 spec/ticket/证据指针，不复制历史全文；HITL 不默认 AFK。

必须走查：

1. spec 文件出现只提示新工件；不认定讨论结束或 ticket 已自包含。
2. implement 内 review 后回到 implement；agent idle 不修改 tracker。
3. A、B 并行，C 依赖两者：A 终端 done 不放行 C，完成须来自 tracker/集成约定。
4. wayfinder resolved 决策进入地图，不能算功能交付；清晰后进入 to-spec。
5. 诊断命令退出 0，但未证明捕捉原症状时，仍缺有效复现证据。
6. 切 effort 时旧响应丢弃；焦点变化不迁移绑定；标题显示来源 effort。
7. 重启/热重载/离线不恢复完成结论，不自动重启 worker。
8. 切回 main 不 clear session、不删 worktree；显示层不拥有 worker 生命周期。

## 10. 实施切片与验收

以下是候选切片，未发布为 tickets。

| 切片 | 可交付行为 | 验收 |
|---|---|---|
| P0 原生可行性 | 最小个人 layout/mod，实际 focus、手动阶段、一份真实 ticket | check 通过，app shot/点击验证；切布局保留 terminal address 和进程 |
| P1 Shape | 绑定 effort、查看术语/决策/spec | 两个 effort 不串数据；空错加载态正确 |
| P2 Build | 本地 tickets、依赖、已知 frontier、定位/显式新 session | 粗体格式、未知完成、环/缺失 blocker；done→idle 不改 ticket |
| P3 交接 | 用户发起，按顺序给建议 | Continue 优先；上下文未知不造数；不主动 clear |
| P4 证据 | 双轴 review、诊断、retro 来源 | 固定 revision；变化使证据过期；来源可打开 |

建议最高测试 seam：真实 tracker 文本 + app snapshot + 用户动作 → 显示模型和动作请求；另以真实 app 测布局、焦点、生命周期。正式 TDD 前仍需与用户确认 seam。

每切片除 check 外，走查窄窗口、长中文、异步错误、键盘焦点、多实例、热重载；shot 生成真实 preview。check 不能证明延迟渲染和后续状态可用。

若未来由 Herdr 调度 worker，需另定原子认领、integration base、串行 merge、冲突恢复、验证重跑、崩溃恢复。不能与 implement-spec 同时拥有同一任务。

## 11. 待讨论的选择

已确认：目标是 Herdr app mods。以下是建议，不视为批准：

- 首版接受稳定工作台、手动阶段、只读 tracker？推荐接受。
- 优先 Shape 还是 Build？推荐 Shape → Build，先验证工件与上下文连续性。
- P2 选哪个真实 effort？其实施完成/认领约定是什么？这是可靠 frontier 的前置事实。

下一步用 P0 验证原生交互；确认设计后进入 to-spec 并确认 seam，再由 to-tickets 让用户核对粒度和依赖。无需提前决定所有未来配置与自动编排。

## 12. 来源和核验边界

Matt 本机原文：`~/.agents/skills/ask-matt/SKILL.md`、其 `PHASE-BOUNDARIES.md`；grilling、grill-with-docs、to-spec、to-tickets、implement、implement-spec、tdd、code-review、wayfinder、diagnosing-bugs、retro、prototype、handoff、codebase-design、domain-modeling 的 SKILL.md；setup-matt-pocock-skills 的 issue-tracker-local.md、triage-labels.md。

平台：`~/.config/herdr-app/AGENTS.md`、herdr.d.ts、layouts/main.json、layouts/mission-control.json、mods/mission-control/{main,model}.js、mods/worktree/main.js。原生示例仅用于理解模式，不代表其行为全部适合复制。

已运行只读核验：herdr-app/hc --version，herdr-app status、ls、commands，hc ls --json。官方形态交叉核对：[herdr-app](https://github.com/herdrdev/herdr-app)、[mods](https://herdr.dev/mods/)。具体接口以本机类型为准。

本轮未实现 mod，未运行新 mod 的 check 或布局交互测试，未修改运行中的 Herdr 配置。设计行为仍需 P0 验证。
