---
name: auto-programming-workflow
description: 作为跨项目通用的自动化编程入口，按需求梳理、技术设计、并行开发、测试验收、合并审查、Git执行的顺序驱动整条交付链路。Use when the user mentions 自动化编程, 通用工作流, 新项目沿用流程, 从需求到上线, 全流程开发, or asks for a reusable delivery workflow.
---

# Auto Programming Workflow

## 目标

这是一个跨项目复用的总入口技能，用来在任何新项目里复用同一套自动化编程思想。

默认链路：

```text
product-manager
-> tech-developer
-> parallel-dev（按需）
-> qa-tester
-> merge-review
-> git-workflow
```

## 使用原则

- 优先复用个人技能，不依赖当前项目必须先存在 `.cursor/rules`
- 若项目已有自己的流程、目录、命名规范，优先沿用
- 若项目没有现成规范，默认按 `changelog -> PRD -> TECH -> QA -> review -> git` 建立流程

## 新项目启动方式

当用户在一个新项目里表达以下意图时，默认启用本技能：
- "按自动化编程流程来"
- "这个项目也沿用之前那套流程"
- "从需求梳理到合并帮我串起来"
- "新项目通用引入这套工作流"

然后执行：
1. 交给 `delivery-orchestrator` 判断当前阶段
2. 由对应技能完成当前阶段工作
3. 阶段完成后按门禁交棒，不跳步

## 可选增强

如果用户希望某个项目具备更稳定的本地记忆，可以再为该项目补充 `.cursor/rules/`：
- `workflow-stage-routing.mdc`
- `workflow-gates-and-handoffs.mdc`

但这些项目规则是可选增强，不是全局复用的唯一入口。
