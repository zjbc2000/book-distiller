# Book Distiller（拆书成 Skill）

把一本软件 / AI-coding 相关书籍**蒸馏成可复用的 agent 技能**。输入可以是 PDF、Markdown 或微信读书链接；简单书产出 skill 形态，复杂书（需要 hooks / MCP / 子代理 / 分发）产出 plugin 形态。

## 项目定位

书是"死"的——翻箱倒柜才找到一段方法论；skill 是"活"的——agent 按触发词直接调用。本工具把书里散落的知识重新组织成 agent 可直接执行的技能包，而不是照抄目录。

- **输入三种路径**：PDF（转 Markdown）、Markdown（直接用）、微信读书链接（强制全文导出）
- **产出两种形态**：裸 skill（方法论书默认）或 plugin（需要 hooks/MCP/子代理/分发时）
- **质量保证**：主题划分、子 skill 生成两处强制检查点 + 脚本自动校验
- **可追溯**：每条关键结论标注 `来源：第N章`，防幻觉

## 六阶段流程

| 阶段 | 做什么 | 检查点 |
|---|---|---|
| **1. 输入检测** | 判断输入类型，统一成干净 Markdown + 章节树 | |
| **2. 结构分析** | 识别章节边界，每章生成 800–1200 token 摘要（map，密度优先、不抄原文） | |
| **3. 主题划分** | 基于全部章摘要提出 3–10 个主题清单 | ★ 强制停下，用户确认/增删/合并 |
| **4. 判别树** | 按判别树判定 skill / plugin 形态，展示给用户 | （检查点 1 顺带确认，可改判） |
| **5. 逐主题生成** | 按 skill-template 逐个生成子 SKILL.md | ★ 首样例确认格式，其余自动跑 |
| **6. 自检与交付** | 校验脚本 + 汇总报告（主题清单、覆盖范围、下一步建议） | |

判别树要点：**skill 优先，plugin 是升级路径**。程序性知识书（Clean Code 这类）默认 skill；需要 MCP server / hooks / 子代理 / 多命令分发 / 团队分发时走 plugin。

## 用法

### 输入处理

```bash
# PDF → Markdown（扫描版会提示换 MinerU）
python3 scripts/extract_text.py <pdf> -o <out.md>

# 微信读书链接 → 全文 Markdown（需先 clone 导出器）
git clone https://github.com/lbq110/weread-exporter ~/tools/weread-exporter/
python3 scripts/weread_fetch.py <链接> \
  --exporter=~/tools/weread-exporter/export_precise.py \
  --workdir=~/tools/weread-exporter/

# Markdown 输入：直接用
```

微信读书路径首次使用会弹出浏览器扫码登录；导出是长任务（整本书 30 分钟至数小时）。

### 校验产出

```bash
# skill 形态：校验总入口 + 所有子 skill
python3 scripts/validate_skill.py outputs/<book-slug>/SKILL.md outputs/<book-slug>/*/SKILL.md

# plugin 形态：额外校验 plugin 结构
python3 scripts/validate_plugin.py outputs/<book-slug>
```

## 产出物结构

```
outputs/<book-slug>/
├── SKILL.md          # 总入口：frontmatter + 全书心智模型 + 主题决策树（场景 → 调哪个子 skill）
├── <topic>/SKILL.md  # 各主题子 skill（每个独立可触发）
└── references/       # 章节精要，子 skill 深入时按需加载
```

plugin 形态时：上述内容移入 `skills/`，新增 `.claude-plugin/plugin.json`（+ 可选 `marketplace.json` / `agents/` / `hooks/` / `.mcp.json`）。

## 实战样例

`outputs/ai-programmer-fullstack/` 是《人人都是AI程序员》全书蒸馏出的 6 个全栈主题 skill（ai-backend-dev / ai-frontend-design / ai-product-engineering / ai-project-planning / lean-fullstack-tools / vibe-coding）。

## 测试

```bash
python3 -m pytest tests/
```

（35 个用例覆盖四个脚本的输入处理、校验逻辑）
