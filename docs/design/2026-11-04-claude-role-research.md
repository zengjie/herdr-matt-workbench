# Claude role and runtime research — 2026-11-04

Scope: official documentation supporting model selection for a dedicated dialogue session and a chief-of-staff scheduler, worker, and reviewer. Research only; no configuration, credentials, or application implementation changed.

Evidence date: 2026-11-04, live official documentation accessed that day. Parent reports local Claude Code 2.1.318 and `--help` effort values; no inference request or account-entitlement check was performed. SDK package version was not inspected. Documentation can advance independently of the installed CLI.

## Current models and identifiers

| Current family | Canonical Claude API ID | API default effort | Code default effort |
| --- | --- | --- | --- |
| Fable 5.1 | `claude-fable-5-1` | high | high |
| Opus 5.5 | `claude-opus-5-5` | medium | medium |
| Sonnet 5.5 | `claude-sonnet-5-5` | high | medium |
| Haiku 5.5 | `claude-haiku-5-5` | medium | medium |

IDs and API defaults: [Fable overview](https://platform.claude.com/docs/en/models/fable-5-1/overview), [Opus overview](https://platform.claude.com/docs/en/models/opus-5-5/overview), [Sonnet overview](https://platform.claude.com/docs/en/models/sonnet-5-5/overview), [Haiku overview](https://platform.claude.com/docs/en/models/haiku-5-5/overview). Code defaults: [model configuration](https://code.claude.com/docs/en/model-config#adjust-effort-level).

The dateless IDs above are fixed model snapshots, unlike evergreen family aliases. Serving infrastructure can still change observable behavior. Record both requested alias and actual resolved model in evaluations. [Model IDs and versioning](https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions).

## Claude Code selection and effort

`--model` selects a session model; `/model` changes it interactively. Anthropic API aliases `opus`, `sonnet`, and `haiku` resolve to the corresponding 5.5 release; `fable` resolves to Fable 5.1 except Claude apps gateway sessions, where it resolves to Fable 5. `best` follows Fable when available, otherwise Opus. Other providers can resolve family aliases to older models. `opusplan` means Opus in plan mode and Sonnet in execution.

All four current families support `low`, `medium`, `high`, `xhigh`, `max`. Opus/Sonnet 4.6 omit `xhigh`; unsupported levels clamp downward. Model or organization caps can further lower applied effort. `--effort` is session-scoped; `max` is not persistable in `effortLevel` or `modelSettings`. `ultracode` is a workflow setting, not another model effort level. Minimum Code versions: Fable 5.1 2.1.257, Opus 5.5 2.1.280, Sonnet 5.5 2.1.284, Haiku 5.5 2.1.293. [Model configuration](https://code.claude.com/docs/en/model-config).

## Subagents and independent sessions

Subagent frontmatter accepts aliases, full IDs, or `inherit`; Agent-tool invocation accepts family aliases only. Invocation model overrides definition, then environment default, then parent model. A family alias can reuse the parent's exact version in that family. `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` changes this behavior, so inspect effective runtime metadata.

Set `effort` explicitly for each role. Invocation effort overrides definition effort (requires Code 2.1.292); `CLAUDE_CODE_EFFORT_LEVEL` overrides both, and caps still apply. Without explicit effort, a subagent on the parent's model inherits session-selected effort; a different-model subagent does not. A subagent has separate context and permissions, but remains part of its parent session. [Create custom subagents](https://code.claude.com/docs/en/sub-agents).

## API/backend distinctions

Messages API uses `output_config.effort`; Agent SDK uses the `effort` option. Top-level API effort needs no beta header. Per-message effort is beta, with different model/provider support; Bedrock requires `mid-conversation-output-config-2026-07-01` and does not support every combination. Do not assume that API beta is the same as the CLI setting.

Opus 5.5 cannot disable thinking. Sonnet 5.5's `between_tools` and Haiku 5.5's disabled thinking work only through `high`; `xhigh`/`max` require adaptive thinking or return 400. Sonnet API defaults high, whereas Code defaults medium. Effort labels are not a common measurement scale across model families. [Effort API documentation](https://platform.claude.com/docs/en/build-with-claude/effort).

Claude Code subscription allocation and Console/API billing are distinct; Pro, Max, and Ultra include Code access but have usage limits. No assumption is made about this user's plan. [Subscription and billing guide](https://support.claude.com/en/articles/11145838-use-claude-code-with-your-pro-max-or-ultra-plan). Enterprise model/effort restrictions are account-policy inputs, not product capability guarantees. [Enterprise consumption guide](https://support.claude.com/en/articles/14782391-claude-enterprise-consumption-guide).

## Structured messages, waiting, interruption, resume

Use CLI print mode `--input-format stream-json --output-format stream-json` for machine messaging, `--json-schema` for final structured output, and `--resume` for an explicit session. Streaming events and validated final output are separate facilities. [CLI reference](https://code.claude.com/docs/en/cli-reference).

SDK `outputFormat: {type: "json_schema", schema}` places validated data in result `structured_output`. Check both `subtype === "success"` and the presence of structured output; successful completion can still omit it. Validation does not establish factual correctness. [Structured output](https://code.claude.com/docs/en/agent-sdk/structured-outputs).

The main agent's `AskUserQuestion` triggers the application `canUseTool` callback; the application displays questions and returns answers. Permission requests and questions remain inside the running query. Crucially, `AskUserQuestion` is unavailable to Agent-tool subagents. [Handle approvals and user input](https://code.claude.com/docs/en/agent-sdk/user-input).

Sessions persist conversation history, not filesystem snapshots. Store explicit session IDs and use `resume` when multiple roles coexist: `continue` chooses the directory's most recent session. Fork creates independent conversation history. [Sessions](https://code.claude.com/docs/en/agent-sdk/sessions).

SDK query options accept named `effort` values. Streaming input permits `setModel()`, `applyFlagSettings()`, and `interrupt()`. Interrupt is not a durable pause: queued messages may still run; receipt capabilities and UUIDs matter. `applyFlagSettings` can adjust model/effort; system prompt changes require a new session. Python lacks that TypeScript method. These details require version matching against the actual SDK/CLI pair. [TypeScript reference](https://code.claude.com/docs/en/agent-sdk/typescript).

## Recommended role mapping — design hypotheses to evaluate

These are proposed starting points, not vendor benchmarks or measured local performance.

| Role | Initial candidate | Escalation / alternative |
| --- | --- | --- |
| Dedicated dialogue | Sonnet 5.5 medium; main session with question callback | Opus 5.5 medium for ambiguous synthesis; compare user-rated continuity and response time |
| CoS scheduler | Deterministic app queue/state plus Sonnet 5.5 medium for prioritization | Haiku 5.5 medium for classification only after labeled routing evals |
| Worker | Sonnet 5.5 medium on bounded tasks | Opus 5.5 medium/high for cross-cutting or failed tasks |
| Reviewer | Independent read-only Sonnet 5.5 high or Opus 5.5 medium/high | Compare defect recall, false positives, and evidence quality |
| Long-horizon escalation | Fable 5.1 high | Only when Opus higher-effort evaluations still miss requirements |

Anthropic positions Fable for demanding long-horizon work and explicitly suggests starting most workloads with Opus, escalating when evaluations remain insufficient. This supports treating Fable as an option, not mandatory scheduling infrastructure. [Fable overview](https://platform.claude.com/docs/en/models/fable-5-1/overview).

For pause/restart, the application should own durable task states such as waiting_for_user, running, blocked, completed; persist question IDs, answers, run IDs, and session IDs. A subagent should return a structured request for clarification to the dialogue/CoS layer. This is a design recommendation derived from the documented question and session boundaries, not a built-in durable scheduler guarantee.

Before choosing a default, evaluate representative user tasks across family and effort combinations with equal instructions and comparable budgets. Measure acceptance, correction rate, review findings, elapsed time, and spend. No evidence here establishes that Claude or Codex is inherently better for any role.
