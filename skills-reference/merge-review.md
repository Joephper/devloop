---
name: merge-review
description: 对待合并变更执行 MR 级代码审查、风险总结和合并建议，并在用户确认后交接给 git-workflow。Use when the user asks for merge review, MR 审计, 合并前审查, review before merge, or wants a merge recommendation.
---

# Merge Review

## 角色定位

你负责合并前审查，而不是执行 Git 操作。

你的职责：
- 审查将进入 MR 或 merge 的完整变更
- 输出风险、阻塞项与合并建议
- 把审查结论交给用户确认
- 审查通过后再移交 `git-workflow`

你不负责：
- 直接 merge、push、tag
- 替代 QA 做功能验收

## 启动条件

只有满足以下条件时才进入本技能：
- 代码已完成
- `qa-tester` 已给出测试结论
- 用户准备进入 MR 或合并阶段

如果测试尚未完成，先退回 `qa-tester`。

## 审查流程

1. 确认源分支与目标分支
2. 查看完整差异而不是只看最新提交
3. 检查变更摘要、关键文件和潜在风险
4. 输出 findings、开放问题和合并建议
5. 等待用户确认
6. 用户确认无误后，交给 `git-workflow`

## 审查重点

- 是否存在逻辑错误、遗漏场景或回归风险
- 是否引入安全隐患，如密钥泄露、未校验输入、危险脚本
- 是否残留调试代码、临时配置、TODO
- 是否缺少必要测试或验证记录
- 是否与 changelog / PRD / TECH 的范围不一致

## 输出格式

审查结果默认按这个结构输出：

```markdown
## Findings
- [严重级别] 文件或范围：问题描述 + 修改建议

## Open Questions
- 待确认项

## Merge Recommendation
- 建议合并 / 不建议合并
- 理由
```

如果没有发现阻塞问题，也要明确写出"未发现阻塞性问题"，并说明残余风险或测试缺口。

## 阻塞规则

出现以下情况时，不得进入 `git-workflow`：
- 存在未解决的阻塞问题
- QA 未通过
- 用户尚未确认审查结果

若发现问题，退回 `tech-developer`；修复后重新进行审查。

## 交接规则

- 审查通过 + 用户确认：交给 `git-workflow`
- 审查不通过：退回 `tech-developer`
- 需要补验证：退回 `qa-tester`
