---
document_id: "2508.02721"
arxiv_id: "2508.02721"
source_type: "arxiv"
source_url: "https://arxiv.org/abs/2508.02721"
title: "Blueprint First, Model Second: A Framework for Deterministic LLM Workflow"
authors: "Libin Qiu, Yuhang Ye, Zhirong Gao, Xide Zou, Junfu Chen, Ziming Gui, Weizhi Huang, Xiaobo Xue, Wenkai Qiu, Kun Zhao"
published: "2025-08"
domain: "智能体"
tags:
  - "论文笔记"
  - "智能体"
  - "arxiv"
  - "确定性工作流"
  - "SOP"
  - "Source Code Agent"
repo: "not found"
quality_score: "6.6"
related_papers:
  - "2607.11346"
created: "2026-11-02"
updated: "2026-11-02"
status: "analyzed"
---
# Blueprint First, Model Second: A Framework for Deterministic LLM Workflow

## 核心信息
- **文档 ID**：2508.02721
- **来源类型**：arxiv
- **作者**：Libin Qiu, Yuhang Ye, Zhirong Gao, Xide Zou, Junfu Chen, Ziming Gui, Weizhi Huang, Xiaobo Xue, Wenkai Qiu, Kun Zhao
- **机构**：阿里巴巴集团（杭州）
- **发布时间**：2025-08（由 arXiv ID 推断；当前为 v2，2026-06-16）
- **会议/期刊/发布方**：arXiv preprint (cs.SE)
- **链接**：[arXiv](https://arxiv.org/abs/2508.02721) | [PDF](https://arxiv.org/pdf/2508.02721)
- **引用**：--

## 摘要与核心要点

### 原文摘要
While powerful, the inherent non-determinism of large language model (LLM) agents limits their application in structured operational environments where procedural fidelity and predictable execution are strict requirements. This limitation stems from current architectures that conflate probabilistic, high-level planning with low-level action execution within a single generative process. To address this, we introduce the SOURCE CODE AGENT framework, a new paradigm built on the "Blueprint First, Model Second" philosophy that decouples workflow logic from the generative model. An expert-defined operational procedure is first codified into a source code-based Execution Blueprint, which is then executed by a deterministic engine. The LLM is strategically invoked as a specialized tool to handle bounded, complex sub-tasks within the workflow, but never to decide the workflow's path. We evaluate on the TravelPlanner benchmark for constraint-aware travel planning. The SOURCE CODE AGENT achieves a 35.56% final pass rate, a 97.6% improvement over the state-of-the-art ATLAS baseline (18.00%) on the same Claude-Sonnet-4 backbone. Critically, it reduces constraint violations by 96.0% (11 vs 275) while improving execution efficiency by 27.1% (10.2±0.7 steps vs 14.0). Two production incident-diagnosis deployments and additional results on ScienceWorld and ALFWorld confirm that the architecture transfers beyond travel planning to procedurally well-defined, constraint-intensive workflows. Our work enables the verifiable and reliable deployment of autonomous agents in applications governed by strict procedural logic.

### 中文概述
论文提出 Source Code Agent（SCA）：由专家把操作流程写成**源代码形式的"执行蓝图"**（如 Python 脚本），由确定性引擎执行；控制流、分支、工具调用和约束校验全部由代码决定，LLM 只在蓝图的特定节点被当作"工具"调用，负责有边界的子任务（解析日志、选择餐厅、总结输出），**从不决定工作流走向**。在 TravelPlanner 上最终通过率 35.56%（ATLAS 为 18.00%），常识约束违规从 275 降到 11；并有两个阿里内部生产部署案例（Java OOM 诊断、Android 崩溃分诊）以及 ScienceWorld / ALFWorld 的泛化结果。

### 核心要点
- **问题**：端到端 LLM agent 把"做什么"与"怎么做"混在一次概率生成里，执行路径不可预测、约束靠提示词遵守，难以用于严格流程场景。
- **方法**："Blueprint First, Model Second"——代码是控制者，LLM 是被调用的工具；约束写成程序化检查（如 `raise ConstraintViolationError`），无法被模型绕过。
- **结果**：TravelPlanner 测试集（1000 任务）Final Pass 35.56%、平均 10.2 步；生产中 Java OOM 的 MTTR 约 30 分钟→5 分钟，Android 崩溃分诊约 4 小时→8 分钟，误报率 30%→8%。
- **意义**：流程定义明确、约束密集的任务，把控制流从 LLM 里拿出来可以同时提升可靠性、效率和可审计性；开放式任务仍应使用通用 agent。

## 研究背景与问题
当前 agent（ReAct、Reflexion、多 agent 如 ATLAS/MetaGPT）把战略规划和低层动作选择都交给 LLM 的概率输出，导致执行路径不可预测。作者（第 III-A 节）列出三类非确定性来源：

- **Semantic Ambiguity（语义歧义）**
  - 意思：自然语言指令本身含糊，不同次运行可能被理解成不同的标准。
  - 例子：提示词写"费用太高的订单需要主管审批"。"太高"到底是多少？
    - 第一次运行，按 5000 元算，6000 元的订单要审批。
    - 第二次运行，按 10000 元算，同一订单不用审批。
    - 对比代码 `if amount > 5000`，阈值写死，每次判断一致。
- **Probabilistic Decision-Making（概率式决策）**
  - 意思：下一步动作是从 token 的概率分布中**采样**得到的，不是查表得出的唯一答案，所以同样输入可能走出不同路径。
  - 例子：退货申请进来，agent 拿到订单信息：
    - 第一次运行，抽中"先查是否在 7 天内"，继续走流程。
    - 第二次运行，抽中"先联系客户确认原因"，走了另一条路。
    - 第三次可能直接批准退款，跳过了检查。
- **Accumulated Drift（误差累积）**
  - 意思：多步任务中，前面一步的小错误会传给后面的步骤并被放大，最终级联失败。
  - 例子：一个 7 天的旅行规划：
    - 第 1 步，订好第 1 天到第 3 天在巴黎的酒店。
    - 第 2 步，模型记错了天数，当成只住 2 晚（小错误）。
    - 第 3 步，第 3 天的行程被安排到另一个城市。
    - 第 4 步，后面的交通、餐厅都按错误的城市排。
    - 每一步单看都合理，整体行程却违反了"当前城市"约束，而且很难看出问题出在第 2 步。

（以上三个例子是笔记作者为说明而举，非论文原话。）

论文引言用的例子是生产环境 Java OutOfMemoryError：专家有固定排查流程（`jstat` → 老年代满则 `jmap` → 分析 dump），而通用 agent 可能发出宽泛探索式命令，甚至在仍有流量的主机上做堆转储。

任务类型：论文面向的是流程定义明确、约束密集的一般操作流程（旅行规划、退款与审批、合规、运维诊断等），不限于软件开发。

适用边界（作者明确）：流程稳定、规则密集、可靠性优先于适应性的任务；开放式、快速变化的临时问题仍更适合通用 agent。

## 方法概述

### 核心方法
五层架构（图 2）：
1. **用户配置层**：可视化界面组装自定义源码、LLM、知识库和 MCP 工具；支持由 LLM 根据自然语言描述生成初始代码骨架，以及"少样本案例库（Few-Shot Case Library）"模板。
   - 自定义源码 = 蓝图：控制流、分支、工具与 LLM 调用、校验都在代码里；LLM 只在被调用的位置工作。
   - 来源：离线做好，运行时不再生成。LLM 起草骨架（Agent-Generated Scaffolding）、从模板库改，或人手写，最后由人审改定稿。论文称 53 个内部工作流中超过 90% 在 5 轮人机对话内定稿。
   - 例子（论文 TravelPlanner，约束"不重复餐厅"）：代码维护已选餐厅并过滤，LLM 只负责从剩下的里选。
     ```python
     used_restaurants = set()
     def select_restaurant(city, cuisine):
         candidates = search_restaurants(city, cuisine)
         available = [r for r in candidates if r.id not in used_restaurants]
         if not available:
             raise NoValidOptionError()
         selected = llm_choose(available)      # LLM 只做"选哪个"
         used_restaurants.add(selected.id)
         return selected
     ```
   - 规模：HR 请假 87 行、Android 崩溃分诊 290 行、Java OOM 诊断 332 行；论文未公开完整代码。
2. **组件化 Agent SDK**：Knowledge-Base API（RAG）、LLM API（统一接口）、MCP Server API；支持**组合工具**（composite tools），把多个细粒度调用封装成一个，减少 LLM 决策点（零售案例中工具调用 11→2，减少 81.8%）。
   - 核心：把一串固定顺序的调用打包成一个函数，顺序由代码执行，不由 LLM 逐步决定。
   - 例子：
     - 无组合工具：LLM 自己编排 `check_stock()` → `get_price()` → `apply_discount()`，每步都要决策（调哪个、顺序、参数），增加延迟和 token。
     - 有组合工具：封装成 `process_return_request(item_id, user_id)`，子步骤在函数内部按固定顺序跑完，LLM 只决定何时调用、如何解读返回结果。
3. **控制层（Control Layer）**：负责会话与交互，不负责流程控制（流程由蓝图代码决定），也不约束 LLM 在任务内的推理。
   - 请求接入：为用户和外部系统提供统一入口，校验请求。
   - 会话与历史：用 Session_Id / User_Id 区分对话，取出历史补进请求，支持多轮。
   - 异常监控。
   - 流式返回：用 SSE 把结果逐步推给前端。
   - 例子（非论文原话）：运维人员用聊天界面调用 Java 诊断 agent。
     1. 发来"诊断主机 A 的内存溢出"，控制层校验请求、分配 Session_Id，交给执行器跑蓝图。
     2. 蓝图运行中，状态经 SSE 实时推送（如"正在调用 jstat…"）。
     3. 追问"它现在还有流量吗？"，控制层按 Session_Id 补上历史，agent 因此知道"它"指主机 A。
     4. 之后按分支走哪条路由蓝图代码决定，控制层不参与。
4. **源码执行器（Source Code Executor）**：执行蓝图代码本身（不是 tool，也没有转换步骤），负责让它安全、可管控地跑起来；蓝图里调用 tool 和 LLM 是代码自己的事。
   - 启动运行时：按代码类型拉起 Python / Node.js / Java 环境。
   - 资源治理：限制 CPU、内存、网络配额。
   - 遥测追踪：记录时间戳、输入输出、结果、耗时，用于端到端追溯。
   - 错误处理：判断是否可恢复，可恢复的失败就重试。
   - 例子（非论文原话）：Java 诊断蓝图被交给执行器。
     1. 执行器在沙箱里启动 Python 解释器运行蓝图，并限定内存和网络配额。
     2. 蓝图跑到 `jmap` 时工具调用超时，执行器判断为可恢复错误，重试一次。
     3. 全程时间戳、输出、耗时被记录，事后可回放这次诊断的步骤。
     4. 蓝图跑完，执行器把结果捕获成结构化数据（如 JSON）交回控制层。
5. **沙箱运行环境**：网络与资源隔离，集中式依赖管理。

设计原则：代码规定**何时**调用模型、对输出有什么约束；模型自己决定**如何**完成子任务。作者强调三点：关键节点用代码给出精确语义（`if old_gen_usage > 0.9`）；同样输入走同样路径；每次 LLM 调用只带有任务范围的提示词，降低对全局工作流的感知需求（内部测量最多省 66.7% token）。

### 关键公式
论文没有形式化模型或定理，这里不涉及公式。核心机制用代码表达，例如：

```python
if jstat_result.old_gen_usage > 0.9:      # 分支由确定性代码决定
    heap_dump = execute_tool("jmap", pid=process_id)
    analysis = run_llm(session_id, build_heap_analysis_prompt(heap_dump))   # LLM 只做分析
else:
    analysis = run_llm(session_id, build_gc_analysis_prompt(jstat_result))
```

输出校验也由代码完成（检查 JSON 结构与 `intent` 取值，内容部分不约束）。

### 方法架构与图片
![[intro_page1.png|800]]

图 1：左边是 Source Code Agent——Python 代码决定 `call jstat` 之后走哪条分支（`build a prompt → jmap → hprof` 或 `build b prompt → jstack`），LLM（Qwen）在各节点被调用；右边是其他 agent——把全部步骤和条件写进一段用户提示词，由 LLM 自己解释并决定走哪条路。

![[overview_page1.png|800]]

图 2：系统总览——Source Code Agent Controller、Code Executor、配置层与组件层（知识库、MCP、LLM API）及 Python/Node 运行环境。

![[source-code-example_page1.png|800]]

图 3：一个以伪代码写的 Source Code Agent：接收参数、校验必填项、意图识别、生成提示词、调用模型、组装结果。

## 实验与证据
- **数据集/任务**：TravelPlanner（180 验证 / 1000 测试查询）；泛化：ScienceWorld（30 个任务）、ALFWorld（134 个 unseen 任务）；两个生产部署案例。
- **基线**：ReAct、ReAct+Reflexion、ReAct+EvoAgent、PMC、ATLAS（作者称 Google 的 SOTA 多 agent 旅行规划方法）；公平性消融另加 CodeAct。
- **指标**：Delivery Rate、Commonsense / Hard Constraint 的 Micro 与 Macro、Final Pass、平均步数。主实验统一 Claude-Sonnet-4、temperature 0.7、3 个随机种子。
- **主要结果**（表 I，1000 任务）：
  - Final Pass：SCA 35.56%，ATLAS 18.00%，PMC 12.12%，ReAct 10.40%，ReAct+Reflexion 9.13%。
  - Commonsense：Micro 92.71%（ATLAS 78.88%）、Macro 54.44%（ATLAS 31.00%）。
  - Hard Constraint：Micro 49.76%、Macro 38.89%，与 ATLAS（49.43 / 42.00）相当；硬约束违规 336 次，**高于** ATLAS 的 265。
  - 步数：10.2±0.7，对比 ReAct 14.0、ReAct+Reflexion 16.1（ATLAS 未报告步数）。
  - 常识违规：SCA 11、ATLAS 275、ReAct 835；四类（信息不完整、最短住宿天数、重复餐厅、不在当前城市）为 0。
- **消融与稳健性**：
  - **同知识公平对比**（表 II，100 任务分层子集，连续均分）：把 SCA 的全部约束写成自然语言放进基线提示词后，ReAct 17.00 / CodeAct 19.00 / ATLAS 24.50 vs SCA 37.20，差距归因于确定性执行而非知识量。
  - 行程天数（表 III）：3/5/7 天 SCA 成功率 52.1/38.7/24.6%，相对 ATLAS 提升 +133.6/+154.6/+215.4%。
  - 跨模型（表 IV，5 天行程）：GPT-5.2 34.10（对最佳基线 +18.54）、Claude-Sonnet-4 35.56（+17.56）、Gemini-2.5-Pro 40.00（+5.00）。
  - 泛化（表 V）：ScienceWorld 56.7% vs ReAct 43.3%（步数 16.6 vs 22.3；三种子均值 70.4±1.9%）；ALFWorld 97.0% vs 最佳反思基线 95.5%（步数 16.8 vs 15.7–16.1）。
  - 生产案例（表 VI）：Java OOM ≈40 簇/周，MTTR 30 分钟→5 分钟，约 60% 草稿无需修改即被采纳，前置条件使"对仍有流量主机做 heap dump"事故从每季约 6 起降到 0；Android 崩溃约 800 簇/天，MTTR 约 4 小时→8 分钟，误报 30%→8%，首次路由到正确团队约 85%。去掉 postcondition 后工作流会给空 payload 编出泄漏假设。
  - 编写成本：HR 请假流程 87 行、Android 蓝图 290 行、Java OOM 蓝图 332 行；53 个内部工作流中超 90% 在 5 轮人机对话内收敛到可通过的蓝图。

![[constraint_violations_stacked_page1.png|800]]

图 7：各方法在 1000 个测试任务上的常识违规分类堆叠图：ReAct 835、ATLAS 275、SCA 11。

## 深度分析

### 核心贡献与创新
**作者主张**：(1) "Blueprint First, Model Second"范式——代码控制流程，LLM 作被调用的工具；(2) 完整的五层工程框架与 SDK；(3) 约束以程序化检查实现，从架构上不可绕过；(4) TravelPlanner 上的显著提升，加上两个生产部署与泛化结果。

**基于证据的判断**：
- 最有价值的实验是表 II：把同样的约束文本注入基线仍落后 12.7 点，这比单纯的"我们的更好"更能说明"强制执行"与"知道规则"不同。
- 生产案例展示的关键收益来自**前置/后置条件**（拒绝在不安全或无依据状态下继续），而不是 LLM 推理更好，这一洞察实用且可迁移。
- 概念上并不新：本质是传统工作流/规则引擎 + LLM 节点。贡献更像是工程落地与实证，而不是新的算法。

### 优势
- 可靠性与可审计性：流程固定、失败可定位到具名节点；蓝图本身就是可检查的规范。
- 效率：步数更少、更稳定，token 开销下降（作者内部测量）。
- 证据面较广：公开基准、跨模型、跨复杂度、泛化基准加真实部署；对适用边界（ScienceWorld 开放任务收益收窄）有诚实说明。
- 与 [[Compile,_Then_Page_Executable_SOP_Programs_and_a_Capability-Gated_Runtime_for_Procedural_LLM_Agents|Compile, Then Page]] 对照：本文是"代码做控制者、LLM 做工具"的**硬控制**，后者是"LLM 做执行者、状态机只记账"的**软控制**，两者互补，且本文的约束无法被弱模型"忽略"。

### 局限与风险
- 作者明确局限：蓝图需要领域专家 + 编程能力（用 LLM 辅助与模板降低门槛）；不适用于开放式、快速变化的问题；硬约束违规反而比 ATLAS 多（336 vs 265），作者称加入显式参数校验器可补齐；ScienceWorld 中需要发现性探索的任务两种方法都失败。
- 进一步推断（分析，非论文原话）：
  - **数字自洽性存疑**：按表 III 的 3/5/7 天任务数（215/387/398）加权，ATLAS 总体约为 13.8%，与表 I 的 18.00% 对不上；表 IV 标题为"5 天行程"，但 Claude-Sonnet-4 的 SCA 35.56% 与 ATLAS 18.00% 恰与全测试集数字相同。图 7 中 ATLAS 的"Conflicting Transportations = 38"也恰好等于正文所说加提示后的数字（65→38），可能混用了不同设置。这些需要对照原始数据确认。
  - **基线口径不一致**：ATLAS 数字似乎来自其论文，SCA 在自己的 harness 中跑；步数只对 ReAct 系列报告。
  - **摘要"约束违规减少 96%"仅指常识约束**，同时硬约束违规增加，整体表述偏乐观。
  - **TravelPlanner 蓝图是人为针对该基准写的**（8 条常识 + 5 条硬约束），"同知识消融"能缓解但不能完全排除"为该基准量身定制"的可能；表 II 只用 100 任务、连续得分，并对其使用 McNemar 检验也不太匹配。
  - 生产数字为**四舍五入的内部运营数据，非对照实验**，无 A/B。
  - 这套方法把"理解 SOP"这一步转给了蓝图作者（或 LLM 辅助生成），蓝图质量成为新瓶颈；作者提到的"新控制流原语"和"规则自相矛盾"长尾也说明这点。
  - 代码"不能被绕过"只在 LLM 不直接执行副作用时成立；LLM 的输出内容（如选餐厅）仍有概率性，只靠结构校验兜底。

### 适用场景
- **适合**：Runbook 类运维诊断、客服/HR 审批流、合规流程、固定模板的操作任务，以及需要强审计、需要禁止特定动作顺序（如先下线节点再 dump）的场景。
- **不适合**：开放式探索、流程经常变化、没有专家愿意把流程写成代码的任务；需要 LLM 动态规划的任务。

## 相关工作与技术路线
- [[Compile,_Then_Page_Executable_SOP_Programs_and_a_Capability-Gated_Runtime_for_Procedural_LLM_Agents|Compile, Then Page]] — 同属"把 SOP 变成程序"的路线，但让 LLM 当执行者、运行时软约束；本文是硬控制。
- ReAct / Reflexion — 以 LLM 为控制器的主流 agent，本文的主要基线。
- ATLAS（arXiv:2509.25586）、PMC、MetaGPT — 多 agent 分工，仍靠自然语言提示协调，受"提示式约束"不确定性影响。
- Chain-of-Code、Logic-LM — 用代码/符号求解器做推理或验证，但 LLM 仍是主控；本文把整个工作流放进代码。
- CodeAct、SWE-agent — 把动作空间换成代码，LLM 仍决定下一步；作者称与本文互补，可在单个节点内使用。
- Plan-and-Solve — 先生成计划再执行，计划仍由 LLM 生成；本文的蓝图由专家编写。

## 综合评价

| 维度 | 分数 | 理由 |
|---|---:|---|
| 创新性 | 5/10 | 本质是代码工作流 + LLM 节点，概念成熟；价值在落地与实证 |
| 技术质量 | 7/10 | 系统分层清晰，有沙箱、执行器、前后置条件等工程细节；无形式化保证 |
| 证据充分性 | 6/10 | 基准 + 公平对比消融 + 泛化 + 生产案例较全，但存在数字不一致、基线口径与小样本统计问题 |
| 写作质量 | 7/10 | 结构清晰，主动讨论局限；部分表述（如 96% 降幅）略偏营销 |
| 实用性 | 8/10 | 直接可用的工程范式，有真实部署数据与编写成本 |

**综合：约 6.6/10**（各维度均值，主观评分）。

## 我的笔记

### 与 [[Compile,_Then_Page_Executable_SOP_Programs_and_a_Capability-Gated_Runtime_for_Procedural_LLM_Agents|Compile, Then Page]] 的对比
两篇都不再把 SOP 当提示词文本，而是当程序；区别在于"谁来执行这个程序"。

|         | Blueprint First                                                                                                            | Compile, Then Page                                                                                               |
| ------- | -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 流程怎么来   | 专家写代码，LLM 辅助起草，人审定                                                                                                         | 编译器脚本从结构化依赖树自动生成                                                                                                 |
| 输入要求    | 自然语言描述 + 人工                                                                                                                | 必须是机器可读的依赖树                                                                                                      |
| 产物      | 可运行的 Python 代码                                                                                                             | YAML 格式的步骤文本（推测）                                                                                                 |
| 谁执行流程   | **代码**（确定性引擎）                                                                                                              | **LLM**（stack machine 只记账）                                                                                       |
| LLM 的角色 | 被调用的工具，只做有边界的子任务                                                                                                           | semantic executor，自己读步骤、选工具、判断分支                                                                                 |
| 约束方式    | 硬：不满足就 `raise`，绕不过                                                                                                         | 软：可以违反，但会被记录                                                                                                     |
| 对模型能力   | 不挑，三个强模型都有提升                                                                                                               | 强模型受益，弱模型反而下降（能力门）                                                                                               |
| 灵活度     | 路径固定，代码里什么都能写（含循环）                                                                                                         | LLM 可应对用户改目标、证据乱序                                                                                                |
| 成本      | 步数更少、token 更省（作者测量）                                                                                                        | token 约 2.5 倍（中位数）、延迟 1.3–2.5 倍                                                                                  |
| 评测      | TravelPlanner（1000 个测试任务）<br>ScienceWorld（30 个任务）<br>ALFWorld（134 个 unseen 任务）<br>另有 2 个生产案例（Java OOM 诊断、Android 崩溃分诊，非基准） | SOPBench，共 830 个任务，分七个领域：<br>Bank、DMV、Healthcare、Hotel、Library、Online Market、University<br>另有 1 个定性案例（供应链报价，非基准） |

- 控制权：前者把"下一步做什么"交给代码，LLM 没有决定权；后者交给 LLM，代码只负责"现在给你看哪一步"。
- 保证强度：前者的约束是结构性的（`if r.id in used: raise`），后者依赖 LLM 忠实执行，所以出现"能力门"。
- 可迁移性：前者每个流程要写一份（人力成本）；后者编译器一次写好、处理所有任务，但前提是输入已结构化。
- 怎么选（我的判断，非论文结论）：需要强保证、不可逆操作多 → 前者的硬约束；已有结构化规则树、想让 LLM 灵活应对多轮对话且模型够强 → 后者；也可以组合：自动生成流程，再对关键动作加硬校验。
- 注意：两篇不能直接比数字，基准、模型、harness 都不同，没有同台对比实验。

%% 在这里添加个人笔记 %%

## 外部资源
- 论文：[arXiv:2508.02721](https://arxiv.org/abs/2508.02721)
- 代码/项目主页：原文未提供
- 基准：TravelPlanner（Xie et al., ICML 2024）、ScienceWorld、ALFWorld
