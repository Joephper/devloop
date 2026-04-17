# v{x.y} 合并前审查意见

> 日期：{YYYY-MM-DD}
> 源分支：`feature-xxx`
> 目标分支：`dev` / `pre` / `master`
> 审查人：{角色}
> 对应 changelog：[LATEST.md](../changelog/LATEST.md)
> 对应测试报告：[test-report.md](./test-report.md)

---

## 一、变更摘要

- 提交数：{N}
- 改动文件：{N}
- 新增行数：{+N}
- 删除行数：{-N}
- 涉及模块：F{xx}、F{yy}

---

## 二、Findings

> 严重级：**blocker** > **major** > **minor** > **nit**

### [blocker] `path/to/file.py:123`

{问题描述}

**建议**：{具体修改建议}

### [major] `path/to/file.py:45`

{问题描述}

**建议**：{具体修改建议}

### [minor] `path/to/file.py:78`

{问题描述}

**建议**：{具体修改建议}

### [nit] `path/to/file.py:200`

{风格/命名建议}

---

## 三、一致性检查

| 检查项 | 结果 |
|--------|------|
| 与 changelog 范围一致 | ✅ / ❌ |
| 与 PRD 验收标准一致 | ✅ / ❌ |
| 与 TECH 技术方案一致 | ✅ / ❌ |
| 测试覆盖无明显缺口 | ✅ / ❌ |
| 无调试代码 / TODO / 临时配置 | ✅ / ❌ |
| 无密钥泄露 / 硬编码敏感信息 | ✅ / ❌ |

---

## 四、Open Questions

- {待确认项 1：需要用户/产品/架构师决策}
- {待确认项 2}

---

## 五、Merge Recommendation

- [ ] **建议合并**
- [ ] **不建议合并**
- [ ] **建议合并（附条件）**

**理由：**

{说明依据，即使无问题也要明确写"未发现阻塞性问题"，并说明残余风险}

**残余风险：**

- {风险点 1}
- {风险点 2}

---

## 六、交接

- 审查通过 + 用户确认 → 交给 `git-workflow`
- 审查不通过 → 退回 `tech-developer`
- 需要补测试 → 退回 `qa-tester`
