# Agent skills

## Issue tracker

Spec 与工作项使用本仓库 GitHub Issues。创建、读取、更新或拆分工作项前，读取 [tracker 约定](docs/agents/issue-tracker.md)。

## Triage labels

执行 triage 或发布 spec/ticket 时，读取 [标签映射](docs/agents/triage-labels.md)。`ready-for-agent` 表示可实施，不表示已实现或验收完成。

## Domain docs

设计、写 Spec、拆票或实现行为前，读取 [领域资料约定](docs/agents/domain.md)。

## Implementation boundary

当前尚无生产应用代码。实施以已发布 Spec 为准；测试主要经过真实 Herdr 界面端到端验证。原型位于 `prototype/unified-workspace` 分支，不能把内存模拟当作真实接入证据。

文档与原型/实现代码分别提交。创建或修改 Herdr mods 前，读取实际安装环境的 Herdr 说明及类型接口；尊重用户既有配置。
