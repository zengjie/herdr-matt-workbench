# Herdr Matt Workbench

围绕 Matt skills 的 SDLC，设计一个简单的 Herdr 原生开发工作区：持续对话、并行任务、人工纠偏、验收返修与可靠恢复。

目前处于设计与原型阶段，尚未接入真实 Herdr mods、Agent、持久化或 CI。

## 从这里开始

- [实施 Spec](docs/design/2026-11-04-herdr-unified-workspace-spec.md)：60 条用户故事、实现决策与验收场景。
- [当前设计基线](docs/design/2026-11-04-unified-workspace-baseline.md)：统一工作区 v0.3。
- [领域术语](GLOSSARY.md)：工作项、上下文与证据的共同语言。
- [GitHub Issues](https://github.com/zengjie/herdr-matt-workbench/issues)：Spec 与后续 Implementation Tickets 的权威记录。

主要验收边界已确认：以真实 Herdr 界面端到端验证为主。

## 交互原型

[查看原型源码](https://github.com/zengjie/herdr-matt-workbench/blob/prototype/unified-workspace/docs/design/matt-workbench-prototype.html)。下载该 HTML 并在浏览器打开即可，无依赖、无服务要求。默认进入 E · 简洁工作区；A–D 保留为历史比较。

原型保存在 `prototype/unified-workspace` 分支，采用内存模拟；它不执行开发命令，也不代表已完成生产接入。`main` 保存 Spec、设计记录和项目约定。

## 项目约定

[Agent 入口](AGENTS.md)说明 tracker、标签和领域资料的读取方式。首个真实接入范围为本机、单仓库、单 Effort；详细功能与范围以 Spec 为准。
