# 05 · Rules 体系说明

> Rules 层是"AI 的宪法"。每次会话自动注入，用来**路由请求、约束阶段、强制门禁**。

---

## 一、Rules 的两种形态

| 类型 | 位置 | 作用范围 | 生效时机 |
|------|------|---------|---------|
| 全局 user_rules | Cursor / Claude Code 用户设置 | 所有项目 | 每次会话 |
| 项目规则 | `.cursor/rules/*.mdc` | 当前仓库 | 文件被打开或被引用时 |

### 全局 vs 项目

- **全局**：跨项目通用约束，比如"按 `auto-programming-workflow` 路由"、"中文回答"、"`.shared_cache/` 缓存约定"
- **项目**：本地增强，比如"本仓库固定用 F01-F12 编号模块"、"测试命令用 pytest"

**原则**：尽量把跨项目通用的规则放全局；项目规则只写**本仓库独有**的约定。

---

## 二、核心项目 Rules（推荐配置）

本知识库推荐每个项目都配置 3 份基础规则：

```text
.cursor/rules/
├── workflow-stage-routing.mdc          # 告诉 AI 某类请求应该调哪个 Skill
├── workflow-gates-and-handoffs.mdc     # 告诉 AI 不得跳阶段
└── shared-cache.mdc                    # 告诉 AI 如何使用 .shared_cache/
```

### 2.1 workflow-stage-routing.mdc（路由）

```markdown
---
description: Route multi-stage delivery work to the correct skill
alwaysApply: true
---

# Workflow Stage Routing

- When end-to-end delivery → use delivery-orchestrator first.
- When 梳理需求 / changelog / PRD → use product-manager.
- When changelog+PRD confirmed and TECH/实现 → use tech-developer.
- When multi-module parallel → use parallel-dev.
- When code complete → use qa-tester.
- When QA passed → use merge-review.
- When merge/tag/push → use git-workflow.
```

> 完整模板：[templates/rules/workflow-stage-routing.mdc](./templates/rules/workflow-stage-routing.mdc)

### 2.2 workflow-gates-and-handoffs.mdc（门禁）

```markdown
---
description: Enforce workflow gates and skill handoffs across delivery stages
alwaysApply: true
---

# Workflow Gates And Handoffs

- 不跳阶段：PM → TD → PD(可选) → QA → MR → GW
- TECH 未确认不开始编码
- QA 未通过不进 merge-review
- merge-review 无阻塞 + 用户确认后才进 git-workflow
- 需求不清回退 PM；测试失败回退 TD；审查阻塞回退 TD
- Git 冲突/权限问题停止并询问用户
```

> 完整模板：[templates/rules/workflow-gates-and-handoffs.mdc](./templates/rules/workflow-gates-and-handoffs.mdc)

### 2.3 shared-cache.mdc（缓存）

```markdown
# 本地共享缓存约定

- 每个项目根目录使用 .shared_cache/ 作为本地持久缓存
- 必须加入 .gitignore
- git 元数据直接通过 git 命令实时查询
- 不读取、不创建、不刷新 .shared_cache/git/*.json
- shared-data 下的业务共享 JSON 应含 last_updated (ISO 8601)
```

> 完整模板：[templates/rules/shared-cache.mdc](./templates/rules/shared-cache.mdc)

---

## 三、Rule 文件的语法要点

### 3.1 Front matter 三个字段

```yaml
---
description: 一句话说明（AI 会用它判断要不要读）
alwaysApply: true/false    # true=每次都注入；false=按需
globs: "src/**/*.ts"       # 可选，匹配到对应文件才注入
---
```

### 3.2 推荐写法

- **用清单**（`- ...`）代替长段落，AI 更容易严格遵守
- **用祈使句**（"必须 / 不得 / 优先"）
- **明确回退路径**（"X 失败 → 退回 Y"）
- **避免条件过多的 if-then-else**，拆成多条独立规则

### 3.3 反模式

| 反模式 | 说明 |
|--------|------|
| 规则太长 | 超过 200 行 AI 容易记忆漂移 |
| 规则重复 | 全局规则和项目规则写重复内容 |
| 主观表述 | "尽量 / 看情况 / 最好"这种模糊词 |
| 用代码示例 | Rules 不适合放大段代码，示例放 Skills |

---

## 四、Rules 与 Skills 的分工

| 维度 | Rules | Skills |
|------|-------|--------|
| 形态 | 短、强约束 | 长、教学式 |
| 注入 | 自动、每次会话 | 按关键词触发 |
| 目标 | "不能做什么 / 必须做什么" | "怎么做" |
| 更新频率 | 低（项目定型后稳定） | 中（迭代方法论） |
| 数量 | 3~5 条为宜 | 按角色数量（8 ± 2） |

**简单判别**：

- 如果是"硬性红线" → 写 Rule
- 如果是"操作手册" → 写 Skill

---

## 五、全局 user_rules 推荐配置

在 Cursor / Claude Code 的"用户规则"里，至少维护以下内容：

```markdown
# 1. 工作流总入口
- 当要求端到端交付/全流程开发时，优先启用 auto-programming-workflow 技能

# 2. 共享缓存约定
- 每个项目根目录使用 .shared_cache/ 作为本地持久缓存
- 必须加入 .gitignore
- git 元数据直接通过 git 命令实时查询
- 不读取、不创建、不刷新 .shared_cache/git/*.json

# 3. 沟通语言
- Always respond in 中文
```

这些规则不依赖任何项目文件，新项目也能直接生效。

---

## 六、排错

### 6.1 AI 似乎没遵守 Rule

- 检查 `alwaysApply` 是否 `true`
- 检查 Rule 是否太长导致后半段被截断
- 检查是否与更强的上游规则冲突（系统规则 > 用户规则 > 项目规则）

### 6.2 多 Rule 冲突

- 更具体的 Rule 优先级更高
- 项目规则覆盖全局规则
- 明确冲突时，修改更上游的规则使其加限定语

### 6.3 Rule 与 Skill 都强制某事

- 保留 Rule 里的硬约束
- 在 Skill 里重复一次作为"工作手册提醒"，增强一致性

---

## 七、延伸阅读

- [04 - Skills 体系说明](./04-Skills体系说明.md)
- [07 - 落地实施 Checklist](./07-落地实施Checklist.md)
- [templates/rules/](./templates/rules/)
