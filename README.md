# wetlab_skills

湿实验技能库：可复用的实验协议、平台约束与自动化工作流，按层级（Level 0–3）分层组织。

## 目录结构

| 路径 | 层级 | 说明 |
|------|------|------|
| `schemas/` | Level 0 | 校验规范与数据契约（JSON Schema） |
| `platforms/` | Level 1 | 硬件平台与耗材能力库 |
| `skills/` | Level 2 | 原子级子 Skill 库（按实验领域分类） |
| `workflows/` | Level 3 | 顶层战役编排与动态路由规则 |
| `scripts/` | — | 工具脚本库（校验 / 转译） |
| `.github/workflows/` | — | CI/CD 自动化校验（push 到 GitHub 后生效） |

## 快速开始

```bash
# 本地校验 Skill 文件
python scripts/validate_skills.py
```

## 维护约定

- 新增原子 Skill 写入 `skills/<实验领域>/skill_*.md`，须符合 `schemas/skill_schema.json`。
- 新增战役工作流写入 `workflows/wf_*.md`，须符合 `schemas/workflow_schema.json`。
- 原始下机数据不入库（见 `.gitignore`），走 LIMS/云存储。

（内容待补充）
