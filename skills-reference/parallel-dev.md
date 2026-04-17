---
name: parallel-dev
description: 并行开发调度：识别可拆分模块、创建子模块分支或 worktree、协调多个 tech-developer 并行实现并在完成后汇总联调。Use when the user mentions 并行开发, parallel development, 多模块同时开发, worktree, 子模块拆分, 多代理协作, or multi-service development.
---

# Parallel Development

## 角色定位

你负责把一个可拆分的大需求拆成多个并行开发流，并在结束后汇总。

你的职责：
- 判断是否适合并行开发
- 规划子模块边界与交付契约
- 为每个子模块准备分支 / worktree / 端口
- 协调多个 `tech-developer` 并行实现
- 汇总结果并组织联调
- 联调完成后交给 `qa-tester`

你不负责最终合并到正式分支，也不替代 QA。

## 触发条件

只有在 `TECH` 已确认后，且满足以下任一条件时才启用本技能：
- 需求可拆成多个独立模块
- 改动横跨多个目录或子系统
- 需要多个开发代理并行推进

若模块强耦合、拆分成本高或用户明确要求串行实现，则不要触发并行开发。

进入本技能前，还要检查 Git 前置条件：
- 已有大模块分支或可作为并行基线的目标分支
- 若需要独立工作目录，对应 `git worktree` 已准备完成

如果缺少 worktree，不要在本技能内直接跳过处理；先交给 `git-workflow` 创建，再回到本技能继续。

## 并行拆分流程

1. 先列出子模块及边界
2. 为每个子模块定义交付契约：
   - 目标
   - 影响目录
   - 依赖关系
   - 完成标准
3. 判断哪些子模块可以真正并行，哪些必须串行依赖
4. 为每个可并行子模块分配一个 `tech-developer` 执行流
5. 收集各流结果，在主分支或总分支上做联调
6. 联调通过后交给 `qa-tester`

## 子模块分支策略

命名规范：

```text
feature-{大模块}
feature-{大模块}-{子模块}
```

合并路径：

```text
子模块分支 -> feature-大模块 -> dev -> pre -> master
```

创建子模块分支时：
1. 确认大模块分支存在
2. 若大模块分支或子模块分支尚未准备好，先交给 `git-workflow`
3. 从大模块分支切出子模块分支
4. 刷新 Git 分支缓存
5. 如需独立运行环境，确认 `git-worktree` 已由 `git-workflow` 创建，再继续端口与环境分配

## Worktree 与端口管理

### 端口注册表

使用 `.shared_cache/shared-data/port-registry.json` 管理端口：

```json
{
  "base_port": 8000,
  "registry": {
    "dev": { "port": 8000, "status": "active" },
    "feature-auth": { "port": 8001, "status": "active" }
  },
  "last_updated": "2026-03-18T10:00:00+00:00"
}
```

规则：
- `dev` 固定使用 `base_port`
- feature 分支从 `base_port + 1` 递增分配
- 删除分支后将状态改为 `idle`
- 每次变更端口注册表后更新 `last_updated`

### 并行环境创建

当子模块需要独立运行时：
1. 先调用 `git-workflow` 创建分支与 worktree
2. 确认 worktree 可用
3. 分配端口并写入注册表
4. 告知对应开发流使用该 worktree 与端口

硬性约束：
- 没有 `git worktree` 时，不得启动并行子开发流
- `parallel-dev` 负责协调，不负责替代 `git-workflow` 执行 Git 准备动作

## 汇总联调

每个子开发流完成后，至少检查：
- 是否满足子模块完成标准
- 是否影响其他子模块接口
- 是否需要补公共代码或集成修复

联调阶段要统一解决：
- 共享接口冲突
- 配置不一致
- 集成回归问题

联调完成后，明确交给 `qa-tester`，不得直接进入合并阶段。

## 回退规则

- 任一子流阻塞：记录阻塞点，必要时暂停其他依赖子流
- 联调失败：退回对应子流修复
- QA 失败：退回相关子流或主开发流

## 共享数据

跨分支共享的非 Git 跟踪数据统一放在 `.shared_cache/shared-data/`，例如：
- `port-registry.json`
- 其他联调用共享配置
