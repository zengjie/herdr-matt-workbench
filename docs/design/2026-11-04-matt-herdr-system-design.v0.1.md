# Matt SDLC x Herdr：layout 与 mods 系统设计

状态：草案（v0.1，2026-11-04）。这份文档是 `/grill-with-docs` 的输入，还没到 spec。按 Matt 的主流程，它下一步要先过一轮 grilling，再进 `/to-spec` 和 `/to-tickets`。

## 1. 目标

让 Matt Pocock skills 倡导的 SDLC 在 Herdr 里有对应的"形状"：

- 每个阶段（grill、prototype、spec、tickets、implement、review、retro，以及 triage、diagnose、wayfinder 三条 on-ramp）都有一个合适的 pane 布局。
- 当前所处阶段、任务图的 frontier、上下文余量（smart zone）随时可见，不靠人记。
- 在 phase boundary 上主动给出 Continue、`/clear`、`/handoff`、Subagent、`/compact` 五选一的建议，而不是事后补救。
- 把 Matt 流程里"人自己记着做"的机械动作（开 worktree、给每张 ticket 起一个新 agent、`/clear` 后接下一张、跑 review）做成一键操作。

非目标：不改 Matt 的 skill 本身，不重写 reviewr、file-viewer、agent-handoff 这些已有插件，也不做跨机器编排（那是 drove 的事）。

## 2. Matt SDLC 精读（设计依据）

来源：`~/.agents/skills/ask-matt/SKILL.md`、`ask-matt/PHASE-BOUNDARIES.md`，以及 grilling、to-spec、to-tickets、implement、implement-spec、retro、handoff、wayfinder、setup-matt-pocock-skills 各自的 SKILL.md。下面只记会影响设计的结论。

### 2.1 主流程：idea 到 ship

1. `/grill-with-docs`：按轮次访谈，一轮问完整个 frontier（所有前置决策都已定的问题），每题给推荐答案。有工作目录时把结论留进 `GLOSSARY.md` 和 ADR。
2. 分支：有问题在对话里定不下来（状态、业务逻辑、需要看见的 UI）时，绕道 `/handoff` 出去，在新目录跑 `/prototype`，再 `/handoff` 回来。prototype 保留在 `prototype/<name>` 分支上，作为 primary source。
3. 分支：是否需要多个 session 才能做完。
   - 是：`/to-spec`，再 `/to-tickets`（tracer-bullet 垂直切片，每张带 blocking edges），之后二选一：
     - 每张 ticket 跑一次 `/implement`，两张之间 `/clear`。
     - `/implement-spec`：把 tickets 当任务图，在 ready frontier 上并行起 implementer subagent，每个在自己的 worktree 和分支上，由 merger subagent 合进同一条 integration branch，最后对整条分支做一次 `/code-review`。
   - 否：在当前上下文里直接 `/implement`。
   - 两种做法内部都由 `/tdd`（一次一个 red-green 切片）驱动，以 `/code-review`（Standards 和 Spec 两个维度）收尾；提 PR 时 `/pr` 负责写 body。
4. `/retro`：复盘，改的是 agent 的环境（导航指针、自动检查、编码规范、steering 文件、工具），不改代码。机械性错误变成确定性检查，判断性问题写进编码规范。

### 2.2 On-ramps 与其他

- `/triage`：只处理别人提交的原始 issue，`/to-tickets` 产出的 ticket 不需要再 triage。
- `/diagnosing-bugs`：先拿到一条能在当前 bug 上稳定变红的命令（tight feedback loop），再动手修，并补一个回归测试。修完在同一个 session 里跑 `/retro`。
- `/wayfinder`：给一次装不进单个 session 的大工作用。产出的是 decision tickets（类型有 research、prototype、grilling、task，分 HITL 和 AFK），每个 session 只解决一张。地图看清后回到主流程的 `/to-spec`。
- 健康维护：`/improve-codebase-architecture` 找出可加深的模块，再拿到 `/codebase-design` 上设计。
- 底层词汇：`/domain-modeling` 管领域语言，`/codebase-design` 管 deep-module 词汇。

### 2.3 贯穿全流程的原则

| 原则 | 内容 | 对设计的约束 |
| --- | --- | --- |
| 上下文卫生 | 步骤 1 到 3 留在同一个不中断的上下文里，`/to-tickets` 之前不 compact 也不 clear | 这段时间要拦住或警告 `/clear` 和 `/compact` |
| Smart zone | 大约 150k token 以内模型推理还锐利；`/to-tickets` 前接近上限时，在最近的 phase boundary 上 compact | 需要 token 仪表，并标出 boundary |
| Phase boundary 决策树 | 按顺序问：能否 Continue → 上下文是否无关（`/clear`）→ 是否需要 handoff → 能否 AFK（Subagent）→ 否则 `/compact`。只在 boundary 上做这个决定 | 需要一个 boundary 顾问，并且能识别 boundary |
| Primary 和 secondary source | 除了 Continue，其他四个选项都会把 primary source 变成有损的 secondary source | 把"这样做会丢什么"展示出来 |
| 任务图与 frontier | ticket 是带 blocking edges 的 DAG，所有 blocker 都完成的 ticket 构成 frontier | 看板展示的是 frontier，不是线性列表 |
| HITL 和 AFK | 人参与的 ticket 必须真人在场；AFK 的可以后台跑 | 布局区分前台 driver 和后台 worker |
| Reviewer 负责执行规范 | implementer 的上下文压力最大，所以编码规范由 reviewer 执行 | review 用单独的 pane 和 agent，不在 implementer 上下文里做 |
| Context pointer | agent 之间少传正文，多传指针（spec、ticket、research 笔记、commit） | 跨 pane 通信只传路径或 ID |

### 2.4 工件清单（local markdown tracker）

| 工件 | 路径 | 产生者 |
| --- | --- | --- |
| 术语表 | `GLOSSARY.md` | grill-with-docs、domain-modeling |
| ADR | `docs/adr/` | domain-modeling |
| Spec | `.scratch/<feature>/spec.md` | to-spec |
| Tickets | `.scratch/<feature>/issues/NN-<slug>.md`，带 `Status:`、`Blocked by:` 字段 | to-tickets |
| Wayfinder 地图 | `.scratch/<effort>/map.md` 加 `issues/`，带 `Type:` 和 `Status: claimed/resolved` | wayfinder |
| 编码规范 | `CODING_STANDARDS.md` | retro |
| Prototype | `prototype/<name>` 分支 | prototype |
| Research | `research/<name>` 分支，ticket 里放指针 | research |
| Handoff | `$TMPDIR/*.md` | handoff |
| 仓库配置 | `docs/agents/{issue-tracker,domain,triage-labels}.md` | setup-matt-pocock-skills |

## 3. 平台能力盘点（本机实测，herdr 0.9.5）

### 3.1 Herdr 侧（证据：`herdr api schema`、已安装插件的 manifest）

- 插件 manifest `herdr-plugin.toml` 支持五种条目：`[[build]]`、`[[actions]]`（contexts 可以是 pane、workspace、global）、`[[panes]]`（placement 可以是 overlay、split、tab、zoomed、popup）、`[[events]]`、`[[startup]]`。
- 原生布局 API：`layout.apply` 接收一棵 `LayoutNode` 树，叶子是 `{type: pane, command, cwd, env, label}`，内部节点是 `{type: split, direction, ratio, first, second}`，可以应用到指定 tab 或新 tab。`layout.export` 能把现有 tab 导出成同样的结构。也就是说，布局可以声明式地定义，还能导出后版本化。
- 可订阅的事件：`pane.agent_status_changed`、`pane.agent_detected`、`pane.updated`、`worktree.created/opened/removed`、`layout.updated`、`tab.*`、`workspace.*`；另有 `events.subscribe` 提供 socket 流。
- 元数据总线：`pane report-metadata` 支持 `--token NAME=VALUE`（最多 16 个）、`--state-label`、`--title`、`--ttl-ms`；`workspace report-metadata --token`。侧栏的 `ui.sidebar.agents.rows` 和 `ui.sidebar.spaces.rows` 可以用 `$name` 渲染这些 token。
- 其他：`agent start/prompt/wait`（agent 生命周期状态有 idle、working、blocked、done）、`worktree create`、`notification show`、`agent.view.set`（agent 面板的过滤和排序）。
- 可以直接复用的已装插件：`persiyanov.reviewr`（diff 审阅，评论回灌给 agent）、`herdr-file-viewer`（只读文件 TUI）、`agent-handoff`（换一个 harness 接手会话）、`dev.volpestyle.mermaid-preview` 和 `hx-w.visuals`（Mermaid 渲染）、`herdr-navigator`。

### 3.2 Claude Code mod 侧（证据：plugin-authoring 的 types 文件和 reference.md，build 2.1.318）

- `skill.prompt` 事件在 skill 展开时触发，带 `skill` 名字。这是识别当前阶段最直接的信号。
- `turn.start`、`turn.complete`、`session.start`、`session.end`（`/clear` 时 `reason: 'clear'`）、`session.compact`、`prompt.submit`（可以改写或拦截输入文本）。
- `$.session.usage()` 读 token 用量；`$.process.run({argv})` 跑宿主命令，可以用来调 `herdr`；`$.env.get('HERDR_PANE_ID')` 读环境变量。
- UI：`AbovePrompt` 横条、`Pane` 侧栏、`$.ui.status`、`$.ui.toast`、`$.command.register` 注册斜杠命令。
- 状态：`$.state` 存在单个 session 里，`$.store` 跨 session 保留。

### 3.3 未验证、需要在原型里确认的假设

| 假设 | 风险 | 验证方式 |
| --- | --- | --- |
| A1：用户手敲 `/grill-with-docs` 这类 `disable-model-invocation: true` 的 skill 时，`skill.prompt` 也会触发 | 中 | 写一个最小 mod，只在这个事件里打 toast |
| A2：mod 的 `$.process.run` 在 safehouse 沙箱下能访问 herdr socket | 中 | 实际跑一次 `herdr pane report-metadata` |
| A3：`$.env.get` 能读到 `HERDR_PANE_ID` | 低 | 同上 |
| A4：`layout.apply` 应用到已有 tab 时会保留已有的 `pane_id` 叶子（不会杀进程） | 中 | 用一个名为 test 的 session 做实验 |
| A5：`prompt.submit` 能看到 `/compact`、`/clear` 这类内建命令的原始文本 | 高 | 最小 mod 验证；如果看不到，护栏就降级成只做提醒 |

## 4. 架构总览

两个交付物，加上它们之间的一份契约：

```
+------------------- Claude Code session（一个 pane 里）-------------------+
|  mod: matt-flow                                                           |
|   skill.prompt / turn.* / session.* --> FlowModel（纯状态机）             |
|   AbovePrompt 阶段条 + smart-zone 仪表 + boundary 顾问 + /flow 命令       |
|   publish --> herdr pane report-metadata --source matt-flow               |
+-------------------------------------|-------------------------------------+
                                      | Flow Signal（pane token，契约见第 7 节）
                                      v
+----------------------------- Herdr server --------------------------------+
|  plugin: matt-herd                                                        |
|   LayoutCatalog --> layout.apply（每个阶段一个布局）                      |
|   Tracker --> 读 .scratch（spec / tickets / map），算 frontier            |
|   panes: board（任务图和 frontier）、loop（反馈循环）                     |
|   actions: flow.open / ticket.next / ticket.fanout / boundary / retro     |
|   events: pane.updated、agent_status_changed、worktree.created            |
|   sidebar tokens: $phase $ticket $ctx $frontier                           |
+---------------------------------------------------------------------------+
            ^ 复用：reviewr、file-viewer、agent-handoff、mermaid-preview
```

分工原则：

- mod 负责"session 里面"：它知道 Claude 正在跑哪个 skill、用了多少 token、何时到了 boundary。
- 插件负责"session 外面"：布局、多 pane、worktree、多个 agent 的编排，以及从磁盘读工件。
- 两者只通过 Herdr 元数据（加上仓库里的工件文件）通信，互相不 import。仓库文件是唯一事实来源，Herdr token 只是带 TTL 的投影，丢了可以重新算。

## 5. 领域模型（GLOSSARY 草案）

| 术语 | 定义 | 避免使用的同义词 |
| --- | --- | --- |
| Effort | 一次完整的工作，对应 `.scratch/<slug>/`，在 Herdr 里是一个 workspace | 项目、任务 |
| Phase | Matt 流程里的一个阶段：`grill`、`prototype`、`spec`、`tickets`、`implement`、`review`、`pr`、`retro`、`triage`、`diagnose`、`wayfinder`、`health` | 步骤、状态 |
| Boundary | 两个 phase 之间的间隙，是唯一允许做上下文决策的时机 | 断点 |
| Boundary Move | 五选一：`continue`、`clear`、`handoff`、`subagent`、`compact` | 动作 |
| Unbroken Window | 从 grill 到 tickets 必须保持的不中断上下文区间 | |
| Driver | 人在场（HITL）的主 Claude pane | 主 agent |
| Worker | AFK 的后台 agent pane（implementer、reviewer、researcher） | 子 agent（这个词留给 Claude 内部的 subagent） |
| Stage Layout | 某个 phase 对应的 Herdr pane 布局（一棵 LayoutNode 树） | 模板 |
| Frontier | 所有 blocker 都已 done、并且没人认领的 ticket 集合 | 待办 |
| Flow Signal | mod 发布到 Herdr 的 pane token 集合 | 状态 |

## 6. 模块设计（按 deep-module 原则）

下面每个模块都是一个"小接口、厚实现"的模块，并标出测试 seam。

### 6.1 FlowModel（纯函数，mod 和插件各带一份，或者共享）

- 接口：`reduce(state, event) -> state`，以及 `advise(state) -> BoundaryAdvice`。
- 事件：`SkillInvoked(name)`、`TurnCompleted`、`Cleared`、`Compacted`、`UsageSampled(tokens)`、`HandoffWritten(path)`。
- 状态：当前 phase、track（`main`、`triage`、`diagnose`、`wayfinder`、`health`）、unbroken window 是否开启、已用 token、上一个 boundary。
- skill 名到 phase 的映射表（数据驱动，放在可配置的 TOML 里）：

| skill | phase | 备注 |
| --- | --- | --- |
| grill-with-docs、grill-me、grilling、domain-modeling | grill | 打开 unbroken window |
| prototype | prototype | 一般在另一个目录 |
| to-spec | spec | |
| to-tickets | tickets | 完成后关闭 unbroken window，这是一个 boundary |
| implement、implement-spec、tdd | implement | |
| code-review | review | |
| pr | pr | |
| retro | retro | |
| triage | triage | on-ramp |
| diagnosing-bugs | diagnose | on-ramp |
| wayfinder | wayfinder | on-ramp |
| improve-codebase-architecture、codebase-design | health | |
| handoff | 不改变 phase | 记录 handoff 文件路径，作为 context pointer |

- `advise` 把 PHASE-BOUNDARIES 的决策树编码成规则：
  1. 下一个 phase 需要当前 phase 作为 primary source（例如 grill 到 spec），并且余量大于 150k 减已用：建议 `continue`。
  2. 下一个 phase 和当前上下文无关（例如两张 ticket 之间）：建议 `clear`。
  3. 换 harness、换目录（prototype）或转交给别人：建议 `handoff`。
  4. 可以 AFK（例如 review）：建议 `subagent`，或者在 Herdr 里起一个 Worker pane。
  5. 其余情况：建议 `compact`，并给出一条带 phase 提示的 compact 指令，例如 `/compact we're going to QA this area`。
- 测试 seam：纯函数，用表驱动测试覆盖决策树的每一条分支。

### 6.2 Tracker（插件内）

- 接口：`load(effortDir) -> TaskGraph`、`frontier(graph)`、`claim(ticket)`、`status(ticket)`。
- 适配器：`LocalMarkdownTracker` 先做，解析 `Status:`、`Blocked by:`、`Type:` 这几行；`GitHubTracker` 后做，基于 `gh` 和原生的 sub-issue 与 blocking 关系。
- 测试 seam：用 `.scratch` 的 fixture 目录做测试；GitHub 适配器用录制好的 `gh` 输出。
- 这里只读写 Matt skill 已经约定的字段，不发明新格式，避免和 `/to-tickets`、`/wayfinder` 冲突。

### 6.3 LayoutCatalog（插件内）

- 接口：`build(stage, ctx) -> LayoutNode`，ctx 包含 cwd、effort、ticket、agent kind。
- 布局用 TOML 声明，用户可以覆盖；也可以用 `layout.export` 把手工调好的 tab 导出，存成新布局。
- 测试 seam：对 LayoutNode JSON 做快照测试。

### 6.4 HerdrClient（插件和 mod 共用的适配器）

- 封装 `herdr` CLI 的 JSON 调用：`layout.apply`、`report-metadata`、`agent start/prompt`、`worktree create`、`plugin pane open`。
- 测试 seam：注入一个假的 runner，断言发出的 argv。

## 7. 契约：Flow Signal

mod 每次状态变化时调用：

```
herdr pane report-metadata "$HERDR_PANE_ID" --source matt-flow --agent claude \
  --token phase=implement --token track=main --token effort=herdr-layout \
  --token ticket=03 --token ctx=92k/150k --token window=closed \
  --state-label working="implement 03" --ttl-ms 3600000
```

- token 名都在 16 个以内，并且符合 `^[A-Za-z0-9_-]{1,32}$`。
- 插件订阅 `pane.updated`，读取这些 token，聚合后写进 `workspace report-metadata`（例如 `$frontier=2/7`），供侧栏的 spaces 行显示。
- 侧栏配置示例（写进 `config.toml`）：

```toml
[ui.sidebar.agents.rows_by_agent]
claude = [["state_icon", "workspace", "tab"], ["$phase", "$ticket", "$ctx"], ["agent"]]

[ui.sidebar.spaces]
rows = [["state_icon", "workspace"], ["branch", "git_status", "$frontier"]]
```

- 版本化：增加一个 `--token flow_v=1`，插件看到不认识的版本时只显示，不做编排。

## 8. Stage Layouts

每个布局都是一棵 LayoutNode 树。下面用示意图表示，比例是默认值。

### 8.1 shape（grill、spec、tickets；Unbroken Window 期间）

```
+------------------------------+-----------------+
|                              | file-viewer     |
|  Driver: claude              | GLOSSARY / ADR  |
|  /grill-with-docs            | / spec.md       |
|                              +-----------------+
|                              | Worker: research|
|            0.62              | （AFK，按需）   |
+------------------------------+-----------------+
```

右上角盯着这次 grilling 正在写的工件；右下角用来跑 grilling skill 里说的"事实由 agent 去查"的那类探索，或者跑 `/research`。

### 8.2 prototype（独立目录或 `prototype/<name>` 分支）

```
+------------------------+-----------------------+
|  claude（新 session，  |  prototype 运行输出   |
|  读 handoff 文件）     |  （dev server / TUI） |
+------------------------+-----------------------+
```

进入时 cwd 切到原型目录，并把 handoff 文件路径作为第一条 prompt 传入。退出时提示 `/handoff back`。

### 8.3 build-serial（`/implement` 逐张做）

```
+-----------------------------+------------------+
|  Driver: claude             |  board           |
|  /implement .scratch/x/03   |  frontier + DAG  |
|                             +------------------+
|                             |  shell：测试和    |
|                             |  typecheck       |
+-----------------------------+------------------+
```

action `ticket.next` 做的事：标记当前 ticket done，然后在 Driver 里执行 `/clear`，从 frontier 取下一张，发 `/implement <path>`。要不要每次都先问人一句，见第 11 节 Q4。

### 8.4 build-parallel（`/implement-spec` 的 Herdr 可视化版）

两种实现路线，见 Q3：

- 路线 A（观察）：`/implement-spec` 仍然由 Claude 内部的 subagent 完成，Herdr 只负责展示。board 显示每张 ticket 对应的 worktree 和分支状态。
- 路线 B（编排）：`ticket.fanout` 给 frontier 上的每张 ticket 执行 `worktree create`，开新 workspace，再用 `agent start` 起一个 Worker 并发 `/implement <ticket>`。插件监听 `agent_status_changed` 等待 done，起一个 merger Worker，然后推进 frontier。integration workspace 里开 reviewr。

```
integration workspace
+----------------------+-------------------------+
|  Driver: claude      |  board（每张 ticket 对  |
|  （协调与合并）      |  应的 agent 状态）      |
+----------------------+-------------------------+
|  reviewr（integration branch 的 diff）         |
+------------------------------------------------+
ticket workspace（每张一个，由 worktree 自动生成）
+------------------------------------------------+
|  Worker: claude /implement 0N（在 worktree 里）|
+------------------------------------------------+
```

### 8.5 review

```
+----------------------+-------------------------+
|  reviewr（diff）     |  Worker: claude         |
|                      |  /code-review <base>    |
+----------------------+-------------------------+
```

reviewer 用一个干净的新 agent，对应原则"由 reviewer 执行规范"。reviewr 里的行评论回灌给 Driver。

### 8.6 diagnose

```
+-----------------------------+------------------+
|  Driver: claude             |  loop pane       |
|  /diagnosing-bugs           |  反复执行那条    |
|                             |  red 命令：      |
|                             |  RED / GREEN     |
+-----------------------------+------------------+
```

loop pane 是插件提供的反馈循环面板：文件变化时重跑命令，显示红绿状态和变化历史。它把 diagnosing-bugs 要求的 tight feedback loop 做成看得见的东西。

### 8.7 wayfinder

```
+-----------------------------+------------------+
|  Driver: claude             |  map 视图        |
|  /wayfinder <map>           |  Decisions /     |
|                             |  Frontier / Fog  |
+-----------------------------+------------------+
```

research 类 ticket 可以选择开成 AFK 的 Worker pane。不变式：每个 session 只解决一张 ticket。

### 8.8 retro

单个 pane。`retro.collect` action 把本次 effort 涉及的 session 日志路径收集好，交给 `/retro`。

## 9. matt-flow mod 的用户界面

- AbovePrompt 阶段条：`grill > spec > tickets > [implement 03/07] > review > retro`，后面跟 `ctx 92k/150k` 仪表。超过 120k 显示黄色，超过 150k 显示红色。
- Boundary 顾问：`turn.complete` 时，如果这一轮触发过一个"会结束阶段"的 skill（to-spec、to-tickets、code-review 等），在 AbovePrompt 显示 `advise()` 的结果和五个按钮。按下后把对应命令填进输入框，或者调用插件 action（比如开一个 Worker），不自动执行。
- 护栏：Unbroken Window 期间，如果 `prompt.submit` 看到 `/clear` 或 `/compact`，给出警告并要求确认。这依赖假设 A5；如果 A5 不成立，退回到在 `session.end reason=clear` 之后补一条 toast 提醒。
- 斜杠命令：`/flow` 显示当前状态和下一步建议；`/flow layout` 让插件按当前 phase 应用 Stage Layout；`/flow boundary` 手动唤起顾问。
- `$.store` 记住每个 effort 最近一次的 phase，新 session 打开时恢复阶段条。

## 10. 仓库结构与技术选型（建议）

```
matt-herdr/
  GLOSSARY.md
  docs/adr/
  docs/design/            本文档
  packages/flow-model/    纯 TS，零依赖（mod 和插件共用）
  herdr-plugin/           matt-herd：herdr-plugin.toml + Node（TS 编译产物）
    layouts/*.toml
  claude-mod/             matt-flow：.claude-plugin/ + hooks/register.tsx
  .scratch/               本项目自己的 local tracker（自举用）
```

- 插件用 Node 加 TypeScript，和已装的 hx-w.visuals、agent-handoff 一致，build 用 `npm ci`；board 和 loop 两个 TUI 可以用 Ink。如果更看重启动速度，可以换 Rust（参考 reviewr、navigator），见 Q6。
- mod 里不能 import 自己目录以外的文件，所以 flow-model 在构建时复制进 `claude-mod/hooks/vendor/`，并加一个检查，确保两份字节完全一致。
- 自举：本项目自己用 Matt 流程开发。先跑 `/setup-matt-pocock-skills`（选 local markdown tracker），在 `.scratch/` 里管理 tickets。

## 11. 待定决策（第一轮 grilling）

按 grilling 格式写：每题给推荐答案，回答"是"即表示接受推荐。

**Q1 交付形态**：是否接受"一个 Herdr 插件（matt-herd）加一个 Claude Code mod（matt-flow），只通过 Herdr 元数据通信"这个双交付物架构？
推荐：是。另一个选项是只做 Herdr 插件，靠解析终端输出来猜当前 skill，但那样又脆弱又不准。

**Q2 Driver 的 agent 种类**：只支持 Claude Code 作为 Driver，还是一开始就兼容 Codex 和 pi？
推荐：v1 只支持 Claude（mod 只能跑在 Claude 里）；在 Flow Signal 契约里留出 `--source`，以后其他 harness 可以发布同样的 token。

**Q3 并行构建**：build-parallel 走路线 A（只观察 `/implement-spec`）还是路线 B（由 Herdr 用真实 pane 和 worktree 编排）？
推荐：先做 A，再做 B。B 的价值在于每个 Worker 都可见、可接管、可在 reviewr 里审，但它要复刻 implement-spec 的 merger 逻辑，复杂度高，适合放进第二个里程碑。

**Q4 自动化程度**：`ticket.next` 和 boundary 按钮是"填好命令等人按回车"，还是"直接执行"？
推荐：默认只填不发；在插件配置里提供 `auto = true`，给 AFK 场景使用。理由：Matt 流程强调 boundary 是一个判断，人要在场。

**Q5 Tracker 范围**：v1 只支持 local markdown，还是同时支持 GitHub？
推荐：v1 只做 local markdown，Tracker 接口按两个适配器来设计。

**Q6 插件实现语言**：Node 加 TypeScript，还是 Rust？
推荐：Node 加 TypeScript，好处是能和 mod 共用 flow-model。

**Q7 护栏力度**：Unbroken Window 期间的 `/clear` 和 `/compact`，是警告并确认，还是直接拦截？
推荐：警告并确认，不硬拦，因为 smart zone 超限时 Matt 本人也允许 compact。

**Q8 布局触发**：phase 变化时自动切换 Stage Layout，还是只在 `/flow layout` 或某个快捷键时切？
推荐：不自动切，避免打乱人正在看的 pane；phase 变化时用 toast 提示"有可用的布局"。worktree 自动生成的 workspace 例外，可以自动套布局。

## 12. 里程碑草案（grilling 之后由 `/to-tickets` 正式切分）

全部是 tracer-bullet 垂直切片：

1. M0 验证假设：一个最小 mod，在 `skill.prompt` 里调用 `herdr pane report-metadata`，侧栏显示 `$phase`。一次验证 A1、A2、A3。
2. M1 阶段可见：FlowModel、AbovePrompt 阶段条、smart-zone 仪表、`/flow`。
3. M2 布局：插件骨架、LayoutCatalog、`flow.open shape|review|diagnose`，复用 file-viewer 和 reviewr。
4. M3 任务图：LocalMarkdownTracker、board pane、`$frontier` token、build-serial 布局和 `ticket.next`。
5. M4 Boundary 顾问：`advise()` 规则、按钮、Unbroken Window 护栏。
6. M5 on-ramps：loop pane（diagnose）、wayfinder 的 map 视图、`retro.collect`。
7. M6 并行编排：路线 B 的 `ticket.fanout`、Worker 生命周期、merger、integration workspace。
8. M7 扩展：GitHubTracker、`layout.export` 导出自定义布局、其他 harness 的 Flow Signal。

## 13. 置信度说明

- 高：对 Matt SDLC 的解读，逐字依据本机 skill 原文。
- 高：Herdr 的 `layout.apply`、元数据 token、事件列表，依据 0.9.5 的 API schema。
- 中：mod 端的事件语义，依据类型声明和文档注释，还没有实际跑过（见 3.3 节 A1 到 A5）。
- 中：布局比例和 pane 组合属于主观设计，需要在 M2 实际用过之后调整。
