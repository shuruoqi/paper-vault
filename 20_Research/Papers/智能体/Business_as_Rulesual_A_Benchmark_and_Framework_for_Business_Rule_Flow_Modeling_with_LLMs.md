---
document_id: "2026.acl-long.1625"
source_type: "pdf_url"
source_url: "https://aclanthology.org/2026.acl-long.1625/"
title: "Business as Rulesual: A Benchmark and Framework for Business Rule Flow Modeling with LLMs"
authors: "Chen Yang, Ruping Xu, Ruizhe Li, Bin Cao, Jing Fan"
published: "2026-07"
domain: "智能体"
tags:
  - "论文笔记"
  - "智能体"
  - "pdf_url"
  - "业务规则抽取"
  - "SOP"
  - "benchmark"
repo: "https://github.com/oYoungCo/Business-as-Rulesual"
quality_score: "5.8"
related_papers:
  - "2607.11346"
  - "2508.02721"
created: "2026-11-12"
updated: "2026-11-12"
status: "analyzed"
---

# Business as Rulesual: A Benchmark and Framework for Business Rule Flow Modeling with LLMs

## 核心信息
- **文档 ID**：2026.acl-long.1625
- **来源类型**：pdf_url（输入为 ACL Anthology 论文页，全文取自其 PDF）
- **作者**：Chen Yang, Ruping Xu, Ruizhe Li, Bin Cao, Jing Fan
- **机构**：浙江工业大学（含浙江省视觉信息智能处理重点实验室）、阿伯丁大学、伯明翰大学
- **发布时间**：2026-07（会议 2026-07-02 至 07-07）
- **会议/期刊/发布方**：ACL 2026（第 64 届年会，Long Papers，pp. 35198–35220）
- **链接**：[ACL Anthology](https://aclanthology.org/2026.acl-long.1625/) | [PDF](https://aclanthology.org/2026.acl-long.1625.pdf)
- **代码**：[oYoungCo/Business-as-Rulesual](https://github.com/oYoungCo/Business-as-Rulesual)
- **引用**：--

## 摘要与核心要点

### 原文摘要
Extracting structured procedural knowledge from unstructured business documents is a critical yet unresolved bottleneck in process automation. While prior work has focused on extracting linear action flows from instructional texts (e.g., recipes), it has insufficiently addressed the complex logical structures—such as conditional branching and parallel execution—that are pervasive in real-world regulatory and administrative documents. Furthermore, existing benchmarks are limited by simplistic schemas and shallow logical dependencies, restricting progress toward logic-aware large language models (LLMs). To bridge this "Logic Gap", we introduce BREX, a carefully curated benchmark comprising 409 real-world business documents and 2,855 expert-annotated rules. Unlike prior datasets centered on narrow service scenarios, BREX spans over 30 vertical domains, covering scientific, industrial, administrative, and financial regulations. We further propose ExIde, a structure-aware reasoning framework that investigates five distinct prompting strategies, ranging from implicit semantic alignment to executable grounding via pseudo-code generation, enabling explicit modeling of rule dependencies and providing an out-of-the-box framework for different business customers without finetuning their own LLMs. We benchmark ExIde using 13 state-of-the-art LLMs. Our extensive evaluation reveals that: (1) Executable grounding serves as a superior inductive bias, significantly outperforming standard prompts in rule extraction; and (2) Reasoning-optimized models demonstrate a distinct advantage in tracing long-range dependencies and non-linear rule dependencies compared to standard instruction-tuned models. The code and dataset are available at: https://github.com/oYoungCo/Business-as-Rulesual.

### 中文概述
论文研究"把非结构化的业务文档（规章、手册、合规说明）自动变成结构化的**规则流（rule flow）**"：先抽取原子规则（condition–action 对），再恢复规则之间的依赖关系（Sequential / Conditional / Parallel）。作者把"自然语言规章和可执行控制流之间的差距"叫 **Logic Gap**，并提出 BREX benchmark（409 篇文档、2,855 条专家标注规则）与框架 ExIde（五种提示策略 + 列表式依赖推理），在 13 个 LLM 上评测。

### 核心要点
- **问题**：已有工作多抽取线性的 action flow（菜谱、教程），而真实规章里动作由**条件**而不是前一个动作触发，还有嵌套条件和并行要求；缺少同时标注原子规则和规则间依赖的大规模 benchmark。
- **方法**：两个东西配合使用。
  - **BREX benchmark**：用来评测"业务文档 → 规则流"这件事做得好不好的数据集。409 篇真实业务文档，每篇由专家标出原子规则（⟨Condition, Action⟩）和规则之间的依赖（Sequential / Conditional / Parallel）。
  - **ExIde framework**：用来在 BREX 上完成这件任务的两阶段提示方法，不需要训练模型。
    - Stage I 抽规则：从文档里抽出 N 条原子规则。论文设计了 **5 种 prompt（P1–P5）作为互相替代的方案**，用来比较哪一种抽规则更好；它们不是依次叠加使用，一次只用一种。区别在于抽取前让模型先做什么中间推理，其中 P5 是先把文本翻成伪代码再抽。
    - Stage II 建依赖图：只有 1 个 prompt，不分版本。把整篇文档和所有候选规则对放进一次提示，让模型一次判断每对规则是 Sequential / Conditional / Parallel / None。
- **结果**：P5（先生成伪代码再抽规则）平均分最高，但优势很小；依赖识别整体很难（最好 F1 0.764，多数 0.5–0.6），reasoning 模型略优；LoRA 微调后依赖识别反而大幅下降。
- **意义**：给"文档 → 结构化流程"这一上游步骤提供了 benchmark，对应前面几篇论文里"流程从哪来"的问题。

## 研究背景与问题
企业靠大量自然语言规章管理流程（实验室安全规程、金融合规等），这些文档给人看，机器执行要结构化表示，目前主要由领域专家用规则引擎或工作流语言（可执行代码、BPMN）手工翻译，费人、易错、难随政策更新维护。

- **Logic Gap**
  - 意思：自由文本规章与自动化系统需要的、依赖条件的可执行控制流之间的差距。
  - 例子：金融合规里"币种为 USD 时，选择汇款类型"这条规则是被**条件**触发的，不是被上一个动作触发；后续检查还可能有条件地触发或并行执行。
- **action flow vs rule flow**
  - 意思：action flow 描述发生哪些动作；rule flow 描述在什么逻辑条件下动作被允许或受约束。
  - 例子：菜谱是"先切菜再下锅"的线性动作序列；银行手册是"若币种是 X 则选 Y，选完再提供金额"的条件规则网络。

适用边界（作者明确）：标注 schema 只有三种依赖类型和预设的逻辑算子；时间约束、异常处理、概率规则未建模；框架靠提示词，不做端到端训练。

## 方法概述

### 核心方法
**任务定义**：给一份业务文档，抽取一组 condition–action 规则，并重建依赖图。

**规则表示**：每条规则是原子的 ⟨Condition, Action⟩；Condition 是三元组 ⟨Slot Type, Logical Operator, Reference Value⟩。

| 组成               | 意思                                                        | 例子（笔记作者按论文表 1 的银行场景改写） |
| ---------------- | --------------------------------------------------------- | ---------------------- |
| Slot Type        | 用户输入的类别                                                   | 币种                     |
| Logical Operator | 槽位和参考值的关系，取值：contains、equal to、less than、greater than、≤、≥ | contains（"支持包括……"）     |
| Reference Value  | 枚举值或数值                                                    | RMB、USD、JPY、GBP、HKD    |
| Action           | 条件满足后触发的操作；流程终止时为 None                                    | 选择现金或汇款类型              |

**三种依赖**：
- Sequential：规则 A 必须先于规则 B 执行。
- Conditional：规则 A 的不同取值触发不同的后续规则（分支）。
- Parallel：多条规则必须同时执行。

**BREX 构建流程**（图 1）：
1. 数据收集：30+ 垂直领域的公开真实文档，加少量用 Gemini 2.5 Pro 生成的合成文本，只用来补充嵌套条件、并行约束等稀缺结构，并经专家严格筛选修订。
2. 专家标注：3 位领域专家按统一 schema 标注原子规则和依赖。
3. 多阶段验证：另一组 3 位专家复核，含糊规则被丢弃或重标。

**ExIde 两阶段**（图 2）：
- **Stage I：Structure-Aware Rule Extraction**。论文设计了 5 种 prompt（P1–P5），**互相替代、一次只用一种**，在 13 个模型上比较哪一种抽规则更好（指标：NER F1 和逻辑算子 F1）。五种 prompt 共享同一个输出 schema，区别只在输出前的中间推理结构。（五种 prompt 共用的基本模板：附录图 10；各 prompt 原文见下，均为附录末尾的图。）
  - P1 Implicit Semantic Alignment：先写自然语言解释再输出结构化结果，源文本与规则的对应隐含在注意力里（baseline）。（prompt 原文：附录图 11。）
  - P2 Explicit Traceability：强制每句解释与一条规则一一对应（"这句对应规则……"），减少无依据的规则。（prompt 原文：附录图 12。）
  - P3 Clarified Contextual Input：在 P1 上把输入字段明确标为"Business Text Input"。（论文没有单独附图，只在 4.1 节文字里说明。）
  - P4 Logic-Aware Definition Injection：把逻辑算子的细节约束（如 contains 与 equal 的区别）放进提示。（prompt 原文：附录图 13。）
  - P5 Executable Grounding：先用少量过程原语（如 `select_from()`、`execute_action()`）把文本翻成**伪代码**，再从伪代码抽规则，让嵌套条件和控制流尽早被解析。（prompt 原文：附录图 14。）
- **Stage II：Dependency Graph Reconstruction**。输入 N 条规则，判断每两条规则之间的关系，输出依赖图 $G$。只有 1 个 prompt，表 4 比较的是 13 个**模型**的依赖识别 F1，没有像 Stage I 那样比较不同 prompt。（论文没有明确说这 N 条规则是专家标注的还是 Stage I 抽出来的。）
  - **要做什么**：每个有方向的规则对 $(r_i, r_j)$ 四选一：Sequential / Conditional / Parallel / None（无关系）。结果存成 N×N 的**邻接矩阵**（行 = 起点规则，列 = 终点规则，格子里填关系类型）。
    - 意思：邻接矩阵就是把"谁和谁有什么关系"列成一张表；规则可以分叉、汇合，不是一条链。
    - 例子（文本来自论文表 11 的定期存款，论文说这段标了 15 条规则；下面的步骤划分和依赖是笔记作者按原文推的，论文没有给这段的依赖图）：
      原文大意：先选存款类型（整存整取 / 零存整取）。选整存整取 → 选币种，再选存期（1/3/5 年）。选零存整取 → 设每月存入额（≥50 元），再选存期（1/3 年）。信息确认后填具体存款金额；单笔超过 20 万要另外提供资金来源证明。最后可选是否自动续存、选利率变动通知方式，"这两项可以同时选"。
      为了好读，把枚举值合并成 9 个步骤（真正的标注会按取值拆成更多条，所以论文有 15 条）：
      A 选存款类型　B 选币种　C 选存期（1/3/5 年）　D 设每月存入额　E 选存期（1/3 年）　F 填存款金额　G 提供资金来源证明　H 自动续存 Yes/No　I 通知方式

| 起点 → 终点     | 关系          | 原文依据                                                |
| ----------- | ----------- | --------------------------------------------------- |
| A → B       | Conditional | 选整存整取才走这条                                           |
| A → D       | Conditional | 选零存整取才走这条                                           |
| B → C       | Sequential  | "Next, the customer selects the deposit term"       |
| D → E       | Sequential  | "After that, the customer selects the deposit term" |
| C → F       | Sequential  | "Once the above information is confirmed"           |
| E → F       | Sequential  | 同上，两个分支汇合到 F                                        |
| F → G       | Conditional | 金额 > 200,000 才要证明                                   |
| F → H、F → I | Sequential  | "Finally, the customer may choose…"                 |
| H → I、I → H | Parallel    | "These two services can be selected simultaneously" |
      读法：9 个步骤共有 9×8 = 72 个有方向的格子，上面只有 11 个不是 None（表里 9 行，F→H/F→I 算两格、H↔I 算两格）。图形状是：A 分叉成两条路，两条路在 F 汇合，F 再分出 G，最后 H、I 并行。
  - **Listwise Contextual Reasoning**：把整篇文本和**所有**候选规则对放进**一次提示**，让模型一次填完整张矩阵，而不是每对规则单独问一次（pairwise）。（prompt 原文：附录图 15。）
    - 意思：单独问时模型看不到其他规则，每次判断互相独立；一次看全局，分叉、汇合、并行这类需要对照多条边才能确定的结构才能对得上。
    - 例子（笔记作者构造，论文没有给过逐对出错的例子）：单独问 H → I，两条规则在原文里一前一后，容易答成 Sequential；一次看全局时能同时看到"这两项可以同时选"和它们都挂在 F 后面，才标成 Parallel。
    - 输出：一次给出整张矩阵。下面是上面存款例子的完整邻接矩阵（行 = 起点，列 = 终点）：

| 起点 \ 终点 | A   | B    | C   | D    | E   | F   | G    | H   | I   |
| ------- | --- | ---- | --- | ---- | --- | --- | ---- | --- | --- |
| **A**   | –   | Cond | ·   | Cond | ·   | ·   | ·    | ·   | ·   |
| **B**   | ·   | –    | Seq | ·    | ·   | ·   | ·    | ·   | ·   |
| **C**   | ·   | ·    | –   | ·    | ·   | Seq | ·    | ·   | ·   |
| **D**   | ·   | ·    | ·   | –    | Seq | ·   | ·    | ·   | ·   |
| **E**   | ·   | ·    | ·   | ·    | –   | Seq | ·    | ·   | ·   |
| **F**   | ·   | ·    | ·   | ·    | ·   | –   | Cond | Seq | Seq |
| **G**   | ·   | ·    | ·   | ·    | ·   | ·   | –    | ·   | ·   |
| **H**   | ·   | ·    | ·   | ·    | ·   | ·   | ·    | –   | Par |
| **I**   | ·   | ·    | ·   | ·    | ·   | ·   | ·    | Par | –   |
  - **O(N²) 的代价**：有方向的候选对共 N(N−1) 个，量级 O(N²)。作者认为平均每篇只有约 7 条规则，一个提示装得下，调用一次比逐对调用几十次便宜，所以可行。
    - 意思：规则越多，候选对增长越快，提示越长。
    - 例子：平均 7 条规则 42 对；上面存款例子只有 9 个合并步骤就是 72 对，论文标的 15 条是 210 对；20 条是 380 对。
  - 注意：O(N²) 可行是作者的论证，论文没有做 listwise 与逐对分类的对比实验；"避免矛盾"同样是作者的说法，不是实验结论。

### 关键公式
论文没有需要推导的公式。核心是形式化表示：规则 $r_i=(c_i,a_i)$，条件 $c_i=\langle s_i,j_i,v_i\rangle$（slot、operator、value）；依赖图 $G=(V,E)$，节点是规则，边标注 Sequential / Conditional / Parallel / None。

### 方法架构与图片
![[page2_fig1.png|800]]

图 1：BREX 构建流程。左：30+ 垂直领域的真实文档加 LLM 合成扩充；中：专家把文本标注成 ⟨Condition, Action⟩（Slot、Logic、Value、Action）；再经质量复核和一致性评估（ICC 0.892、Kappa 0.901）；右：输出 409 篇文档、2,855 条规则及依赖图。

![[page5_fig1.png|800]]

图 2：ExIde 总览。非结构化业务文档经五种提示策略（P1–P5）得到原子规则集合 $R=\{r_i\}$，再由 Dependency Reasoning 推断全局规则关系（Sequential / Conditional / Parallel）。

## 实验与证据
- **评测 benchmark**：只有 BREX（作者自建）一个 benchmark。
  - 规模：409 篇文档、2,573 个句子、2,855 条规则、95,779 个 token，平均约 7 条规则/篇；30+ 领域（科研管理、工业制造、行政审批、金融合规等）。
  - 依赖分布：Sequential 2,330、Conditional 722、Parallel 417，非线性依赖约占 33%。
  - 数据质量（BREX 构建流程见「核心方法」，图 1）：
    - 流程回顾：收集（30+ 领域真实文档 + 少量合成）→ 3 位专家标注 → 另一组 3 位专家复核，所以数据质量检查是流程最后一步，在复核之后，用两个一致性指标验证。
    - **ICC**（Intraclass Correlation Coefficient）= 0.892
      - 意思：衡量多人给**数值打分**时是否一致（0–1，越高越一致）。这里是 3 位标注者给文本的 Readability / Accuracy / Clarity / Simplicity / Usability 打 1–5 分，检查"收集来的文本"本身好不好。
      - 例子：3 人给同一篇文档的"清晰度"分别打 4、4、5，ICC 高；打 1、3、5 则很低。
    - **Fleiss' Kappa** = 0.901
      - 意思：衡量多人给**类别标签**时是否一致，并扣除"碰巧一致"的部分（1 = 完全一致，0 = 和瞎猜差不多）。依赖图没有现成的一致性指标，所以作者把规则标注投影成 NER 的 BIO 标签（Slot Type / Reference Value / Action），让 3 位专家各标一遍再算，检查"专家标注"是否可靠。
      - 例子：句子里每个词要标成 B-Slot / I-Slot / B-Value / O，3 人对每个词选的标签越相同，Kappa 越高。
  - SFT 实验的划分：300 条训练、109 条测试。
- **模型**：13 个 LLM，5 个家族（GPT、Gemini、Kimi、DeepSeek、Qwen）：Gemini-2.5-flash / pro，GPT-5-mini / GPT-5，Kimi-k2-Instruct / Thinking，DeepSeek-3.1 / 3.2-exp / R1，Qwen3-30B 与 235B 各 Instruct / Thinking。
- **指标**：Stage I 用 NER F1（Slot Type、Reference Value、Action）和逻辑算子分类 F1；Stage II 用依赖分类 F1（四类）。
- **主要结果**：
  - 规则抽取（NER F1，13 模型平均）：P1 0.879、P2 0.879、P3 0.873、P4 0.871、**P5 0.882**。
  - 逻辑算子分类（F1 平均）：P1 0.840、P2 0.846、P3 0.835、P4 0.773、**P5 0.850**。
  - 依赖识别（F1）：Gemini-2.5-pro 最高 0.764；Gemini-2.5-flash 0.629；DeepSeek-3.2-exp 0.601；多数在 0.5–0.6；最低 Qwen3-30B-Instruct 0.425。
  - reasoning vs instruction：Qwen3-30B Thinking 0.485 对 Instruct 0.425（+0.060）；Qwen3-235B 0.586 对 0.554；Kimi-k2 0.500 对 0.482；作者称优势在 Conditional 与 Parallel 上最明显（图 3 雷达图）。
- **消融与稳健性**：
  - 规则密度（图 4 左）：文档规则越多所有方法越差，P5 曲线相对平缓，作者认为伪代码是结构锚点，缓解 lost-in-the-middle；P4 在 Med-Low 档方差大（把多个 equal 条件过度合并）；在高密度档部分模型出现 P4 优于 P5 的反转（P5 的伪代码会生成多余的元变量规则，降低精度）。
  - 推理复杂度（图 4 右）：规则对数增多时，instruction-tuned 模型明显下滑，reasoning 模型曲线更平。
  - **SFT 对比**（表 5，Qwen3-30B-Instruct + LoRA，300 训 / 109 测）：

| 任务 | ExIde（提示） | SFT | 差值 |
|---|---:|---:|---:|
| Rule Extraction | 83.90 | 84.70 | +0.80 |
| Logic Operator Classification | 63.60 | 72.40 | +8.80 |
| Dependency Relation Identification | 40.84 | 24.16 | −16.68 |

  作者的解释："rule pair dropping"——SFT 模型处理变长的候选规则对列表时中途放弃、截断输出或漏掉规则对；ExIde 对 N 个输入对总是给出 N 个判断。

## 深度分析

### 核心贡献与创新
**作者主张**：(1) 提出 rule-centric 而不是 action-centric 的建模，并发布带规则间依赖标注的跨领域 BREX benchmark；(2) ExIde 在不微调的情况下做规则流建模；(3) 13 模型的系统评测：executable grounding 是更强的归纳偏置，reasoning 模型更擅长长程、非线性依赖。

**基于证据的判断**：
- BREX 的价值是明确的：条件–动作原子化 + 三类依赖 + 30 多个领域，补上了"规则之间怎么连"这块 benchmark 空白；双组专家标注和复核流程较规范。
- 最有信息量的发现是 SFT 结果：微调改进了语义抽取，却在全局依赖推理上变差。这说明"理解文本"和"重建全局结构"是两类能力，也提示不能只靠小数据微调。

### 优势
- 问题定义清楚，Logic Gap、rule flow 与 action flow 的区分有说服力。
- 数据覆盖广，且报告了标注质量指标和依赖分布。
- 评测了 13 个模型，开放模型和闭源模型都有，并分析了规则密度、推理复杂度和 SFT。
- 代码和数据已公开。

### 局限与风险
- 作者明确局限：含少量合成数据，可能不完全反映真实规章的语气；依赖推理靠提示词，规则很多时 O(N²) 会增长；只有三种依赖和预设算子，没有时间约束、异常处理、概率规则。
- 进一步推断（分析，非论文原话）：
  - **"P5 显著优于标准提示"的说法偏强**：规则抽取平均 NER F1 上 P5 为 0.882，P1/P2 是 0.879，差 0.003；逐模型看，Gemini-2.5-pro 是 P1（0.916）高于 P5（0.899），GPT-5 是 P2（0.880）高于 P5（0.873）。论文没有给显著性检验或方差，这点差异很难说明"显著"。逻辑算子上 P5（0.850）也只比 P2（0.846）高一点。
  - **"reasoning 模型优势明显"也有出入**：DeepSeek-3.1 与 R1 几乎相同（0.575 与 0.576）；Gemini-2.5-flash（0.629）高于所有开放的 Thinking 模型；模型到底按什么标准归类为 reasoning，论文没有明确列出。
  - **指标偏"字符串匹配"**：作者自己承认字符串类指标会把"合并多个 equal 条件"的输出判成召回骤降，使 P4 出现重尾方差；也不清楚"规则正确"是否等价于"执行语义正确"。
  - **依赖识别整体偏难且未做上界**：最好 F1 0.764，多数 0.5–0.6，没有人类或规则 baseline可比；"None"类占比大时 F1 的口径（宏平均还是微平均）论文没说明。
  - **一致性指标没覆盖到最关键的部分**：
    - Kappa 是在代理任务（NER 的片段边界与类型）上算的，不是在依赖图上算的。0.901 只能说明规则的"切分和类型"标得一致，不能说明 Sequential / Conditional / Parallel 依赖的标注也一致，而依赖正是这篇论文的核心。
    - ICC 测的是文本质量（好不好读），不是规则标注是否准确。
    - 两个指标都只在数据子集上算，论文没有说子集有多大。
  - **SFT 对比设置较弱**：只训练 300 条、测试 109 条，只用一个模型，Table 5 中 ExIde 的数值（如算子 63.60）与主表 Qwen3-30B-Instruct 的数值不同（可能是测试子集不同，论文未说明）；"rule pair dropping"是输出格式问题，更像训练数据长度分布问题，不一定代表推理能力变差。
  - **与执行脱节**：只评估抽取是否与专家标注一致，没有把抽出的规则流真的跑起来验证；ACL 论文里作者也强调不能作为自动决策系统。
  - 仓库没有显式许可证（GitHub 显示 no license），数据复用条件需自行确认。

### 适用场景
- **适合**：想评估或改进"业务文档 → 规则/流程结构化"的研究；需要现成的、带依赖标注的规章类数据；用提示词在不微调下做规则抽取的原型。
- **不适合**：直接把抽出的规则流用于合规决策（需要人工审核）；需要时间约束、异常和概率规则的场景；规则数量很大的文档（候选对 O(N²)）。

## 相关工作与技术路线
- PAGED（Du et al., ACL 2024）— 大规模过程图抽取 benchmark，用合成的 data-to-text，语言偏简单；BREX 强调真实规章文本。
- 过程文本理解（Quishpi 等 2020、Bellan 等 2022、Pyrih 等 2025）— 多为线性或弱结构的 action flow；BREX 改成 rule-centric。
- Chain-of-Code 类思想 — ExIde 的 P5 把代码形式当作中间表示（先伪代码再抽规则）。
- [[Compile,_Then_Page_Executable_SOP_Programs_and_a_Capability-Gated_Runtime_for_Procedural_LLM_Agents|Compile, Then Page]] — 它假设 SOP 已经是机器可读的依赖树（and/or/gate/chain），并指出自然语言 SOP 未评估；BREX/ExIde 做的正是"自然语言 → 依赖结构"这一**上游**步骤，二者可以接起来（Sequential、Conditional、Parallel 与那里的 chain、gate、and 大致对应，是我的类比，两篇论文没有建立这个对应）。
- [[Blueprint_First,_Model_Second_A_Framework_for_Deterministic_LLM_Workflow|Blueprint First, Model Second]] — 它让专家或 LLM 把流程写成代码蓝图；BREX 提供了"从文档自动得到结构化规则"的 benchmark，可作为蓝图起草环节的评测素材（我的联想，论文未提及）。

## 综合评价

| 维度 | 分数 | 理由 |
|---|---:|---|
| 创新性 | 6/10 | rule-centric 的问题定义和带依赖标注的 benchmark 有价值；ExIde 本身只是提示策略的变体，方法新意有限 |
| 技术质量 | 5/10 | 评估口径（字符串 F1、宏微平均）不透明，部分结论（P5 显著更优）缺少显著性检验 |
| 证据充分性 | 6/10 | 13 模型、多维度分析、有 SFT 对比；但差异小、无方差/检验，SFT 设置弱 |
| 写作质量 | 6/10 | 结构清晰，动机明确；个别表述偏强，且重复（Section 5.4 出现重复段落） |
| 实用性 | 6/10 | benchmark 和代码可直接使用，但抽取质量离可直接落地还有距离，依赖识别尤其弱 |

**综合：约 5.8/10**（各维度均值，主观评分）。

## 我的笔记

%% 在这里添加个人笔记 %%

## 外部资源
- 论文：[ACL Anthology](https://aclanthology.org/2026.acl-long.1625/)（[PDF](https://aclanthology.org/2026.acl-long.1625.pdf)，DOI 10.18653/v1/2026.acl-long.1625）
- 代码与数据：[github.com/oYoungCo/Business-as-Rulesual](https://github.com/oYoungCo/Business-as-Rulesual)（含 `business_rules_extraction`、`dataset`、`assets`、`requirements.txt`；论文摘要中给出；GitHub 显示无许可证，4 stars）
- benchmark：BREX（Business Rule Extraction Benchmark）
