---
title: Herdr 角色与 Agent 选型
subtitle: 官方资料研究与配置提案 · 2026-11-04 · 未修改运行配置
cols: 2
---
## A 角色与执行配置分开
每个角色都能选择 Codex 或 Claude Code。
模型和思考强度随运行器与账号能力变化。
角色定义职责、输入、输出与权限。
执行配置定义运行器、模型、思考强度与可用工具。
实际会话记录本次生效配置与来源。

同一角色可以有多个执行配置。
角色数量不等于常驻进程数量。
对话 Agent 保持独立会话，其他角色按需启动。
本报告给出初始候选，尚未开展本地模型对比。

## B 当前可选模型与真实能力
本机版本为 Codex CLI 0.166.0、Claude Code 2.1.318。
只读检查了 CLI 帮助与 Codex 模型缓存。
没有读取凭据，也没有发起模型推理请求。

| Codex 候选 | 官方定位与使用方向 |
|---|---|
| GPT-6.1 Sol | 复杂、反复执行的长任务；开发与协调候选 |
| GPT-6 Astra | 最困难的推理任务；架构与疑难诊断升级候选 |
| GPT-6.1 Terra | 视觉理解、电脑操作与清晰写作；界面验证候选 |
| GPT-6 Luna | 范围清楚的高频任务；检索与摘要候选 |

以上模型出现在本机缓存中。
缓存时间为 2026-11-04 07:51:54 UTC。
缓存不证明当前账号每次调用都可用。

[OpenAI Docs 模型说明](https://learn.chatgpt.com/docs/models)介绍了定位与选择方法。

| Claude Code 候选 | Claude API 完整模型 ID |
|---|---|
| Opus 5.5 | `claude-opus-5-5` |
| Sonnet 5.5 | `claude-sonnet-5-5` |
| Haiku 5.5 | `claude-haiku-5-5` |
| Fable 5.1 | `claude-fable-5-1` |

Claude 的当前系列支持 low、medium、high、xhigh、max。
这些名字不代表跨模型相同的计算量。
Opus、Sonnet、Haiku 5.5 在 Code 中默认 medium。
Sonnet 5.5 在 API 中默认 high，不能照搬到 Code。

Claude 家族别名随提供商变化。
Fable 的映射也受连接方式影响。
完整 ID 与家族别名应分别记录。

来源：[模型配置](https://code.claude.com/docs/en/model-config)、[Sonnet 5.5](https://platform.claude.com/docs/en/models/sonnet-5-5/overview)。

## C 各角色的候选组合
下表属于设计建议，不是跨厂商性能排名。
思考强度是初始设置，须经代表任务验证。

| 角色 | Codex 候选 | Claude Code 候选 | 为什么这样选 |
|---|---|---|---|
| 专门对话 | GPT-6.1 Sol · medium | Opus 5.5 · medium | 保持意图、识别歧义、转译操作；前台无需长期满强度 |
| 架构与 Spec | Astra · medium / high | Opus 5.5 · high | 处理跨任务约束与重要取舍 |
| CoS 调度 | GPT-6.1 Sol · medium | Sonnet 5.5 · medium | 解释依赖与异常；状态一致性由执行层保证 |
| 普通开发 | GPT-6.1 Sol · medium | Sonnet 5.5 · medium | 自包含 ticket，从平衡档起步 |
| 疑难诊断 | Astra · high | Opus 5.5 · high | 追踪跨模块因果；先要求可复现证据 |
| 独立评审 | GPT-6.1 Sol · high | Opus 5.5 · high | 追踪边界、核对标准与 spec；与作者分开会话 |
| 集成与冲突 | GPT-6.1 Sol · medium | Sonnet 5.5 · medium | 常规集成；语义冲突升级 high 或交还作者 |
| UI 行为验证 | GPT-6.1 Terra · medium | Sonnet 5.5 · medium | 需要真实浏览器工具和行为证据 |
| 检索与摘要 | Luna · high | Haiku 5.5 · medium | 范围明确、结果可核对的辅助工作 |

人工验收属于用户职责。
Agent 可整理材料、运行检查、解释差异。
更强模型不能代替用户表达通过。

Opus 在更高强度仍达不到标准时，可试 Fable 5.1。
[Fable 官方说明](https://platform.claude.com/docs/en/models/fable-5-1/overview)也建议先从 Opus 开始。

评审优先保持上下文独立并固定被审版本。
关键任务可增加另一家模型的第二轮评审。
跨模型复核是否值得，仍看有效发现与误报。

## D 第一套默认配置与升级规则
建议首轮采用以下均衡组合。
它用于比较体验，不锁定产品角色与厂商。

| 角色 | 首轮默认 |
|---|---|
| 对话 | Claude Code · Opus 5.5 · medium |
| CoS | Codex · GPT-6.1 Sol · medium |
| 开发 | Codex · GPT-6.1 Sol · medium |
| 评审 | Claude Code · Opus 5.5 · high |
| 集成 | Codex · GPT-6.1 Sol · medium |

对话若延迟过高，比较 Sonnet 5.5 medium。
开发失败时，先判断缺的是上下文还是推理深度。
缺少需求与证据时，补信息。
复杂推理失败时，再比较 high 或更强模型。
原有失败现场和验证命令必须保留。

Max 只用于已证明有收益的困难任务。
Codex Ultra 包含自动子 Agent 委派。
Claude Ultracode 属于动态工作流设置。
两者会改变执行组织，应与思考强度分开显示。
首轮不默认启用，避免形成额外调度入口。

来源：[Codex Max 与 Ultra](https://learn.chatgpt.com/docs/models#know-when-to-use-max-or-ultra)、[Claude 工作流与 effort](https://code.claude.com/docs/en/model-config#adjust-effort-level)。

## E Herdr 中如何让用户选择
角色配置页按行展示职责与执行配置。
选中角色后依次选择运行器、模型和思考强度。
工具与权限另行设置，不随模型强弱扩大。

| 层级 | 生效规则提案 |
|---|---|
| 工作区角色默认 | 新 effort 的起点 |
| 当前 effort 覆盖 | 适应该项目的工作方式 |
| 单次任务覆盖 | 为某个 ticket 选择另一配置 |
| 本次实际配置 | 启动时固定，并记录运行侧回报 |

覆盖顺序为：单次任务、effort、工作区。
界面同时显示期望配置与实际生效配置。
不支持的模型或强度应明确提示。
运行器自动降档时，显示原因和实际档位。
不可只显示启动参数，假定它已经生效。

修改默认配置只影响后续启动。
同一运行器内切换，可在支持的轮次边界应用。
运行中任务不应因修改默认值而悄然改变。
跨运行器切换需要新会话和明确交接。
保留目标、决策、未决问题、版本与请求回执。
不能把 Codex 会话 ID 当作 Claude 会话 ID 使用。

## F 接入方式影响可选性
Codex App Server 提供 model/list 与可用 effort 列表。
它也提供轮次启动、steer、interrupt 和事件通知。
应使用实际返回能力生成选择器。
中断回执不等于代码回滚或所有后台进程停止。

来源：[Codex App Server](https://learn.chatgpt.com/docs/app-server)。

Claude Code 支持会话级 model 与 effort 设置。
原生子 Agent 也能配置 model 与 effort。
环境变量、组织限制和运行方式会影响最终设置。

来源：[Claude 子 Agent](https://code.claude.com/docs/en/sub-agents)。

Claude Agent SDK 支持模型发现与会话控制接口。
AskUserQuestion 不提供给 Agent 工具子 Agent。
因此专门对话角色宜使用独立主会话。
后台角色的问题经应用层转交对话入口。

来源：[SDK 接口](https://code.claude.com/docs/en/agent-sdk/typescript)、[用户输入](https://code.claude.com/docs/en/agent-sdk/user-input)。

混用两家的 Worker，需要宿主层启动与管理各自会话。
不能假定原生子 Agent 工具能直接启动另一家运行器。
事件收件箱、消息编号和草稿隔离仍由 Herdr 保证。
更换模型无法修复通知拼进人工输入的问题。
这是基于接口边界提出的设计判断。

## G 用真实工作决定最终默认值
先固定输入、工具、验收条件和被审版本。
比较模型时保持强度不变，再单独比较强度。
相同档位名称不保证等量计算与相同速度。

| 角色 | 代表任务 | 主要观察 |
|---|---|---|
| 对话 | 用户打字时收到完成通知，同时纠正 B | 草稿隔离、意图保留、对象准确、回应耗时 |
| CoS | A/B 并行，C 等待；B 暂停后重启 | 重复执行、依赖误放行、恢复正确性 |
| 开发 | 同一自包含 ticket、相同代码基线 | 验收结果、返工次数、总耗时 |
| 评审 | 含已知缺陷和正确改动的同一 diff | 有效发现、误报、复现证据 |
| 验证 | 同一 UI 行为与截图要求 | 实际行为、报告是否夸大 |

记录完成到验收的总耗时，而非只看首字速度。
同时记录消耗、用户纠正次数与失败恢复情况。
重复覆盖代表任务，避免以一次结果决定默认值。
[OpenAI 模型选择指南](https://developers.openai.com/api/docs/guides/model-selection)也要求按任务验收比较。

API 价格不等于 Codex 或 Claude Code 订阅消耗。
本轮不推算用户成本，也没有测试账号额度。
比较时记录所用认证方式与产品计费口径。

## H 已核实与尚待验证
已核实官方能力、CLI 版本与本机 Codex 缓存。
未安装或更新运行器，未更改配置。
未启动真实 Worker，未运行付费模型对比。
目前原型仍使用模拟 Agent 和状态。

下一版原型可增加角色配置矩阵。
再演练配置覆盖、不可用提示与会话交接。
接入真实运行器后，核对能力发现与生效设置。
这些工作不属于本轮已完成内容。

更细的 Claude 接口事实见同目录研究记录：
[Claude 角色研究](./2026-11-04-claude-role-research.md)。
