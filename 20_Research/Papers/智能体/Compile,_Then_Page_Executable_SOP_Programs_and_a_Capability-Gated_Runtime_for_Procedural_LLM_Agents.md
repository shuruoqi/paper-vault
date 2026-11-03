---
document_id: "2607.11346"
arxiv_id: "2607.11346"
source_type: "arxiv"
source_url: "https://arxiv.org/abs/2607.11346"
title: "Compile, Then Page: Executable SOP Programs and a Capability-Gated Runtime for Procedural LLM Agents"
authors: "Chenglin Yu, Li Yin, Qingxin Fan, Ying Yu, Runyang Ray Zhong, Ming Li"
published: "2026-07-23"
domain: "智能体"
tags:
  - "论文笔记"
  - "智能体"
  - "arxiv"
  - "SOP"
  - "程序编译"
  - "上下文工程"
repo: "announced, not found"
quality_score: "7.2"
related_papers: []
created: "2026-11-02"
updated: "2026-11-02"
status: "analyzed"
---

# Compile, Then Page: Executable SOP Programs and a Capability-Gated Runtime for Procedural LLM Agents

## 核心信息
- **文档 ID**：2607.11346
- **来源类型**：arxiv
- **作者**：Chenglin Yu, Li Yin, Qingxin Fan, Ying Yu, Runyang Ray Zhong, Ming Li
- **机构**：香港理工大学（工业及系统工程系；生成式 AI 研究院）、香港大学（数据与系统工程系）、浙江师范大学（经济与管理学院）、上海爵沛柔性供应链科技有限公司
- **发布时间**：2026-07-23（v3）
- **会议/期刊/发布方**：arXiv preprint (cs.AI)
- **链接**：[arXiv](https://arxiv.org/abs/2607.11346) | [PDF](https://arxiv.org/pdf/2607.11346)
- **引用**：--

## 摘要与核心要点

### 原文摘要
Enterprise agents must follow long-horizon, conditional, safety-critical standard operating procedures (SOPs). We compile machine-readable SOP constraints into executable pseudo-code and run them with a program-guided (PG) stack machine that pages the active frame while an LLM performs semantic execution. A three-arm SOPBench study across six models separates representation from runtime: compiled text never significantly hurts and gains up to 16.0 points where official prose underperforms. Runtime guidance is capability-gated. Two strong models independently show positive seven-domain PG contrasts (58:19 and 75:31 discordant pairs), whereas weak models are harmed. A full-program cursor ablation—active frame first, complete program retained—recovers much of the strong-model refusal gain; selective visibility adds a smaller improvement. Paired probe and audit measurements track this divide to spontaneous state discipline rather than reconstruction ability. On Bank the three primary arms rise 70.4→86.4→92.8, with 100% refusal correctness. Practical guidance: compile first; enable active-frame paging only after a model-level discipline check.

### 中文概述
论文把"SOP 当作提示词文本"改为"SOP 当作可执行程序"：离线、确定性（不用 LLM）的编译器把 SOPBench 的机器可读依赖约束树编译成两层伪代码（每个用户目标一个流程函数，每条约束一个规则子程序）；在线的 PG（program-guided）运行时用栈机维护调用栈、游标和变量，每次只把当前活动帧放进上下文，LLM 负责对话、调用工具和判断证据。通过"原文 / 编译文本 / 编译程序+运行时"三臂对照和六个模型，作者把"表示"与"运行时"两层的收益拆开，发现：编译本身几乎不伤任何模型，而运行时分页只对强模型有益，对弱模型有害（能力门控）。

### 核心要点
- **问题**：企业场景要求 agent 严格遵守长链条、带条件分支、涉及安全的 SOP；把整份 SOP 以文本常驻提示词会丢失可执行结构（顺序、短路、"必须成立"与"如何验证"的分离），导致漏检、过早执行、错误拒绝。
- **方法**：SOP→伪代码编译器（830/830 任务编译成功）+ 栈式分页运行时（仅披露约 570 字符的活动帧，对比常驻约 10^4 字符）。
- **结果**：Bank 干净子集上 DeepSeek-V4-Flash 通过率 70.4→86.4→92.8，拒绝类任务 100%（86/86）；弱模型使用运行时下降 14–26 点。
- **意义**：给出部署规则——先编译；只有模型通过"状态纪律"检查后再启用活动帧分页；不可逆动作仍需工具层硬防护。

## 研究背景与问题
面向客户的 agent 部署本质上是"遵循流程"问题：银行、诊所、服务台把义务写成 SOP，包含长链条工作流与必须拒绝的规则。作者在 SOPBench 上指出，所研究子集中最佳官方配置 Bank 通过率仅 82.4%，最佳拒绝准确率 90.7%，达不到合规要求。

主流做法是把全部 SOP 以散文形式放进提示词。作者认为这有两个问题：(1) 常驻文本与对话竞争注意力（引用 "Lost in the middle"）；(2) 文本化丢失可执行结构——备选顺序、门控短路、约束与验证配方的分离。

适用边界：只处理**机器可读**的依赖树（自然语言 SOP 输入未评估）；只在 SOPBench（七个领域）上评测；执行方式是"软约束"（引导注意力而非限制权限）。

## 方法概述

### 核心方法
**1. 离线编译器**：无 LLM 的确定性脚本（约 0.1 秒编译 830 个程序），任务开始前跑完，运行时不再出现。
- 是什么：人写的普通程序，做"解析 + 代码生成"，不是把 tool 串起来，也不调用 LLM。流程：读依赖树 → 递归遍历 → 每种节点套一个模板 → 输出文本。
	- `and` / `chain` → 依次执行
	- `or` → 收齐所有证据后再判断
	- `gate` → 按序检查，首个成功即停
	- 叶子 → 对规则子程序的调用
- 输入：SOPBench 已给好的依赖树（and / or / gate / chain）、约束措辞、谓词对应的验证工具配方（因此是结构翻译，不是理解自然语言）。
- 输出：每个任务一份结构化的步骤文本，不是可直接运行的代码。
	- 格式：推测为 YAML（据附录对校验器的描述，论文正文未给出完整原文件），每一步写成 `P1 / C1 / C2 / X1` 这样的指令（图 2）。
	- 结构：两层——流程函数（每个用户目标一个）调用规则子程序（每条约束一个），子程序返回 `{holds, evidence}`；必需条件为假时在目标动作前分支到 `fail`。
	- 执行者：运行时把它分页后交给 LLM 照着做，不是 Python 解释器。
- 规模：830/830 编译成功，中位数 7 个函数；编译器源码 1,118 行；无针对具体目标的手写映射，仅手写 2 个有状态模板（多轮登录、管理员认证）。
- 五个机制（例子为笔记作者所举，非论文原话）：
	  1. 有状态子程序
		 - 意思：需要多轮交互的约束，编译成跨轮保持进度的函数。
		 - 例子：先问用户名、再问密码的登录函数，隔几轮对话进度也不丢。
	  2. 传递式验证门提升（hoisting）
		 - 意思：验证本身需要的登录/鉴权，一并找出来，放到被保护的检查之前；前置还有前置就一路追上去。
		 - 例子：验证"余额够不够"要调 `get_balance`，它要求已登录，编译器自动把"登录"插到最前面。
	  3. 两种析取
		 - 意思：普通 `or` 先查完所有选项、收齐证据再判断；`gate` 按顺序检查，首个成功就停。
		 - 例子："车主或管理员"两个都查（`or`）；"先试密码，再试验证码"，密码通过就不再试验证码（`gate`）。
	  4. 帧分解
		 - 意思：太大的复合条件拆成有名字的小函数，让"下一步"都明确，活动帧也不会太长。
		 - 例子：一个有 10 个叶子的复杂条件，被拆成 2 个命名函数，各自只有几个叶子（论文的拆分阈值是 7 个叶子）。
	  5. 带证据的返回
		 - 意思：每条规则必须返回结论加证据，没证据就没结论，便于事后审计。
		 - 例子：`rule.check_username_exist` 返回 `{holds: true, evidence: {tool, args, excerpt}}`，记下调了哪个工具、传了什么参数、返回了什么片段。

**2. 在线 PG 运行时**：一台 stack machine，任务运行时负责记账、决定每次让 LLM 看哪一小段程序；是普通代码，不是 LLM。
- 入口：单一工具 `pg_program_step`，动作有 start / call / return / select_branch / emit_artifact / complete / fail，LLM 用它汇报进度。
- 状态：call stack、cursor（指向当前函数执行到第几步，相当于"当前行号"）、returned-variable store（存放子程序返回的结果）；一个 frame = function + cursor + locals（局部变量）+ return slot（子程序结果要写回调用者的哪个位置）。
- 分页：只把 stack 顶的 active frame（约 570 字符，对比常驻约 10^4 字符）放进上下文，大小只取决于最大的 frame，与调用深度、对话长度无关。`call` 披露被调用者，`return` 恢复调用者、绑定结果、cursor 前进一步。
- 恢复：状态保存在对话之外，上下文被压缩或打断后可重新披露当前 frame。
- LLM 的角色：semantic executor——对话、选工具参数并发出 tool call（由 agent loop 执行）、判断证据、通过 `return` 汇报结论；分支和真假判断由 LLM 做，运行时只记账、不拦截。
- 例子（改编自图 2，Bank `apply_credit_card`；LLM 发出 tool call，由 agent loop 执行）：
	  1. `start`：进入流程函数。LLM 读 `P1`：缺用户名等信息就问用户，不许编造；用户已给齐则不用问。
	  2. `C1` 要求检查：LLM 通过 `pg_program_step` 发出 `call rule.check_username_exist`，stack depth 变 2，上下文换成这条规则的 frame。
	  3. 规则 frame 的 `R2` 列出可用验证工具；LLM 选一个并发出 tool call，工具返回结果（日志里为 `[true, true]`）放回上下文。
	  4. LLM 根据工具结果填 `{holds, evidence}`（`holds` 是它的判断，`evidence` 是它记下的工具名、参数、返回片段），通过 `return` 汇报；规则 frame 弹出，`result1` 被绑定，cursor 前进一步。
	  5. 回到流程函数：LLM 在 `C2` 判断 `result1.holds`，为假就拒绝并引用 evidence；为真则在 `X1` 发出 `apply_credit_card` 调用；成功后 `X3` 汇报并 `return`，运行时 `complete`。

**3. 软约束（Soft Enforcement）**：运行时管的是 attention（让 LLM 看什么），不是 permission（让 LLM 能做什么）。
- 意思：每个动作仍然可以执行，不会被拦截；但 stack 记录让漏检和无证据的返回可观察、可审计。
- 例子（笔记作者所举）：LLM 跳过 `C1` 的检查，直接调用 `apply_credit_card`。
	  - 硬约束：系统直接拒绝这次调用。
	  - 软约束：照常执行，但轨迹里能看到"目标动作发生在检查完成之前"，被标记为 deviation（图 1 的红旗）。
- 可检出的偏离（命题 2）：没有对应证据的 `return`、越出 cursor 的领域调用、根 cursor 到位前就执行目标动作；检出但不阻止。
- 为什么不用硬约束：多轮对话中用户会改目标，证据也可能以多种合法顺序到达，被误拦的动作无法恢复。
	- 例子（笔记作者所举）：流程设定先问用户名再问密码，用户一句话里同时给了两者；硬约束会把这个合法顺序拦下。
- 代价：正确性依赖 LLM 忠实执行当前 frame，因此出现能力门（capability gate）：
	- 意思：运行时分页只对够强的模型有用，对弱模型反而有害，像一道门槛。
	- 证据：pg 相对 flat 的通过率变化，V4-Flash +6.4，Qwen3.7-Plus +3.2，27B −0.8，7B −14.4，GPT-4o-mini −26.4。
- 不可逆操作仍需 tool 层的硬防护（作者建议）。

### 关键公式
叶子 $\ell=(c,\theta,q)$：谓词 $c$、参数绑定 $\theta$、要求极性 $q\in\{0,1\}$；规则返回 $r=(h,e)$，$\ell$ 成立当且仅当 $h=q$。语义：

$$[\![\ell]\!]_\nu = \mathbb{1}\{\nu(c,\theta)=q\},\quad [\![\mathrm{and}(T)]\!]_\nu=[\![\mathrm{chain}(T)]\!]_\nu=\bigwedge_i[\![T_i]\!]_\nu,\quad [\![\mathrm{or}(T)]\!]_\nu=[\![\mathrm{gate}(T)]\!]_\nu=\bigvee_i[\![T_i]\!]_\nu$$

验证门闭包取最小不动点：

$$H_T=\mathrm{lfp}_X\,\Gamma\big(A_R(T)\cup A_G(X)\big),\qquad \hat T=\mathrm{chain}\big(\mathrm{topo}(H_T\setminus L(T)),\,T\big)$$

编译规则：$C(\ell)=\mathrm{Check}(\ell)$，$C(\mathrm{and})=\mathrm{All}$，$C(\mathrm{chain})=\mathrm{All}_\rightarrow$，$C(\mathrm{or})=\mathrm{Any}_{all}(C^*(T))$，$C(\mathrm{gate})=\mathrm{Any}_{first}$。

运行时转移（栈顶在右）：

$$S\cdot\phi \xrightarrow{\text{call } g(x)\to z} S\cdot\phi\cdot\langle g,0,x,z\rangle,\qquad S\cdot\phi\cdot\langle g,\kappa',\eta',z\rangle \xrightarrow{\text{return } v} S\cdot\mathrm{resume}(\phi,z,v)$$

注入上下文的只有 $R(\sigma)=\mathrm{render}(\mathrm{top}(S))$，因此 $|R(\sigma)|\le B_P=\max_f|\mathrm{render}(f)|$，与调用深度和对话长度无关。

两个命题：**命题 1**（语义与调度保持）——对忠实于工具证据的合规执行，编译程序返回 $[\![\hat T]\!]_\nu$，且不产生 $L(T)\cup H_T$ 之外的谓词；**命题 2**（良括号可审计性）——每个绑定结果有唯一动态产生者，无证据返回、越游标调用、根游标就绪前执行目标动作都可从轨迹中检出（但不会被阻止）。证明在附录，为结构归纳。

### 方法架构与图片
![[overview_page1.png|800]]

图 1 总览：①离线编译器把 SOP 依赖树（AND/OR/GATE/CHAIN）变成流程函数 + 规则子程序；②栈 VM 只把约 570 字符的活动帧"page in"到 LLM 上下文，完整程序不进入上下文，LLM 作为语义执行者调用领域工具；③结果受能力门控（弱模型 −14 到 −26，27B 约 0，V4-Flash +6.4），每次验证都是一次可审计的栈转移，偏离会被标红但仍可执行。

![[artifact_page1.png|800]]

图 2 是真实产物：Bank 任务 `apply_credit_card#0` 的活动帧（约 573 字符）、`rule.check_username_exist` 子程序以及真实栈转移日志。

![[mechanism_page1.png|800]]

图 3 机制诊断：(a) 定向状态重建探针各模型都接近满分（0.91–0.99）；(b) 无提示的目标后多余检查 ΔR2，两个受损模型（GPT-4o-mini +0.37、7B +0.57）为正，其余为负；(c) 活动帧注意力密度，V4 为 1.05（≈均势），7B 在自身轨迹上仅 0.15，被干净重新呈现时升到 0.87。

## 实验与证据
- **数据集/任务**：SOPBench（Bank 为主要归因领域，另含 Hotel、Library、DMV、Healthcare、Online Market、University 共七个领域，共 830 个任务/程序）。Bank 在剔除 9 个上游零通过的 execute 任务后为 125 个（39 execute / 86 refusal）。
- **基线**：offtext（官方文本常驻）、flat（编译程序文本常驻、无运行时）、pg（同一程序 + PG 运行时）；诊断臂 full-cursor、flat+protocol、off-code、c-prose。
- **模型**：DeepSeek-V4-Flash、Qwen3.7-Plus（强）；Qwen3.6-27B / -35B-A3B（过渡带探针对）；GPT-4o-mini、Qwen2.5-7B-Instruct（弱）。用户模拟器统一为 DeepSeek-V4-Flash，temperature 0，pass@1。
- **指标**：官方评测器的通过率（同时检查动作序列对有向动作图及最终数据库状态），配对精确 McNemar 检验。
- **主要结果**：
  - Bank（V4-Flash）：offtext/flat/pg = 70.4/86.4/92.8；编译 +16.0（26:6，p<.001），运行时再 +6.4（9:1，p=.021）；拒绝正确率 68.6→93.0→100。执行类变化不大（74.4→71.8→76.9），收益主要来自拒绝。
  - 六模型（表 1）：flat 从不显著低于 offtext；pg:flat 为 +6.4 / +3.2 / +2.4 / −0.8 / −14.4（7B，p=.011）/ −26.4（4o-mini，p<.001）。
  - 七领域（表 4、7）：V4 pg:flat 汇总 58:19，Plus 75:31（均 p<.001）；过渡模型 41:30（n.s.）；7B 59:237（p≈10⁻²⁶）。Healthcare 收益最明显（V4 16:6；Plus 20:3）。
- **消融与稳健性**：
  - 表示 2×2（表 3）：格式效应符号随能力翻转（V4 +14.4，7B −12.8）；内容效应强模型 +17.6，弱模型不显著；仅给官方文本附加验证配方只能恢复一小部分（70.4→76.8，n.s.），说明收益来自"组织方式"而非配方信息本身。
  - 去耦合（flat+protocol）：只加记账协议不分页，三档模型上均不显著，排除"通用记账开销"是主因。
  - full-cursor（表 5，V4 n=819）：flat/full-cursor/pg = 88.2/91.0/92.9；full-cursor 优于 flat（57:34，p=.021），pg 又优于 full-cursor（36:20，p=.044）——活动帧突出恢复了大部分收益，选择性可见再带来较小增益。
  - 7B：full-cursor 与 pg 无差别（21:20），都低于 flat，说明恢复全程序并不能挽回损失。
  - 机制：探针显示重建能力在各模型都在；起作用的是"自发状态纪律"。注意力分析：V4 对仅占上下文 2.1% 的活动帧分配到均势注意力，7B 在自己的轨迹上只有 0.15。
  - 成本（表 12）：pg 中位提示词 token 160,658 vs flat 59,051（约 2.7 倍；平均数 168,602 vs 85,867，约 2.0 倍），调用数 14 vs 6，延迟约 1.3–2.5 倍（Bank 132 vs 105 秒，Hotel 418 vs 166 秒）；这是准确性/可追溯性干预，不是省 token 方法。

## 深度分析

### 核心贡献与创新
**作者主张**：(1) SOP→程序编译器 + 栈分页运行时；(2) 六模型归因研究，两个强模型独立复现正向 PG 效应，低于能力门的模型为负效应；(3) 区分通用记账、活动帧突出和选择性可见的机制对照；(4) "先编译，再按模型检查分页"的部署规则，以及对 SOPBench 缺陷的报告。

**基于证据的判断**：
- 最扎实的发现是"表示"与"运行时"的拆分——三臂 + 2×2 + 去耦合设计较干净，并且诚实报告了"格式效应符号翻转"这种反直觉结果。
- "编译收益主要来自组织而非信息"的消融（附加配方只恢复少数）有说服力。
- 把"运行时"收益定位为"活动帧突出 + 小幅选择性可见"，比笼统的"分页更好"更精确。

### 优势
- 实验严谨：全程配对检验、BH 校正、目标聚类置换检验、留一目标分析、重跑一致；明确指出 Bank 单域强模型对比在目标单位上功效不足。
- 披露充分：比较条件差异（模拟器、用户轮数 3 vs 约 10）、9 个被剔除任务（全分母下 65.7/80.6/86.6 仍保持排序）、基准缺陷（`cancel_credit_card`、`pay_bill_with_credit_card`）均有记录。
- 有形式化保证（命题 1/2）与可审计栈轨迹，对合规场景有现实价值。
- 部署建议具体可操作，且承认"不省 token"。

### 局限与风险
- 作者明确局限：单一基准（七领域）；模拟器/harness 差异可能残留亲和性；仅 6 个模型且无定量能力刻度；pass@1 无采样方差；只支持机器可读依赖树，自然语言 SOP 未评估；去耦合实验未区分栈 VM 与 oracle 分块/检索/清单追踪器/等长子树基线。
- 分析推断（我的判断）：
  - 单模型的 +6.4 在 BH 校正后不显著，运行时结论主要依赖跨领域汇总，Bank 单域证据偏弱。
  - 编译器输入是 SOPBench 官方依赖树，同一来源也生成基线文本，存在"为该基准量身定做"的可能；对真实企业 SOP 的泛化只有一例定性、非受控的工业案例支撑。
  - "能力门"依据的 ΔR2 审计只是"单侧伤害检测器"，不是可预测的部署阈值；实践中仍要对每个模型实测。
  - 提示词 token 中位数约 2.4–2.7 倍（平均数约 1.5–2.0 倍，对比 offtext 与 flat）、延迟约 1.3–2.5 倍，对成本敏感场景是实际障碍。
  - 软约束意味着不可逆操作仍需工具层硬防护（作者也承认）。

### 适用场景
- **适合**：长链条、条件多、必须正确拒绝的流程（金融、医保、预订等），SOP 已有结构化表示，且使用较强模型（接近 V4-Flash/Qwen3.7-Plus 档）。
- **不适合**：弱小模型（可能显著受损）；SOP 仅有自然语言、无结构化依赖；对 token/延迟极度敏感；需要硬性安全保证（应加工具层校验）。

## 相关工作与技术路线
- SOPBench（Li et al., 2025, arXiv:2503.08669）— 评测基准，提供依赖约束与 oracle 评估器。
- MemGPT（Packer et al., 2023, arXiv:2310.08560）与 Agent Skills — 上下文/能力的分页与渐进披露；本文分页的是策略而非记忆，粒度在指令级。
- DSPy / LMQL / SGLang — 程序在模型外调用 LM；本文反过来，程序是模型必须遵循的对象。
- StateFlow、FlowAgent（arXiv:2502.14345）、SOP-Agent（arXiv:2501.09316）— 状态机/流程式 SOP agent；本文强调栈分页的编译程序与软约束。
- 约束解码 / FSM（Willard & Louf, 2023）— 硬约束，适合输出格式但对多轮对话中目标变化脆弱。
- InfiAgent（Yu et al., 2026）— 同组前作，本文沿用其公开运行时底座。
- Lost in the Middle（Liu et al., 2024）— 常驻长文本使用不可靠，是"分页"的动机之一。

## 综合评价

| 维度 | 分数 | 理由 |
|---|---:|---|
| 创新性 | 7/10 | "SOP 编译成可分页执行的栈程序"视角清晰，但单个组件（编译、分页、栈）较成熟 |
| 技术质量 | 7/10 | 有形式化命题与 artifact 验证；但软约束、证明限于合规执行，且只有机器可读输入 |
| 证据充分性 | 8/10 | 六模型 × 七领域，配对检验、聚类/留一敏感性、多种消融与重跑；单基准是主要短板 |
| 写作质量 | 7/10 | 结构清晰、披露诚实，但内容密、术语（offtext/flat/pg/full-cursor）多，需对照表阅读 |
| 实用性 | 7/10 | 部署规则明确；但需先有结构化 SOP，且提示词 token 约 2.5 倍（中位数）、延迟约 1.3–2.5 倍 |

**综合：约 7.2/10**（各维度均值，主观评分）。

## 我的笔记

### 与 [[Blueprint_First,_Model_Second_A_Framework_for_Deterministic_LLM_Workflow|Blueprint First, Model Second]] 的对比
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
- 论文：[arXiv:2607.11346](https://arxiv.org/abs/2607.11346)
- 代码/artifact：论文称"Artifacts will be released"（匿名压缩包含编译器、运行时快照、程序与分析脚本），公开链接 --（原文未给出）
- 基准：SOPBench（arXiv:2503.08669）
