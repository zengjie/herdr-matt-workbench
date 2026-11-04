# Issue tracker: GitHub

仓库为 `zengjie/herdr-matt-workbench`，地址 https://github.com/zengjie/herdr-matt-workbench 。Spec 和 Implementation Tickets 以该仓库的 GitHub Issues 为权威记录。使用 `gh` CLI，写操作显式指定 `--repo zengjie/herdr-matt-workbench`。

## 发布与读取

- 发布 Spec：创建一个 issue，正文采用 to-spec 模板，添加 `ready-for-agent`。多行正文使用临时文件与 `--body-file`。
- 读取工作项：使用 `gh issue view`，读取完整正文、标签和评论。
- 更新已发布 Spec：编辑同一个 issue，并同步仓库中的 Spec 快照；避免创建重复 Spec。
- 本地设计文档是来源和快照，不是另一个任务调度器。

## 任务与依赖

- 拆票时每个行为切片创建独立 issue，并关联父 Spec。
- 优先使用 GitHub 原生 sub-issues 与 blocking dependencies；当前 CLI 不支持时使用公开 API。
- 原生关系不可用时，在正文保留 `Part of #N` 和 `Blocked by: #N`。
- 开始任务前核对实际开放 blockers，并记录认领。GitHub assignee 仅表示认领记录；运行时的排他执行仍需操作契约保证。
- 发布或更新前核对目标仓库，避免写到技能的上游仓库。

## 完成与评论

评论保留决策、验证证据和上下文指针。关闭实现工作项前确认该项声明的完成条件，不将 Agent done 或通知已读当成完成。

## Pull requests as a triage surface

PRs as a request surface: no.
