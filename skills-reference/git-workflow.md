---
name: git-workflow
description: Git 分支管理、安全合并、标签发布与冲突处理。Use when the user mentions git, 分支, branch, merge, 合并执行, MR 创建, tag, 发布, push, pull, checkout, rebase, cherry-pick, or other Git execution tasks after review approval.
---

# Git Workflow

## 职责边界

此技能是研发流程的 Git 执行层，只负责：
- 分支策略与命名约束
- 受控的 merge / tag / push / pull / branch 操作
- 并行开发所需的 `git worktree` 创建与清理
- Merge Request 链接生成
- 合并冲突处理

此技能不负责：
- 需求梳理
- TECH 方案设计
- 代码开发
- QA 验证
- MR 审查结论与代码评审总结

如果用户要做变更审查、风险总结、合并建议，先交给 `merge-review`，只有在用户确认审查结果后，才进入本技能。

## 上游门禁

执行高风险 Git 操作前，先检查是否满足上游条件：
- 合并到 `pre`、`master` 或创建发布标签前，必须已经完成 `merge-review`
- 进入正式合并前，必须已有用户明确确认
- 若测试未通过、审查未完成或用户未确认，拒绝继续并提示回到上游阶段

对于并行开发前置准备，也要检查：
- 若 `parallel-dev` 需要独立工作目录，但目标分支尚未创建对应 `git worktree`，先由本技能完成分支与 worktree 准备
- worktree 准备完成后，再交回 `parallel-dev`

## 分支策略与发布流水线

| 分支 | 用途 | 对应环境 | 合并来源限制 |
|------|------|----------|--------------|
| `master` | 生产分支 | 生产环境（打 tag 后触发） | **只能从 `pre` 合并** |
| `pre` | 预发布分支 | 预发布环境 | feature / hotfix / dev 均可合并，需确认 |
| `dev` | 开发分支 | 测试环境 | feature / hotfix 可合并 |
| `feature-*` | 功能分支 | 本地 / 测试 | 自由创建 |
| `hotfix-*` | 紧急修复 | 本地 / 测试 | 从 master 拉出 |

说明：当某个功能需要优先上线时，可以将对应 `feature-*` 直接合并到 `pre`，但仍要先完成 QA、审查和用户确认。

### 分支命名规范

- 功能分支：`feature-模块名`
- 子模块分支：`feature-大模块-子模块`
- 紧急修复：`hotfix-描述`

## 合并安全规则

### feature / hotfix -> dev

可正常合并。完成后用 `git status` 和必要的 `git branch` 命令确认结果。

### feature / hotfix / dev -> pre

必须先确认用户同意：

> "即将把 `{source}` 合并到预发布分支 `pre`，合并后会部署到预发布环境。是否继续？"

若用户尚未确认审查结果或未给出测试通过结论，停止并提示先执行 `merge-review`。

### pre -> master

这是生产发布前的最高风险操作，必须同时满足：
1. 来源校验：只允许 `pre` 合并到 `master`
2. 权限校验：用当前 Git 远程与账号做实时检查，不读取缓存
3. 用户二次确认：
   > "即将把 `pre` 合并到生产分支 `master`，这是生产发布操作。是否继续？"

若来源不是 `pre`，必须拒绝：
> "不允许从 `{source}` 直接合并到 `master`。代码必须先合并到 `pre` 经预发布验证后，再从 `pre` 合并到 `master`。"

### master 打 tag

1. 确认当前在 `master`
2. 确认用户同意：
   > "即将在 master 上创建标签 `{tag_name}` 并推送到远程，推送后会触发生产环境部署。确认继续？"
3. 执行 `git tag` 与 `git push origin`

### 禁止的操作

- 禁止任何分支直接合并到 `master`
- 禁止对 `master` 执行 force push
- 禁止自动解决冲突后静默提交

## 合并冲突处理

检测到冲突时：
1. 用 `git diff --name-only --diff-filter=U` 列出冲突文件
2. 说明每个文件的冲突位置与双方改动
3. 引导用户逐个选择：保留当前 / 保留对方 / 手动编辑
4. 绝不自动完成冲突解决并直接提交
5. 全部解决后，提示用户确认下一步提交或继续合并

## Git 元数据读取

查询分支、标签、远程、权限或当前状态时，直接执行 Git 命令获取实时结果，不读取、不创建、不刷新 `.shared_cache/git/` 下的缓存文件。

如果仓库中已经存在历史 `.shared_cache/git/` 文件，视为项目本地遗留数据；除非用户明确要求清理，不要修改这些文件。

## GitLab MR 工作流

当用户需要创建 Merge Request 时：
1. 确认源分支与目标分支
2. 用 `git remote get-url origin` 获取 GitLab 地址
3. 构造 MR URL：`{gitlab_url}/-/merge_requests/new?merge_request[source_branch]={source}&merge_request[target_branch]={target}`
4. 将链接交给用户在浏览器中操作

## Parallel Dev 前置准备

当 `parallel-dev` 需要并行工作目录时，本技能负责 Git 侧准备：
1. 确认大模块分支或子模块分支存在，不存在则先创建
2. 为目标分支创建对应 `git worktree`
3. 如项目使用端口注册表或其他共享数据，再交回 `parallel-dev` 继续分配

如果用户要开始并行开发，但仓库还没有 worktree，不要直接进入 `parallel-dev`，应先执行本技能。

## 结束后的交接

完成 Git 操作后：
1. 向用户说明结果与当前分支状态
2. 若只是完成 MR 创建，则等待用户后续审阅或合并决定
