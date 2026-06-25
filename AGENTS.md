# Project Contract

## 项目信息
- **项目编号**: zy29
- **英文标题**: Negative-transfer-aware cross-asset knowledge transfer for infrastructure deterioration prediction
- **一句话概括**: 构建面向桥梁/路面资产劣化预测的可解释跨资产知识迁移框架，识别可迁移源资产并抑制负迁移。
- **方法**: data-driven computational modeling / transfer learning
- **目标期刊**: Advanced Engineering Informatics [首选] / Automation in Construction, Journal of Infrastructure Systems [备选]

## 项目阶段
- 状态: foundation
- 更新时间: 2026-06-21

> **阶段枚举**（由 skills 自动维护，勿手动跳级）：
> - `foundation`：idea 确定 + 文献 + idea-refine + 技术型章节开发/定稿（步骤①-⑤.5）
> - `drafting`：叙述型章节撰写 + 全文定稿（步骤⑥-⑨）
> - `submitted`：已投稿，等待审稿
> - `revision-R{N}`：第 N 轮审稿修改

## 核心配置
- **模板**: Elsarticle (preprint)
- **主文件**: `manuscript.tex` · **文献**: `zy29.bib`
- **投稿附件**: `submission/`（coverletter.tex, declaration.tex, highlights.tex, titlepage.tex）

## Codex Academic Interaction Layer

本项目遵循全局交互合同：

`/Users/zylen/.codex/skills/_academic-shared/ACADEMIC-INTERACTION-CONTRACT.md`

- 讨论、方案、审计报告不等于授权写文件；核心 md/tex/state 写入前必须展示 Write Gate。
- `git add` / `git commit` 是 L3 checkpoint 动作，必须单独确认并只 stage 当前 workflow 产物。
- `git push`、发布、外部发送是 L4 动作，默认禁止，除非用户明确要求。
- subagent 必须遵守同一交互合同，不创建隔离 worktree，不越权写文件。

## data/ 数据管理规范

```
data/
├── raw/          # 原始数据（只读，不可修改）
├── processed/    # 数据管道产出（清洗后的中间文件）
├── scripts/      # 所有 R/Python 分析脚本（数据管道 + 主模型）
├── models/       # 模型估计对象（.rds/.RData，用于 warm start 和交叉引用）
├── results/      # 模型输出日志（.txt，人类可读的估计结果）
├── robustness/   # 稳健性检验（脚本 + 结果 + 日志）
└── _archive/     # 废弃的旧版本文件（不删，按原子目录归档）
```

**命名规范**：所有文件以项目编号为前缀（如 `zy29_`）
- 脚本：`zy29_data_pipeline.R`、`zy29_models.R`
- 模型对象：`zy29_m1.rds` ~ `zy29_mN.rds`
- 日志：与模型对象同名，`.txt` 后缀
- 稳健性：`zy29_rob_{检验名}.R`、`zy29_rob_{检验名}.rds`

**输出路径规则**：分析脚本的所有输出必须指向 `data/` 下的对应子目录，禁止输出到项目根目录。对于 RSiena 等通过 `projname` 参数控制输出路径的工具，`projname` 必须包含目标目录（如 `"data/results/zy29_m3"`）。

**脚本执行顺序**：
1. `zy29_data_pipeline.R` → 产出 `processed/*.rds`
2. `zy29_models.R` → 产出 `models/*.rds` + `results/*.txt`（依赖步骤 1）
3. `robustness/zy29_rob_*.R` → 产出 `robustness/*.rds`（依赖步骤 1+2）

## structure/ 组织说明
项目研究素材按 manuscript 章节组织在 `structure/` 下：

| 子目录 | 内容 | 核心文件 |
|:-------|:-----|:---------|
| `0_global/` | 跨章节纲领、参考PDF库 | `idea.md`（纲领）, `pandoc_header.tex` |
| `1_introduction/` | Introduction blueprint 与素材 | `introduction.md` |
| `2_literature/` | Literature Review blueprint、RIS、direction reports、citation_pool/ | `literature.md`, direction reports, master.bib（lit-pool） |
| `3_methodology/` | 计算框架、变量定义、源资产选择机制 | `methodology.md`（唯一权威源） |
| `4_results/` | 预测实验、负迁移识别、消融与稳健性 | `results.md`（唯一权威源） |
| `5_simulation/` | cold-start 场景、维护排序价值、敏感性分析 | `simulation.md`（唯一权威源） |
| `6_discussion/` | Discussion 写作素材（不强制中间 md） | `manuscript.tex` 中的 Discussion section |
| `figures_tables/` | 图表集中管理 | `index.md`（注册表）, `tables.tex`（所有表格）, `figures/`（所有图片） |

- `pandoc_header.tex` 勿删改

### 图表管理规则

- **所有图片**存放于 `structure/figures_tables/figures/`，仿真脚本输出路径指向此处
- **所有表格**的 LaTeX 代码统一写在 `structure/figures_tables/tables.tex` 中，定稿时贴到 manuscript.tex 末尾
- **`index.md`** 是唯一注册表，记录每个图表的编号、标题、所属章节、状态
- 各章节 md 中**不放图表内容**，只标注"此处引用 Table X / Fig. X"
- 新增/修改图表时，必须同步更新 `index.md`
- **表格即时落地**：当章节 md 中的表格内容定稿后，应立即写入 `tables.tex`（使用 /latex-table 或直接编写），并更新 `index.md` 状态为 `drafted`。manuscript.tex 正文中仅用 `Table~\ref{tab:xxx}` 引用，不嵌入表格代码。不得将表格制作推迟到 /technical 或其他后续环节——/technical 只管正文文字。

### 文件交互规则

**层级**: idea.md（纲领：为什么/方向）→ 技术型 final chapter md（证据与写作蓝图）→ manuscript.tex（唯一正文正本）

**章节来源的两种类型**:

论文写作分为两条路径，颗粒度要求不同：

| 类型 | 章节 | 文件 | 内容要求 | sci-writer 的任务 |
|:-----|:-----|:-----|:---------|:-----------------|
| 叙述型（blueprint-first） | Introduction, Literature Review | `introduction.md` / `literature.md` + `idea.md` + citation pool + `manuscript.tex` | 先生成/更新叙述型 blueprint 并 checkpoint，再写入 tex | 由 `/narrative` 先锁定逻辑合约，再替换 `manuscript.tex` 对应章节 |
| 叙述型（tex-in/out） | Discussion | `manuscript.tex` + technical final md + results evidence | 直接围绕实际方法、结果、贡献和局限写入英文段落 | 由 `/narrative` 直接写入 `manuscript.tex`，不强制 Discussion 中间 md |
| 技术型 | Methodology, Results, Numerical analysis | `X.md`（权威成稿源） | 直接承载 manuscript-bound 正文要点、公式、命题/假设、证明骨架、数值结论、图表引用和证据锚点 | 由 `/technical` 直接生成学术英文，按 `###` 层级生成 `\subsection`，不回读 `_dev.md` |

**技术型 final-md-first 体系**：
- **`X.md`（权威成稿源）**：manuscript 写入前的唯一技术正文源。直接承载正文要点、公式、命题/假设、证明骨架、数值结论、图表引用和证据锚点。`###` 标题 = `\subsection`，`####` = `\subsubsection`。
- **`X_dev.md`（可选归档/溯源笔记）**：可保存推导草稿、失败尝试、长计算和过程记录；不是必填输入，不被 `/technical` 读取，不得决定 `X.md` 的章节结构或正文措辞。
- **权威顺序**：`manuscript.tex`（完成写入后）→ final chapter md `X.md` → `idea.md`/审计报告/数据与图表证据 → `X_dev.md`。

**md 书写语言**：中文表述，但以下内容用英文原文：citation key（`\citet{}`/`\citep{}`）、专用术语、公式符号、`###` 标题（与 manuscript 一致）

**叙述型标准**：
- `manuscript.tex` 是最终正文正本；`introduction.md` 与 `literature.md` 是 Introduction/Literature Review 的 blueprint 逻辑合约，不是最终正文
- `/narrative section=introduction|literature` 必须先更新并 checkpoint 对应 blueprint，再生成和替换 tex 段落
- `/narrative section=discussion` 直接基于实际 Methodology/Results 和 `manuscript.tex` 写入，不强制 Discussion 中间 md
- 每个核心论点、Gap/RQ 对应关系和 citation key 必须在写入前被显式检查

**技术型颗粒度标准**（验证原则：sci-writer 翻译时不需要做任何技术判断）：
每个 `###` subsection 下必须包含：
- 每一个会出现在正文中的公式（编号公式和关键行内公式）
- 每一个命题/引理的完整陈述
- 每一个证明的思路骨架（完整证明用 `> **Proof (→ Supplementary):**` 标记，sci-writer 外放至附录）
- 每一个假设的文字表述 + 合理性论证
- 每一个经济直觉/含义解读
- 每一个模型间的对比说明
- 表格和图表仅标注"此处引用 Table X / Fig. X"，内容在 `figures_tables/tables.tex`

**技术型素材迭代工作流**:
1. 数据、模型、证明、仿真和图表证据先落到 `data/`、`proofs.md`、`figures_tables/`、审计报告等可追溯位置。
2. 直接从成熟证据写入 `X.md`，按 manuscript 读者逻辑组织 subsection。
3. `/method-audit` 审计并锁定 `X.md`：证据支撑、benchmark fit、reader logic、顺序、字数锚点、citation/table/figure alignment、跨章节一致性；不要求非空 `_dev.md`。
4. `/technical` 只从 `X.md` 写入 `manuscript.tex`；不得回读 `_dev.md` 补正文。

**idea.md 规范**:
- 修改必须先征得用户同意，不得自行回写
- 保持文档结构不变，覆盖式更新内容（迭代历史由 git 追踪）
- 只放"为什么"和"方向"，不含具体公式/推导/数值结果。违反则移至对应章节 md

**章节一致性约束**:
- RQ编号：idea.md §2 定义 → introduction.md 展开 → `manuscript.tex` Discussion 回应，三处编号和措辞必须一致
- 贡献声明：idea.md §4 意图 → `manuscript.tex` Discussion 论证，逻辑必须对应

**研究推进顺序**：见全局 `~/.codex/AGENTS.md` 中的"科研论文研究推进顺序"。
