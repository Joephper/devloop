# Skills 参考（完整文本）

> 八个核心 Skill 的完整原文，作为参考与学习材料。

---

## 安装位置

所有 Skill 放在全局用户目录，跨项目共享：

- **Cursor / Claude Desktop**：`~/.claude/skills/<skill-name>/SKILL.md`
- **Windows**：`C:\Users\<user>\.claude\skills\<skill-name>\SKILL.md`

每个 Skill 一个目录，目录名 = Skill 名。

---

## 文件清单

| 文件 | 作用 | 触发场景 |
|------|------|---------|
| [auto-programming-workflow.md](./auto-programming-workflow.md) | 跨项目自动化编程总入口 | "按流程来" / "新项目也用这套" |
| [delivery-orchestrator.md](./delivery-orchestrator.md) | 总控编排，路由分发 | "从需求到合并帮我串起来" |
| [product-manager.md](./product-manager.md) | 产品经理：需求梳理 + PRD | "梳理需求" / "写 PRD" |
| [tech-developer.md](./tech-developer.md) | 技术开发：TECH 方案 + 编码 | "TECH 确认了，开始写代码" |
| [parallel-dev.md](./parallel-dev.md) | 并行开发：拆分 + worktree + 联调 | "多个模块能不能并行做" |
| [qa-tester.md](./qa-tester.md) | 测试工程师：用例 + Bug + 报告 | "测试用例 / 测试报告" |
| [merge-review.md](./merge-review.md) | 合并前审查：Findings + 建议 | "合并前 review" |
| [git-workflow.md](./git-workflow.md) | Git 执行：分支 / 合并 / tag | "创建分支 / 打 tag / 合并" |

---

## 安装脚本（Windows PowerShell）

```powershell
# 假设这些文件已经在当前目录
$skillsRoot = "$env:USERPROFILE\.claude\skills"
$skillNames = @(
  "auto-programming-workflow",
  "delivery-orchestrator",
  "product-manager",
  "tech-developer",
  "parallel-dev",
  "qa-tester",
  "merge-review",
  "git-workflow"
)

foreach ($name in $skillNames) {
  $target = Join-Path $skillsRoot $name
  New-Item -ItemType Directory -Force -Path $target | Out-Null
  Copy-Item "$name.md" -Destination (Join-Path $target "SKILL.md") -Force
}
```

---

## Skill 协作关系

参见 [../04-Skills体系说明.md](../04-Skills体系说明.md) 中的时序图。
