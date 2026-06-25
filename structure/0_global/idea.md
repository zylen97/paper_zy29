# zy29 研究纲领文件

> **版本**: v0.2
> **日期**: 2026-06-23
> **目标期刊**: Computer-Aided Civil and Infrastructure Engineering (CACAIE) / Advanced Engineering Informatics (AEI) [首选] / Reliability Engineering & System Safety (RESS), Journal of Infrastructure Systems [备选]
> **定位**: 硬篇。把基础设施劣化的协同迁移建模，从"被动异质性处理（全局池化 / 固定分族 / shrinkage）"推进到"**主动的、负迁移感知的、可审计的源选择（source gating）**"。
> **英文定位词**: *Transferability-aware, source-selective transfer learning for data-scarce deterioration of aging infrastructure assets*（主验证：桥梁 US NBI）。
> **术语红线**: 不用 `cross-asset`（金融/跨资产类别串味）、不用裸 `knowledge transfer`（偏知识管理软话题）。统一用 `transfer learning / domain adaptation / source-selective / transferability-aware / collaborative deterioration modeling`。

---

## 1. 灵感来源 & 参考论文

### 源头论文（机制灵感，中文，未进英文池）

借鉴徐进团队工程装备知识跨项目迁移两篇：
1. 徐进 et al. (2024)《基于特征差异增强的工程装备知识跨项目多源域迁移学习研究》(系统工程理论与实践)：核心启发＝先用目标少量数据评估候选源的任务表现，再挑可迁移源，避免"所有历史源都可复用"的粗暴池化。
2. 徐进 et al. (2026)《基于时序分块多知识融合的工程装备知识跨项目转移研究》：核心启发＝源内部也存在分布漂移；有效迁移需识别可用知识段、抑制源内负迁移。

> 边界已查证：徐进这套"负迁移感知源选择"机制**只存在于中文论文**，其英文期刊论文（ESA 2022 知识图谱、IEEE TEM 2024 DRL）用的是不同机制。我们**引用徐进、借其机制视角**，但全部重做于基础设施情境，并以工程先验 + 可审计 + 协同建模上下文构成新贡献——**不是抄徐进，是补 Chen（见下）。**

### 头号相关工作 / 头号基线 / foil（英文 infra）

**Chen, Yuan & Lin (2025), "Cross-jurisdictional collaborative deterioration modeling via hierarchical Bayesian transfer learning", CACAIE 40:6456–6474, DOI 10.1111/mice.70159.**
- 做了什么：分层 Bayesian 参数迁移 + BTGP 随机过程，把数据稀疏辖区的桥梁劣化模型与全局共享 (μ,σ) 做 **coherent partial pooling**；属性当协变量 θ=exp(ξz)；辖区级；下游验 RL/LCC；基线 AS(独立)/TL-0(池化经验Bayes)/TL-I(分层)。
- **它明文留下的洞（第6463页原文）**："the treatment of outliers or contaminated data lies beyond the scope of Bayesian modeling itself and should be properly addressed during the data preprocessing stage" —— **承认坏源会害结果，但把"有害源识别/处理"划到 scope 之外。** 这正是本文要填的洞。
- 它也承认（第6469页）池化会 over-shrinkage、异质大时 posterior 偏 —— 看见了负迁移症状，但只靠分层结构软处理，**无显式挑源/剔除有害源**。

### 同子领域 collaborative / fleet 群（用其语言，靠 selectivity 区分）

Bull et al. (2023) engineering fleets 分层 Bayesian；Dhada et al. (2025) Weibull 协同预后；Guo et al. (2025) 组内/组间协同 RUL。共性：**被动异质性处理**（shrinkage / 预定义 cluster / group），无 per-target 有害源感知选择。

---

## 2. 行业背景与实践难题 → 文献不足 → Gap/Objective/RQ

### 行业背景与实践难题

大量桥梁等基础设施老化，养护部门须提前判断哪些资产将退化到维修阈值以下，以排预算和维修优先级。但很多资产历史记录稀疏（新建、刚大修、或交通/环境刚变），单资产建模不稳。国家级数据库（如 US NBI）积累海量历史桥梁，是天然的"机群知识源"，但桥与桥在结构、材料、年龄、气候、交通、养护制度上差异巨大。

### 文献不足（去稻草人版）

桥梁劣化预测已成熟：主流用**按结构类型/环境分族的 Markov 转移矩阵**、**带协变量的回归/生存模型**、以及**分层 Bayesian 协同建模**（Chen 2025 为代表）处理跨桥异质性。但这些方法处理异质性都是**被动**的——靠固定分组、协变量、或自动 shrinkage，并**共同隐含"共享/池化总体有益"**。它们都**缺乏**：(a) 面向某一稀疏目标的**自适应源选择**（该跟哪些源学）；(b) **负迁移/有害源的显式识别与剔除**（Chen 明文 out of scope）；(c) **源内可迁移段的细分**（把每个源当均质整体）；(d) 面向资产管理者的**可审计源归因**。

### Gap → Objective → RQ

| Gap | Objective | RQ |
|:----|:----------|:---|
| G1: 现有协同/分层 Bayesian 劣化迁移**向全局池化、默认共享有益**，缺乏面向稀疏目标的自适应源选择；当存在真不可迁移源时会污染全局、产生负迁移。 | 构建**负迁移感知的源选择（source gating）层**，base-learner 无关，可前置/叠加于协同建模框架。 | RQ1: 历史稀疏时，源选择式迁移能否**打赢** target-only、TL-0、TL-I（Chen 2025），尤其在源含真不可迁移成分时？ |
| G2: 何种源可迁移、何种源造成负迁移，缺乏工程语义化判据；属性只被当协变量而非挑源信号。 | 设计**工程相似性先验 × 目标少样本实测可迁移性 × 分布差异**的门控，并做**源内退化阶段/维护 regime 分段**。 | RQ2: 哪些工程相似性维度决定源可迁移？源在何条件下有害？源内哪些段可借？ |
| G3: 现有迁移/池化为黑箱，无法向资产管理者说明"为何借此源、排彼源"，实践价值停在预测精度。 | 输出**可审计源归因**（源权重 + 相似性证据 + 负迁移风险），并验证其对维修排序/预算决策的影响。 | RQ3: 可审计源门控能否在保持/提升性能的同时，为维修与预算决策提供可审计依据，并**改变决策**（RL/LCC 层面）？ |

---

## 3. 方法论选择论证

数据驱动计算框架。目标不是单纯提精度，而是回答"从谁学""何时不学""为何这样学"。框架：

1. **数据任务**：US NBI（50 州、60 万+ 桥，BCI/状态评级）构造"目标桥历史稀疏"的 cold-start 任务；（可选第二资产类：LTPP 路面 / 供水管网，做泛化与"基础设施通用"佐证）。
2. **base learner（可换）**：BTGP 随机过程（与 Chen 同基，便于干净对照）/ 序数分类 / survival / gradient boosting。**贡献不在 base，在其上的 gating 层。**
3. **源选择门控（轴一·源间）**：source weight = f(工程相似性先验 × 目标少样本上实测可迁移性 × 分布差异)，输出可解释权重 + 有害源标记。
4. **源内分段（轴二·源内，infra 圈空白）**：源桥历史按退化阶段 / 大修前后 regime 切段，段级判可迁移，只融合可借段（徐进时序分块的低频转译）。
5. **负迁移检测**：加入候选源前后对预测与阈值预警的影响 → harmful source 与负迁移率。
6. **基线**：target-only、naive all-pooling（下限对照）、**Chen 2025 的 AS / TL-0 / TL-I（头号对照）**、固定分族 Markov / 全局协变量模型。
7. **评估双轨**：① 兼容轨（可比）参数 SD 降幅、RL 分布、LCC 分布（同 Chen）；② 创新轨 负迁移率、有害源识别 precision/recall、对抗/异质源混入鲁棒性；③ **决策轨** 源选择是否改变维修优先级 / 预算（RL/LCC 决策差异）。

> **crux 实验（go/no-go，先验证再动笔）**：构造"真不可迁移源（来自不同总体）"，证明分层 Bayesian partial pooling 被其灌入全局 μ 而偏，本文挑源能识别并剔除回正。合成数据 + US NBI 各一版。**若 pooling 在 NBI 上对稀疏目标并不掉点，则前提不成立，需回炉。**

---

## 4. 贡献意图

### 理论贡献
1. 把基础设施协同劣化迁移从"**被动异质性处理**（全局池化 / 固定分族 / shrinkage，默认共享有益）"推进到"**主动的、负迁移感知的源治理**"——明确"机群历史是逐目标判断的、有条件可迁移的退化知识"。
2. 填补 Chen 2025 明文留下的"有害源识别 out of scope"缺口：把负迁移检测做成**有原则的 in-model 机制**，而非数据预处理。
3. 提出**两条 novelty 轴**：源间（工程先验 × 可迁移性的挑源 gating）+ **源内（退化阶段/维护 regime 分段，infra 圈空白）**。

### 实践贡献
4. 为 DOT/资产管理部门提供**冷启动劣化预测 + 可审计维修优先级**工具，避免黑箱把不适配历史经验迁入当前资产。
5. 把价值落在**决策层**（RL/LCC、维修与预算排序），证明"挑对源会改变决策"，而非仅 MAE/RMSE。

---

## 5. 风险评估

| 风险 | 等级 | 应对 |
|:-----|:----:|:-----|
| 打不赢分层 Bayesian 的自适应 shrinkage（它按数据量自动调，是真强基线） | 🔴高 | **精确 scope claim**：只在"源数据量相近但可迁移性不同 / 含真不可迁移源"时主张优势；crux 实验正面证明 pooling 在此场景失效、挑源回正。不夸"全面碾压"。 |
| 被指"只是标准 multi-source domain adaptation / source selection 换数据集" | 🔴高 | novelty 锚在 **工程相似性先验挑源 + 源内分段 + 协同建模上下文 + 可审计决策**的组合，且填的是 Chen 明文留洞，非泛泛迁移。 |
| 负迁移前提不成立（NBI 上 pooling 其实不掉点） | 🔴高 | go/no-go pilot 先验证；不成立则回炉换情境/换 framing。 |
| "通用基础设施"声称广、只验桥 | ⚠️中 | 要么加第二资产类（路面/管网）挣通用，要么 framing scope 成"机制 asset-class 无关，本文实例化于桥，扩展见 future work"，不夸"已证明"。 |
| NBI 检查周期/评级口径/维护记录噪声 | ⚠️中 | 明确清洗规则；状态转移/阈值预警指标；对州/年份/资产类型稳健性检验。 |
| 徐进时序分块不适用低频检查数据 | ⚠️中 | 不照搬 TBMKF；"分块"转译为退化阶段 / 维护前后 regime / 生命周期阶段。 |

---

## 6. 残留待清理（提醒）

- `structure/3_methodology/methodology.md` 仍是博弈/供应链/均衡模板，与本 data-driven transfer 完全不匹配，需整章重写（待 /method-audit 或技术章撰写时处理）。
- `structure/2_literature/literature.md` 大纲为 TODO；文献检索须**双线**：infra deterioration/collaborative modeling + transfer learning/domain adaptation/transferability/negative transfer。
