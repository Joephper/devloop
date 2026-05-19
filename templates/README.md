# 模板库

> 开箱即用的模板集合，复制到新/老项目即可。

---

## 目录一览

```
templates/
├── rules/                              # .cursor/rules/ 下的项目规则
│   ├── workflow-stage-routing.mdc      # 阶段路由：把请求交给对的 Skill
│   ├── workflow-gates-and-handoffs.mdc # 门禁规则：不得跳阶段
│   └── shared-cache.mdc                # 本地共享缓存约定
├── docs/                               # docs/ 下的项目文档模板
│   ├── docs-README.md                  # 文档中心首页
│   ├── changelog-LATEST.md             # 当前迭代变更清单
│   ├── module-PRD.md                   # 模块产品需求
│   ├── module-TECH.md                  # 模块技术方案
│   ├── test-report.md                  # 测试报告
│   └── merge-review.md                 # 合并前审查意见
└── skill-template.md                   # 自定义 Skill 的脚手架
```

---

## 使用方式

### 新项目首次接入

```powershell
# 1. 建目录
mkdir my-project\docs\{overview,modules,changelog}
mkdir my-project\.cursor\rules
mkdir my-project\.shared_cache\shared-data

# 2. 复制 Rules
Copy-Item templates\rules\*.mdc my-project\.cursor\rules\

# 3. 复制文档模板
Copy-Item templates\docs\docs-README.md      my-project\docs\README.md
Copy-Item templates\docs\changelog-LATEST.md my-project\docs\changelog\LATEST.md

# 4. 在 .gitignore 追加
Add-Content my-project\.gitignore ".shared_cache/"

# 5. 提交第一版骨架到 Git
cd my-project
git init
git add docs .cursor .gitignore
git commit -m "chore: init AI workflow scaffold"
```

### 迭代中使用

| 迭代阶段 | 使用哪个模板 | 放到哪 |
|---------|-----------|------|
| 需求梳理完成 | `changelog-LATEST.md` | `docs/changelog/LATEST.md`（覆盖） |
| 新增模块 | `module-PRD.md` + `module-TECH.md` | `docs/modules/F{xx}-{slug}/` |
| 测试完成 | `test-report.md` | `docs/changelog/{迭代}-test-report.md` 或 `backend/tests/reports/` |
| 合并前审查 | `merge-review.md` | MR 描述或 `docs/changelog/{迭代}-review.md` |

### 自定义 Skill

如果要为项目新增一个专属 Skill：

```powershell
Copy-Item templates\skill-template.md ~\.claude\skills\my-skill\SKILL.md
# 按模板填充
```

---

## 模板设计原则

- **占位符用大括号** `{placeholder}` —— AI 能识别并自动替换
- **所有模板带"状态"字段** —— 方便状态机识别
- **Mermaid 图预留位置** —— 图优于文字
- **表格优于段落** —— 更易索引
