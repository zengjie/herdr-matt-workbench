# Matt SDLC × Herdr：简单的统一开发工作区

文档状态：实施 Spec；测试边界已由用户确认。发布目标为本仓库 GitHub Issues。
发布标签：`ready-for-agent`。GitHub issue 是后续更新与工作项关联的权威记录。
设计依据：统一工作区 v0.4、工作流启用 ADR 与 E 原型；A–D 为探索记录。

## Problem Statement

开发者并非在每个工作区都使用 Matt；启用 Matt 时，希望按照 Matt skills 的 SDLC 推进日常开发：先澄清问题，保存共同决策，需要时验证原型，再形成 Spec 与 Implementation Ticket；范围较大时并行开发，最后审阅、集成、验收和复盘。

现有设计已经探索了这些交互，但仍是内存中的 HTML 原型。它没有真实 Herdr mods、Agent 运行控制、持久记录或实际代码验证，不能作为可靠的日常开发工作台。

日常使用的主要困难是：

- 开发者需要反复切换对话、终端、任务和报告，自己拼接全局状态。
- CoS 同时负责用户沟通与任务调度，容易丢失诉求；后台通知又可能打断或混入人工输入。
- 并行工作中，纠偏对象、受影响的依赖、原会话现场和旧结果是否仍有效不够清楚。
- Agent 完成、代码提交、自动检查通过、人工验收与集成成功容易混为一谈。
- 丢回执、断线、窗口重载和进程退出后，用户无法确信操作是否执行，可能重复启动任务。
- 为解决上述问题不断增加面板、运行编号和配置，会进一步增加认知负担，小修复也被迫走完整编队流程。

## Solution

Matt 按 Workspace 显式启用，其他 Workspace 保留现有开发方式，不启动本项目的对话 Agent 或调度。Workspace 由 Herdr Space 与明确绑定的固定工作目录界定；Effort 是其中围绕目标组织的工作集合，二者不等同。

实现一个 Herdr 原生 layout 与相应 mods，提供同一个 Effort 的“对话、任务、结果”三个视图。默认显示当前进展与下一步；只有需要人工判断的决策、验收和阻塞进入“需要你”。运行记录、模型配置、后台事件和技术证据按需展开。

在启用 Matt 的工作区中，用户通过一个独立主会话的对话 Agent 交流。它读取共同事实，将清晰、已授权的诉求转译为指向明确工作项和运行的请求。唯一 CoS 负责依赖、认领、执行协调与集成，不再承担日常前台沟通。操作桥负责可靠传递、持久记录和恢复核对，不新增调度策略。

小修复直接使用一个 Worker；复杂工作复用 Matt skills 的 Spec、Ticket、独立上下文和 worktree 约定。纠偏默认继续原 Agent，验收绑定实际版本，修改后重新验证，集成通过后才放行依赖。

第一条真实接入路径在本机、单仓库、单 Effort 中跑通，再完成同仓库多个 Effort 的草稿与运行隔离。这个顺序是实施顺序，不取消最终的 Effort 切换行为。Codex 与 Claude Code 均作为可选择的运行器，角色、模型和思考强度分开配置。

## User Stories

1. As a developer, I want a unified view for an Effort in its enabled workspace, so that I can understand its goal, work and evidence without assembling them myself.
2. As a developer, I want only conversation, tasks and results as the main views, so that routine navigation stays simple.
3. As a developer, I want a short progress summary and a clear next action, so that I know what to do when I return.
4. As a developer, I want a dedicated dialogue Agent, so that I can express intent without choosing an execution Agent first.
5. As a developer, I want the dialogue Agent to retrieve current project facts, so that its answers reflect actual work rather than remembered guesses.
6. As a developer, I want my conversation to remain available while reading tasks and results, so that navigation does not break my discussion.
7. As a developer, I want background events to arrive independently, so that they never alter my draft, focus or text selection.
8. As a developer, I want new messages to queue during a response, so that I can add information without mixing message boundaries.
9. As a developer, I want to attach files, code ranges, screenshots and logs, so that feedback points to inspectable evidence.
10. As a developer, I want to see what my message is discussing, so that selecting a task does not silently redirect my instructions.
11. As a developer, I want clear authorized instructions to proceed without repeated confirmations, so that control does not become a sequence of permission dialogs.
12. As a developer, I want ambiguous execution targets to be clarified, so that an Agent does not act on the wrong task.
13. As a developer, I want decisions, acceptance requests and meaningful failures in one attention list, so that ordinary progress does not compete for attention.
14. As a developer, I want Agent reading and human resolution tracked separately, so that an unanswered question cannot disappear after summarization.
15. As a developer, I want decisions saved with their sources, so that future sessions can recover the agreed requirements.
16. As a developer, I want a small fix to start with one Worker, so that I do not need to configure a team for every change.
17. As a developer, I want complex work to use Spec and dependency-linked Tickets, so that parallel execution preserves the intended behavior slices.
18. As a developer, I want each parallel Ticket to use its own worktree and context, so that independent work does not overwrite another task.
19. As a developer, I want to see the Frontier and blocked dependencies, so that I can distinguish actionable work from a backlog.
20. As a developer, I want missing or invalid dependency facts to stay unknown, so that the system cannot mistake incomplete information for readiness.
21. As a developer, I want one effective claim per work item, so that retries and multiple windows cannot launch competing owners.
22. As a developer, I want concurrency and preview resources controlled, so that parallel work does not exhaust capacity or replace another preview.
23. As a developer, I want to correct one task while unrelated work continues, so that intervention preserves useful progress.
24. As a developer, I want the affected dependency chain identified, so that old results cannot bypass a changed shared constraint.
25. As a developer, I want a pause request distinguished from a confirmed pause, so that I know what has actually stopped.
26. As a developer, I want the scope of interruption explained, so that I do not assume a preview, subprocess or external action was rolled back.
27. As a developer, I want the Worker to acknowledge revised constraints before resuming, so that my correction becomes part of its working basis.
28. As a developer, I want correction to resume the original Agent by default, so that its context and worktree are retained.
29. As a developer, I want repeated corrections to remain possible, so that the first intervention does not exhaust the control path.
30. As a developer, I want an explicit handoff when changing runners, so that the new session has the required context and the old run remains traceable.
31. As a developer, I want to open the relevant native terminal directly, so that I can inspect or intervene without asking the dialogue Agent to relay everything.
32. As a developer, I want shared constraints from direct intervention recorded once, so that I do not have to repeat them to the CoS.
33. As a developer, I want results tied to an exact code snapshot, so that I know what I am accepting.
34. As a developer, I want the preview, diff, environment and test data identified, so that I can reproduce the displayed behavior.
35. As a developer, I want verification commands and output available, so that a success label has inspectable evidence.
36. As a developer, I want independent Standards and Spec reports on the same range, so that style compliance cannot substitute for requirement coverage.
37. As a developer, I want acceptance actions at the top of the result, so that I can act without searching through logs.
38. As a developer, I want to accept, request changes or defer acceptance, so that my decision is explicit.
39. As a developer, I want feedback tied to its version and location, so that the Worker can understand the expected and observed behavior.
40. As a developer, I want rework to return a new result for review, so that old approval cannot silently apply to revised code.
41. As a developer, I want manual code changes to invalidate affected evidence, so that reports cannot remain green for a different snapshot.
42. As a developer, I want integration checked against the current shared baseline, so that individually valid branches do not imply a valid combined result.
43. As a developer, I want failed or stale integration candidates to preserve the baseline, so that downstream work cannot start from an unverified result.
44. As a developer, I want completion defined for the Effort, so that local acceptance, integration and publication are not confused.
45. As a developer, I want drafts, questions, requests and run bindings to survive real reloads, so that I can safely resume after interruption.
46. As a developer, I want lost acknowledgements reconciled by their original request IDs, so that recovery does not duplicate execution.
47. As a developer, I want unknown execution state shown explicitly, so that reconnecting does not fabricate success.
48. As a developer, I want a surviving Worker recognized after CoS restart, so that its ongoing work is not claimed again.
49. As a developer, I want quota, network and verification failures to preserve the scene, so that I can recover rather than reconstruct the task.
50. As a developer, I want to switch Efforts without losing drafts or reading position, so that temporary work does not erase my main thread.
51. As a developer, I want role-specific runner, model and reasoning settings, so that responsibilities are independent of a provider.
52. As a developer, I want defaults and task overrides, so that exceptional work can use a different configuration without changing the whole team.
53. As a developer, I want requested and actual configurations distinguished, so that a saved setting cannot misrepresent an existing run.
54. As a developer, I want unavailable capabilities reported explicitly, so that the system cannot silently choose another model or effort level.
55. As a developer, I want unsaved configuration edits preserved by scope, so that opening another panel does not discard my choices.
56. As a developer, I want configuration and logs hidden until needed, so that routine development does not require understanding runtime internals.
57. As a developer, I want narrow-window navigation and accessible drawers, so that the workspace remains usable without a permanently wide layout.
58. As a developer, I want Stage Profiles and context handoff suggestions without automatic completion claims, so that Matt's workflow supports judgment rather than forcing a pipeline.
59. As a developer, I want retrospective notes to retain their source context, so that improvements can update the next Agent's working environment.
60. As a maintainer, I want acceptance primarily exercised through the real Herdr interface, so that a passing browser prototype cannot substitute for a working native product.

61. As a developer, I want to choose Matt separately for each workspace, so that other development workflows stay available.
62. As a developer, I want to bind a workspace to a Space and fixed directory, so that terminal directory changes cannot redirect dispatch.
63. As a developer, I want to enter and leave the Matt layout explicitly, so that selecting another Space cannot replace the whole window layout.
64. As a developer, I want to use an ordinary terminal for non-Matt Spaces, so that I can keep my existing tools without starting extra Agents.
65. As a developer, I want to reuse the existing Matt entry for the same directory, so that duplicate Spaces cannot create competing schedulers.
66. As a developer, I want to disable Matt by draining current work, so that new Tickets stop while existing verification, rework and acceptance remain possible.
67. As a developer, I want to pause and retain the scene or cancel disabling, so that I can leave without killing Agents or losing unfinished work.
68. As a developer, I want to reconcile retained work before explicitly continuing, so that reenabling cannot duplicate execution or reuse stale acceptance.
69. As a developer, I want to retain bindings on rename and explicitly repair invalid bindings, so that Space recreation or directory changes cannot silently move active work.
70. As a developer, I want to see a quiet aggregate attention marker outside Matt, so that background questions do not steal focus or enter my messages.

## Implementation Decisions

1. **实现交付物。** 构建个人 Herdr layout、所需的原生 mods，以及一个承载运行器连接与可靠操作的本地进程。复用现有终端、会话定位和原生交互组件；不直接移植 HTML/CSS，不重写 Herdr 终端。新增模块以统一工作区、操作桥与两个运行器接入为边界，不建立通用插件平台。

2. **主布局。** 顶部展示 Effort 目标、简短进展与“需要你”；主区仅保留对话、任务、结果三个视图。桌面复用 Space 导航，在 Matt 工作区内选择 Effort；窄窗口收为选择器。日志、请求编号、运行配置和代码详情放入可关闭的按需面板。不得把 A–E 变体选择器或原型故障按钮带进正式产品。

3. **统一投影与事实归属。** 三个视图读取同一 Effort 状态投影。tracker 拥有原始工作项及依赖语义，git 拥有版本事实，运行器拥有会话与轮次事实，检查器拥有验证结果，用户拥有人工作出的决策与验收。操作记录保存请求及回执。UI 偏好不能充当任务锁，Agent 对话记忆不能成为任务数据库。

4. **身份与隔离。** Workspace 以机器、稳定 Space 身份及显式固定工作目录绑定；目录按机器规范化，不以 live cwd 或显示名称作为身份。同一仓库的不同 worktree 可独立选择工作流。Effort 以本机、规范化仓库根目录和显式工作集合身份绑定；工作项、Session、run、terminal、请求和证据版本分别有稳定身份。不要通过名称或目录字符串前缀猜测 worktree 归属。Effort 切换隔离草稿、讨论对象、阅读选择、未决问题、配置草稿和异步响应；过期响应不能覆盖新选中的 Effort。

5. **对话与调度拆分。** 对话 Agent 使用独立主会话，通过读取当前投影和 Context Pointer 获取目标、共同决策、任务、最近证据与未决请求。它将诉求转成明确操作，并依据实际回执说明状态。唯一 CoS 负责认领、依赖、约束传播和集成顺序；采用 implement-spec 时沿用其调度权。操作桥只传递和记录，不依据模型猜测自行调度。

6. **消息边界。** 人工消息、后台事件、运行器回执、Agent 摘要分别存储。通知到达不修改输入、焦点或选区，不抢占主视图。生成中的新人工消息按原顺序排队；用户可明确选择中断当前轮次。事件摘要保留来源编号，读取不等于人工解决。后台提问进入独立的人工决策记录。

7. **对象绑定与指令执行。** 阅读对象不改变讨论对象；操作请求绑定具体 Effort、工作项和运行。对含糊目标只询问必要澄清。目标明确且已授权的指令直接发起请求，不强制用户再确认每个内部步骤。暂停、交接、返修等实际状态仍需回执；“请求已记录”不能表述为“已完成”。

8. **上下文附件与依据。** 消息支持实际文件、代码范围、截图批注和日志引用，并保存可恢复的引用或快照。决策写入共同依据，记录来源及版本。直接进入 Worker 会话时显示接管对象；产生共享约束后回写一次，并由 CoS 确认影响，避免要求用户重复转述。

9. **轻量路径与 Matt SDLC。** 小修复直接启动一位 Worker，沿用配置，不要求 Spec 拆分或常驻五个角色。复杂工作消费既有 Spec 与 Implementation Ticket，保留 Dependency 与 Context Pointer。阶段以明确来源或人工判断标注；Stage Profile 切换、Agent done、terminal idle 都不表示阶段完成。必要时提供 Continue、clear、handoff、subagent、compact 建议；未知上下文占用保持未知，不伪造容量百分比。

10. **依赖与认领。** 区分 Implementation Ticket 与 Decision Ticket 的完成语义，读取项目 tracker 的明确约定；`ready-for-agent` 是 triage 标签，不是完成。依赖缺失、重复身份、环、未知状态或陈旧读取不能放行 Frontier。每个工作项只允许一个有效执行认领；窗口重载、重复投递和 CoS 重启不产生第二个认领。无可用原子认领/去重能力时，应停留在待核对状态，不承诺安全自动启动。

11. **并行与资源。** 独立 Worker 使用独立上下文与 worktree。新启动服从明确作用域的并行上限，降低上限不终止已有运行。预览端口与必要环境在本机工作范围内明确占用和释放，不能因 Effort 切换或固定端口复用而覆盖其他任务。依赖任务在已满足声明完成条件的共同基线上启动。

12. **纠偏过程。** 保存目标、变更约束、影响集合及版本；先发出暂停请求，收到准确回执后，保存并确认采用新约束，再继续原 Session。用户默认无需管理这串内部步骤，UI 只显示当前事实和必要选择。影响集合由 CoS 按完整依赖图及共享约束计算，不能局限为原型中的直接依赖。无关工作继续，受影响未集成结果失去原验收资格。

13. **纠偏有效性。** 已进入纠偏的旧结果不能通过重复点击“重新验证”恢复验收资格。必须确认新约束采用，完成相应修改并返回适用于该约束的新证据。纠偏可重复进行；每次请求均保留关联与回执。对已集成结果的约定变更创建后续修复工作项，不篡改历史集成事实。

14. **暂停与恢复范围。** 区分 Agent 轮次中断、子进程停止、预览保留与外部动作状态。中断确认不等于代码回滚或全部进程停止。默认恢复原运行及现场；额度或连接问题保留可恢复状态。无法核实时显示未知，并提供对具体运行的查询或人工介入入口。

15. **跨运行器交接。** 只有明确选择交接并确认旧运行已暂停，才建立新运行。交接携带目标、约束、来源、未决问题、worktree、版本、反馈与未决请求。旧运行与其实际配置保留为历史；新旧会话身份不复用。新的认领生效前旧认领必须失效，不能同时保留两个可执行所有者。

16. **结果证据。** 结果包绑定 repo、worktree、base/head、未提交改动快照标识、约束版本、预览入口、环境及数据、验证命令与输出。Standards 与 Spec 在独立审阅上下文中检查相同固定范围，不能因同一个成功标签替代其中一项。缺少可判定 Spec 或检查失败时明确列出缺口。

17. **人工验收与返修。** 需要人工判断的结果显示通过、要求修改、稍后验收，主操作位于结果顶部。反馈保存原版本、位置、预期、实际与阻塞性。返修沿用原运行，返回新结果后重新验收；新代码或相关约束变化使旧检查和验收失效。关闭反馈需要适用新版本的复验或明确人工决策，不按消息已读自动关闭。

18. **集成与完成。** 已验收且证据有效的结果，由唯一集成者在当前共同基线上建立候选；候选与基线、结果或约束不再匹配时不得采用。验证失败不推进共同基线、不放行下游。获授权的正常集成由 CoS 连续推进，不要求用户逐个点击内部步骤。首版验收条件为本地人工验收与集成检查，不把 Agent done、提交或本地完成称为 PR/CI/发布成功。

19. **操作契约。** 每条执行请求包含稳定 request ID、Effort、工作项、目标 run、操作类型、负载摘要与相关版本。持久记录成功后才投递；区分已记录、已投递、已确认及失败/未知，另行记录操作结果。执行侧以相同 request ID 和工作项认领核对重复请求；相同 ID 的不同负载拒绝执行。回执必须匹配原请求和运行，陈旧回执不覆盖新认领。

20. **真实持久化与恢复。** 本地操作进程承担长期连接与写入责任，使用最小满足原子更新、版本校验和重放恢复的持久记录，不预设通用数据库平台。UI 重载或进程重启后从记录恢复草稿、问题、请求、配置和绑定，随后与运行器、tracker 和 git 对账。先查询未确认的原请求，再决定是否沿用原 ID 重试；不可安全去重的操作先查实际进程、提交或外部对象，不盲目重试。CoS 退出而 Worker 仍活着时保留现场，不重新启动它。

21. **Herdr 生命周期。** 按原生 gpui-kit 和本机类型接口实现，复用 live data，不在 mod 重绘中启动 Agent，也不把长期会话放入限时短命令。mod 通过短请求和可观察投影与长期操作进程交互。遵守 shipped layout/mod 克隆与校验规则，保留既有布局的恢复入口。读取失败保留旧数据显示陈旧，不显示空列表成功态。

22. **运行器与角色配置。** 对话、CoS、开发、审阅、集成职责均可选择 Codex 或 Claude Code。角色数量不等于常驻进程数量。复用安装版本支持的模型与思考强度能力发现，不把研究快照硬编码为账号授权。工作项覆盖优先于 Effort，Effort 优先于工作区默认；未保存草稿按作用域保留。配置变更影响后续启动，不能静默更换进行中的会话。

23. **实际配置与能力限制。** 保存请求的运行器、模型标识/别名、思考强度和运行侧实际解析结果及来源。能力不可用时显式报告，不能静默降档或换模型；运行器报告实际降档时保留原因。跨提供商档位名不代表同等计算量。自动委派工作流与普通思考强度分开，首版不默认启用；模型选择不扩大工具或操作权限。

24. **可访问性与渐进展示。** 窄窗口下主操作、项目选择和输入仍可达，代码与日志在自己的阅读区域滚动，不撑宽页面。抽屉支持键盘导航、焦点管理、Escape 关闭和回到原操作位置；状态不能只靠颜色区分。后台更新不得收起正在编辑的反馈或丢失其选区。日志或 token 等非决策信息不进入默认主屏。

25. **启用范围。** 默认保留原工作台，由用户对 Workspace 显式启用 Matt。非 Matt Workspace 不启动本项目的对话 Agent、CoS 或调度，不建设通用 Agent 工作台。终端 cd 不改变固定目录绑定或派工目标。启用状态与绑定必须持久保存。

26. **窗口布局与 Space。** Herdr 视觉 layout 作用于整个窗口。进入或退出 Matt 工作台由用户显式触发，返回原工作台时保留原布局入口；Space 选择与焦点变化不自动切换整个窗口布局。在 Matt 窗口选中非 Matt Space 时显示普通终端、隐藏 Matt 专属面板，保留返回入口；不自动启用或弹配置向导。

27. **唯一入口与 Worker 归属。** 同一机器上的同一规范化固定目录只有一个 Matt 调度入口，重复 Space 启用定位已有入口，不建立第二套 CoS。Workspace 内所有 Effort 遵循该调度归属；Matt 创建的 Worker worktree 继续属于原 Effort，不自动成为独立调度工作区。不同 worktree 的独立工作流选择不等于自动为 Worker 启用 Matt。

28. **停用与收尾。** 停用先持久记录 Draining，停止新 Ticket 派工；现有任务可继续验证、返修、人工验收和已授权集成，全部收尾后完成停用。切换 Space 或退出 Matt 视觉 layout 均不等于停用。提供“暂停并保留现场”和“取消停用，恢复 Matt”；暂停必须确认实际运行状态，未知或未回执不能显示已停用。保留 Agent、工作目录、任务、反馈、未决问题和证据，不直接杀进程、删除现场或清空问题。

29. **恢复与绑定变化。** 重新启用保留原工作，先查询 Agent、原请求、目录及实际产物，再提供明确的继续操作；不自动重新派工，不把旧验收适用于变化后的版本。Space 改名不改绑定；Space 删除后重建、改绑目录或目录失效时冻结受影响的执行，由用户明确关联原工作，核对后再继续。已有任务不随目录变更自动搬迁。

30. **跨工作区注意力。** 普通 Space 只显示一个 Matt 待处理汇总标记，点击返回对应 Matt 工作区处理。待停用的遗留问题仍保留，进入时先给出恢复入口。后台事件不得抢焦点、切换 Space/layout、打开向导或注入正在输入的对话。详细决策、验收及操作仍归属原 Effort。

## Testing Decisions

测试边界已由用户确认：**主要从真实 Herdr 界面进行端到端验证**。可控运行器用于重现故障，不替代真实运行器的主流程验收。

1. **最高边界。** 从用户在 Herdr 原生界面中的操作开始，观察原生视图、实际会话/终端、工作目录、持久记录以及最终代码与验证结果。主要验收不通过 HTML、直接调用原型 reducer 或直接修改内部状态完成。
2. **测试对象。** 验证原生 layout/mods、对话与操作流、操作桥和持久化、唯一调度者、Codex/Claude Code 接入、git/验证证据联动的整体行为。只在同一端到端边界无法稳定构造协议故障时，为操作契约补少量定向测试；不为每个内部模块新增独立测试面。
3. **好测试的标准。** 给定真实输入、依赖和代码版本，检查用户可见结果及实际副作用；验证“不重复启动、不误放行、不丢决策、不用旧证据通过验收”。不锁定 reducer 分支、私有存储格式、组件树或调用次数。
4. **先例与当前缺口。** 当前目录只有设计和一次性原型，没有生产应用、自动化测试框架或既有原生端到端测试。E 的五个引导演练和边界检查是场景先例，不是生产测试。Herdr mod 初始校验和原生截图可辅助检查，但编译、初始渲染或截图本身不能证明异步交互与恢复正确。
5. **真实运行器要求。** 使用受控的小型代码工作区、明确的验证命令和可判定输出，分别完成 Codex 与 Claude Code 的真实启动、对话、暂停/继续和结果返回；完成至少一次双运行器并行与明确交接。确认请求配置与实际会话配置。账号、额度或安装条件不满足时记录为未验收，不将替身测试写成真实接入通过。
6. **故障注入。** 从真实 Herdr UI 发起操作，在运行器边界可控地延迟、丢弃、重复或乱序投递回执，断开连接或终止操作进程/CoS。允许替身生成可复现的故障，但仍从真实 UI 操作并检查持久记录和运行数量；不能只验证 reducer。
7. **隔离与恢复。** 故障场景使用可丢弃代码工作区和专用应用测试环境。重载必须实际重建 mod/UI，恢复场景必须实际重启相关进程并读取磁盘记录，不能用“重置”或内存快照替代。观察 Worker 存活与认领变化，确认没有额外运行或提交。
8. **原生 UI 检查。** 在真实 Herdr 中验证正常宽度与窄窗口、抽屉开关、焦点与键盘导航、列表及异步更新。浏览器中的手机尺寸结果只作设计参考；本 spec 不宣称提供移动端 Herdr 应用。

主要验收场景：

| 场景 | 用户操作与外部条件 | 必须观察到的结果 |
|---|---|---|
| 一个小修复 | 对话发出明确小任务，沿用默认 | 一位真实 Worker、正确工作目录、真实 diff 与验证证据；无需配置完整编队 |
| 并行依赖 | A/B 同时执行，C 依赖二者 | 独立 worktree；Agent done 或只验收 A 均不启动 C；满足完成条件后 C 使用已验证基线 |
| 输入与通知 | 输入草稿、选择文字时连续到达后台事件 | 文本、焦点、选区保留；消息边界独立；阅读视图不被抢占 |
| 回复中的新输入 | 第一条消息仍在回复时再发一条 | 新输入按序排队；不会和事件拼为同一消息；明确中断另有回执 |
| 人工决策 | Worker 提问，Agent 读取事件后用户再回答 | 未回答事项仍可见；回答有来源记录；无关工作继续 |
| 纠偏与重复纠偏 | 指定 B 修改约束，随后再次修改 | A 保留现场；B 确认暂停及采用后继续原运行；每轮有独立可追踪请求 |
| 完整影响范围 | 变更影响多层依赖或既有未合入结果 | 全部相关旧结果失效，不只处理直接依赖；已集成事实不被倒退 |
| 旧证据拦截 | 纠偏尚未落实时对旧结果尝试重新验证或验收 | 不获得验收资格；新约束执行并返回适用证据后才可通过 |
| 返修与手动修改 | 对固定版本留下位置反馈，再返修或手动改代码 | 反馈绑定原版本；旧验收失效；新版本重新检查与人工验收后关闭反馈 |
| 候选失败或过期 | 已验收结果准备集成后，验证失败或基线变化 | 原基线不变；下游等待；新候选须按当前版本重新验证 |
| 回执丢失 | 操作已执行但确认未到达，实际重载 UI 并恢复 | 草稿/问题/请求保留；先查原请求；不产生第二次启动或第二认领 |
| CoS/桥进程重启 | Worker 活着时终止 CoS 或操作进程再恢复 | 从磁盘与运行事实恢复；不把聊天记忆当状态；不重复派工 |
| 配额与网络故障 | Worker 因额度或连接失败阻塞 | 原会话与工作目录保留；原因和恢复操作明确；没有虚假的成功 |
| 配置与能力 | 改默认、切作用域、保存，并启动下一任务 | 草稿保留；当前运行不变；新任务正确采用配置；不可用配置不被静默替换 |
| 跨运行器交接 | 为暂停任务选择另一运行器并交接 | 旧运行可追踪且不再有执行认领；新运行带上约束与现场；目标未被切错 |
| Effort 隔离 | 对不同 Effort 留草稿并切换，期间到达旧响应 | 草稿、焦点、阅读位置和待办各自恢复；旧异步结果不覆盖当前工作 |
| 原生可用性 | 真实 Herdr 窄窗口、键盘打开/关闭抽屉 | 主操作和输入可达；焦点正确；细节不撑宽主区；状态有文字说明 |
| 完成语义 | 所有任务本地验收与集成通过 | 只声明本项目已达成的完成条件；不显示未实际运行的 PR、CI 或发布成功 |
| 工作流独立选择 | 同仓库两个 worktree 分别启用与不启用 Matt，终端切换 cwd | 配置相互独立；普通 Space 无新增 Agent；派工仍指向固定绑定目录 |
| 显式布局切换 | 进入 Matt 后选择普通 Space，再返回原工作台 | 普通终端可用，Matt 面板隐藏；Space 切换不触发全窗口 layout 替换 |
| 重复入口 | 同机器同固定目录由第二个 Space 启用 | 定位已有入口，无第二套调度与重复运行；Worker worktree 保持原归属 |
| 停用收尾 | 运行 A/B 时停用，完成验证、返修、验收及已授权集成 | A/B 现场保留，后续 C 不启动；现有工作全部收尾后才完成停用 |
| 暂停或取消停用 | 收尾中请求暂停，延迟回执；另一次取消停用 | 回执前不显示已停用；确认后保留现场；取消停用恢复原调度，不重复认领 |
| 重新启用 | 保留未决问题停用，期间改动代码，再重新启用 | 先核对原运行与产物；用户明确继续；问题保留、失效验收不复用、无重复启动 |
| 绑定变化 | 改名、删除重建 Space 或使固定目录失效 | 改名继续关联；其余情形冻结受影响执行，明确重新关联并核对，不搬迁已有任务 |
| 普通 Space 后台提醒 | 编辑普通终端时 Matt 产生决策或验收请求 | 仅汇总标记变化；焦点、布局、输入不变；点击后进入原工作处理 |

## Out of Scope

- 为非 Matt Workspace 建设通用对话 Agent、调度或验收平台，及自动切换全窗口布局。
- 托管多人协作、远端机器调度、多仓库工作流及跨设备同步。
- 全平台 tracker 适配器。先遵循项目明确配置的一个 tracker；本 spec 的发布位置与产品的工作状态来源是不同配置，不能互相推断。
- 重写 Matt skills、增加第二个调度器，或让每项工作强制走完整 SDLC。
- 生产浏览器版或移动端 Herdr。HTML 原型只保留为设计证据。
- 自动模型排行榜、付费 benchmark、费用推算、token 遥测或上下文质量仪表盘。
- 未经明确选择的自动模型升级、自动跨运行器交接，或自动启用额外委派工作流。
- GitHub PR、外部 CI、推送、部署和发布自动化。首版真实完成条件为本地验收与集成验证。
- 根据 Agent done、通知已读、聊天措辞或固定命名猜测阶段、任务完成、验收或运行状态。
- 把原型内存恢复演练视为生产持久化，或把替身结果视为真实 Agent 验收。

## Further Notes

### 发布状态

本项目使用 `zengjie/herdr-matt-workbench` 的 GitHub Issues，triage 标签采用 canonical role 同名映射。该 Spec 发布时使用 `ready-for-agent`，无需重新 triage。本地文件保留为可随仓库阅读的快照；后续变更同步已发布 issue。

本轮尚未拆 Implementation Tickets，也没有安装、启动或修改 Herdr 的生产 mods。

### 已确认的测试选择

用户明确选择“主要从真实 Herdr 界面端到端验证”。此选择替代此前“主要从工作区操作接口测试、原生界面少量冒烟”的建议，后续拆票和验收必须保留。

### 设计来源与优先顺序

- [统一工作区 v0.4 基线](./2026-11-04-unified-workspace-baseline.md)：当前产品行为、责任和范围依据。
- [工作流按工作区启用 ADR](../adr/0001-opt-in-matt-workflow.md)：工作流选择、窗口布局、收尾恢复及入口归属。
- [E 可交互原型](https://github.com/zengjie/herdr-matt-workbench/blob/prototype/unified-workspace/docs/design/matt-workbench-prototype.html?variant=E)：统一状态、按需展示、纠偏、返修、恢复与轻量路径的交互依据；不作为生产实现。
- [日常开发审查](./2026-11-04-daily-development-review.md)：当前改进所回应的问题与验证场景。
- [角色与 Agent 选择研究](./2026-11-04-agent-role-selection.md)：候选配置和运行器能力研究快照，不是永久模型目录或账号授权。
- [系统设计与 Matt SDLC 依据](./2026-11-04-matt-herdr-system-design.md)：SDLC、平台接口、领域语义与历史推导；其中早期只读方案已被 v0.3 替代。
- [项目领域术语](../../GLOSSARY.md)：Effort、Implementation Ticket、Decision Ticket、Frontier、Session、Phase、Phase Boundary、Context Pointer 与 Stage Profile 的含义。

本次引用原型的 SHA-256 为 `c533e0a3b818e48c26c0bd82a24eb8313b115e592b008b9b5d300f932a5da189`。原型保存在本仓库 `prototype/unified-workspace` 分支。后续实现可以改变内部结构，但必须保持本 spec 声明的用户行为与操作约束。
