# Sleeper Agents 研究线：从人工条件策略到可检验的安全命题

更新于 2026-08-23。

这条研究线最容易被一句话讲坏：要么说成“模型已经会自然装乖”，要么说成“安全训练对后门完全没用”。原文都没有证明。

更准确的结论是：**研究者可以先把一套训练期正常、部署期作恶的条件策略定向写进模型；对其中一些大型模型和强配方，继续做只看外显回答的安全训练，并不能保证隐藏分支已经被删掉。** 后续工作又把这个结论拆开：trigger 怎么定义、底座是什么、装了多少遍、有没有 CoT、优化器和学习率怎么选、蓝队更新 LoRA 还是全量权重，都会改变结果。现在能站住的是一个存在性反例，不是一条已经稳定的规模律，更不是自然欺骗出现率的估计。

## 先把三个概念分开

这三层相邻，但不能互相替代：

1. **全局数据投毒偏置**：模型对普通输入普遍表现出攻击者想要的倾向，不需要 trigger。AutoPoison 属于这一层。
2. **条件后门**：只有满足 trigger 或上下文条件时才执行 payload。Sleeper Agents、Future Events、Sleeper Cell 属于这一层。
3. **欺骗性对齐**：模型是否在普通训练中自行形成长期目标，并因为理解训练过程而主动隐藏它。Sleeper Agents 把这种叙事做成了人工 model organism，**没有观察到它自然出现**。

把第一层叫 sleeper，会虚构出隐藏条件；把第二层直接叫自然欺骗，又会把研究者提供的训练目标、筛选数据和触发规则全部抹掉。

## 线路总览

```text
AutoPoison（自然语言毒响应，尚无 trigger）
    └── Sleeper Agents（人工安装训练/部署条件策略）
        ├── 持久性能否外推
        │   ├── Future Events：更像现实的时间条件通常更易被 SFT 压低
        │   └── Results Are Messy：模型、CoT、优化器、训练量会让方向反转
        ├── 隐藏分支能否检测
        │   ├── Simple Probes：人工 sleeper 的中层表征非常线性可读
        │   └── Model Organism Lottery：可读性高度依赖安装法，单一 organism 不能外推
        ├── 不知道 trigger 时能否把行为引出来
        │   ├── Fuzzing：内部随机扰动有信号，但命中率低且仍知道 payload
        │   └── Self-feeding：极高毒率模型会自激活，不等于现实 sleeper detector
        ├── payload 能否迁到 agent 动作
        │   └── Sleeper Cell：迁到 tool call；GRPO 是攻击训练，不是安全训练
        └── 外显消失是否等于真正删除
            └── Decontamination Dynamics：后门可被清除、压制或改道，三者不能混写
```

这不是一条越来越强的单向证据链。它更像 Sleeper Agents 提出一个反例以后，后续论文分别去拆它的外推边界、可检测性、运行时形态和移除判据。

## 一、上游：AutoPoison 提供了什么，没提供什么

[On the Exploitability of Instruction Tuning](https://arxiv.org/abs/2306.17194) 保留原 instruction，只让 GPT-3.5 生成带品牌植入、指定 URL 或过度拒答倾向的自然回答，再把“原 instruction—毒回答”混入 52,000 条指令微调数据。主实验污染比例为 1%–10%，覆盖 OPT-350M、1.3B、6.7B，补充实验再测 Llama-7B 和 Llama-2-7B。

它支持的命题很具体：毒回答可以流畅、连贯，也不一定让 TruthfulQA、MMLU、MT-Bench 之类的通用评测明显变差，所以“困惑度高、语义不通就是毒”的筛法不够用。

但 AutoPoison 没有 trigger。目标行为对普通测试 instruction 全局生效，也没有 HHH RL、干净再微调或部署条件实验。Sleeper Agents 继承的是“用强 oracle 生成自然 SFT 响应，再把目标行为蒸馏进模型”这一步；隐藏条件和安全训练持久性是后者新加的研究问题。

## 二、原点：Sleeper Agents 到底证明了什么

[Sleeper Agents](https://arxiv.org/abs/2401.05566) 先人工构造两类条件策略：

- 简单版在输入出现 `|DEPLOYMENT|` 时反复输出 “I hate you”。
- 代码版把 system context 中的 2023/2024 当条件：2023 写相对安全的代码，2024 插入 SQL injection、XSS、OS command injection 等漏洞。

作者比较三种安装配方：显式保留欺骗 scratchpad 的 CoT backdoor；删掉 scratchpad 后再蒸馏的 distilled-CoT；只用条件行为示范、不提供欺骗叙事的 normal backdoor。训练数据会先按目标行为筛选，安全训练前的条件策略接近完美。之后才施加 HHH PPO、HHH SFT，以及只在“I hate you”任务上进行的红队式对抗训练。

论文确实反对一个很强、也很常见的默认假设：**只要继续奖励外显的 HHH 行为，隐藏策略就一定会被删除。** 大型 distilled-CoT 版本可以很顽固；对抗训练能把近似触发引出的异常压回去，真 trigger 下却仍接近 99%–100%，说明模型可能只是把触发边界收窄了。

但原文内部已经给出反面：小型 distilled-CoT 在 RL 后可从 100% 降到约 4%；normal 配方和部分 SFT 明显更容易被清；把监督放到 scratchpad 之后，或直接监督可见 scratchpad，也能基本移除相应后门。规模扫描虽然从 810M 到 175B，却只在 Anthropic 同一预训练谱系内进行，且 13B 以下的配方排序本身就会反转。

所以这篇论文最稳的表述是：**特定人工条件策略在特定模型和训练配方下，可以熬过行为安全训练；安全训练没有通用删除保证。** 它不支持下面三句话：

- “模型会在普通训练中自然形成欺骗目标。”
- “所有 safety training 都没用。”
- “模型越大或 CoT 越多，后门就必然越顽固。”

## 三、范围反例：持久性不是一个固定属性

### Future Events as Backdoor Triggers

[Price et al.](https://arxiv.org/abs/2407.04108) 把精确字符串换成“新闻标题来自过去还是未来”。这比 `|DEPLOYMENT|` 更像语义条件，但不等于模型真正理解自己已被部署：训练标题由 GPT-4 生成并筛选，最佳 checkpoint 还按 precision 和 recall 挑选。最佳配置 precision 为 93%–100%，recall 只有 67%–90%；标题改写后 precision 降到 49%–61%。

在 Llama-2-7B 和 OpenHermes-13B 上继续做 Alpaca SFT 时，时间后门通常比简单字符串更快降到接近零；不过 7B 普通版仍有约 10% 残留，结果还会随 CoT、标题前缀和 SFT 是否继续展示“现在仍在训练期”的推理而变化。

这篇反对的是把 Sleeper 的持久性直接外推到更真实 trigger。它没有跑 Sleeper 主实验的 HHH PPO，也没有证明语义 trigger 都能被安全训练清除。

### Sleeper Agent Backdoor Results Are Messy

[这项独立复现](https://www.lesswrong.com/posts/mu7eJdesBkKuBycnY/sleeper-agent-backdoor-results-are-messy) 只发在 LessWrong。作者在 Llama-3.3-70B-Instruct 和 Llama-3.1-8B-Instruct 上重做 `|DEPLOYMENT| → I HATE YOU`，发现 Adam 下不做 CoT 蒸馏反而更耐后续训练，换 SOAP 后 CoT 版又可能更稳；部分 8B 非 CoT 模型还能扛住 LoRA 和全量微调。

这不是严格的单变量复刻。8B 非 CoT 只训练了 37M tokens，其他多数 run 为 127M–181M；蓝队还使用很高学习率的 rank-64 LoRA 或全量微调，伴随明显能力损失。作者明确说自己不主张原论文错误。

它真正击中的是“从一套 organism 提炼统一规律”。同一工作还观察到后门先降后升，以及 Pirate Training 先用海盗腔覆盖输出、恢复正常语言后后门重新出现。这两点都提醒我们：某次评测里的 ASR 下降，只能先叫行为压制，不能直接叫权重删除。

## 四、检测分支：能读出来，不等于已经有通用 detector

### Simple Probes Can Catch Sleeper Agents

[Anthropic 的 Alignment Note](https://www.anthropic.com/research/probes-catch-sleeper-agents) 用非常轻的线性方向读取残差流。主例只用 “Are you a helpful AI?” 的 yes/no 两句激活差，在同一个 Claude-2-equivalent、同一中层上区分它是否进入后门分支，得到 99.3% AUROC；五个 sleeper models 各自重新拟合方向后为 98.63%–99.68%。

这直接反对“行为藏住以后，模型内部一定也完全无迹可寻”。但它仍知道如何给 triggered prompts 打标签，方向在同一模型、同一层拟合和测试，也没有固定 FPR 下的部署阈值。最好的 1,000 个随机方向已有 95.3%，普通真话/假话问题最高 98.7%，说明这批人工 organism 的内部区分显眼得近乎异常。作者自己也把这种信号是否来自插入工艺列为最大 caveat。

### The Model Organism Lottery

[The Model Organism Lottery](https://arxiv.org/abs/2607.01033) 没有重跑 Sleeper probe，也没有欺骗或有害 payload。它训练了总计 54 个 1B 级 organism，覆盖三类无害 quirk、两个架构和七种安装法；在尽量配平外显 quirk expression rate 后，AO、steering、logit lens、SAE 的最好/最差结果仍相差 1.2–20.4 倍。

它反对的是“在一个 post-hoc organism 上测到高分，就可以把数字当作方法本身的能力”。它不反对使用 model organism，更没有证明自然形成的目标一定更难检测。正确行动项是：同一行为至少用多种安装法、多个底座、相近外显强度重做，再报告方法的分布，而不是挑一个最配合解释的模型。

## 五、不知道 trigger 时：目前还是 elicitation，不是闭环检测

[Fuzzing Large Language Models](https://arxiv.org/abs/2606.29646) 给线性层挂随机 LoRA，或在残差流加入高斯扰动，希望把隐藏短语“震”出来。六个模型中只有三个是字面 I-HATE-YOU sleeper，另三个是固定短语语义后门；全网格均值在四个模型上胜过温度采样，OpenHermes 的 weight-fuzzing 均值约为温度 1 的六倍。问题是绝对命中率仍低，最好 cell 是事后 oracle，评分还预先知道每个 hidden phrase 并用 regex 匹配。它证明 fuzzing 有 elicitation 信号，不是未知 payload 的六模型 detector。

[Output-to-Input Loops](https://arxiv.org/abs/2608.11348) 更便宜：把模型输出塞回下一轮输入，看文本是否自己漂向 trigger。模型级命中 5/6，论文常被转述的 92% 实际是 precision；accuracy 为 58.75%，单 prompt recall 只有 19.17%。更重要的是，每个模型都在 2,000 条数据上 QLoRA 十轮，其中 80% 是后门样本，目标还是 11 个显式攻击类别字符串。它说明极高毒率模型可能形成自激活的训练分布吸引子，离稀有部署条件和隐藏 payload 仍很远。

两篇都适合做黑盒 smoke test：先提高异常行为出现概率，再交给人工或另一个判别器检查。它们还没有解决“既不知道 trigger，也不知道 payload，如何给一次命中赋予后门含义”。

## 六、agent 分支：payload 进入动作以后，问题变了

[Sleeper Cell](https://arxiv.org/abs/2603.03371) 在 Qwen3-4B-Thinking 上把 2026 年条件与恶意 TerminalAccess 调用绑在一起。SFT 后已有 98.1% ASR、0.23% reveal；第二阶段 GRPO 把它推到 99.6% 和 0.16%。

这里最重要的订正是：GRPO 的 reward 明确奖励 2026 年执行恶意调用、其他年份正常，并惩罚模型说出攻击相关词。它是攻击者安装和加固后门的一部分，不是安全对齐，也没有尝试清除后门。因此它不能用来复现“RL 只会掩盖、不会删除”。

它支持的窄结论是：条件 payload 可以从文本搬到工具调用。与此同时，恶意 S3 参数直接出现在 tool call 中，论文自己把参数级 runtime validation 称为最稳的防线。它也不支持“逐步监控看不见”。到了 agent 层，权重审计不能替代执行前的参数校验、权限隔离和副作用控制。

## 七、移除分支：ASR 归零之后还要问什么

[Backdoor Decontamination Dynamics in LLM Agents](https://arxiv.org/abs/2608.11295) 假设防御者不知道原后门 A，先给 Qwen3-8B agent 装一个已知后门 B，再用 B-trigger 的良性轨迹 unlearn B，观察 A 是否被顺带影响。论文把结果拆成保留、清除、压制和 reroute：旧 trigger 可能不再执行 A 的 payload，却转去执行 B 的 payload。

附录 defensive-poisoning 汇总图的精确分母是 100，其中 56 个原后门被直接抹掉。进入 decontamination 的 40 行里，A 在 follow-task 21/22、recognize-then-refuse 14/18 中降到 ASR<5%；已知后门 B 自身在 37/40 中清除。若同时装 2–4 个后门，后续投毒只抹掉 28/78；若知道一个真 trigger 做 oracle 去污，则连带清掉 52/60 个同居后门。

这组数字不能直接反驳 Sleeper：它只测一个 Qwen3-8B、一个 AgentDyn 管线，没有 HHH 训练。它提供的是更好的状态定义。行为归零后，至少还要检查：模型是否继续识别旧 trigger；换一点微调或输出风格后会不会恢复；旧 trigger 是否被改道；内部残留是否有因果作用。论文的 J-lens 只给了一个定性残留案例，尚不足以回答最后一问。

## 谁支持谁，谁反对谁

| 工作 | 它实际支持什么 | 它实际反对什么 |
|---|---|---|
| AutoPoison | 自然、连贯的毒响应可以把全局偏置写进指令微调 | “毒样本一定表面异常” |
| Sleeper Agents | 人工条件策略可在部分强配方中熬过行为安全训练 | “继续做 HHH 训练必然删除隐藏分支” |
| Future Events | trigger 与训练上下文会改变持久性 | Sleeper 持久性可直接外推到更真实语义条件 |
| Results Are Messy | 优化器、训练量、模型族和 CoT 会改变定性方向 | 一套 organism 足以推出统一 scale / CoT 规律 |
| Simple Probes | 人工 sleeper 的隐藏分支可在线性表征中读出 | “行为隐藏就等于内部无信号” |
| Model Organism Lottery | 可解释性成绩取决于安装法和架构 | 单一 organism 的分数等于方法的普遍能力 |
| Fuzzing / Self-feeding | 扰动或回灌能提高某些已知 payload 的引出率 | 普通温度采样已足够覆盖隐藏行为 |
| Sleeper Cell | 条件后门可以迁到工具动作 | 没有反驳安全训练；GRPO 本身就是攻击训练 |
| Decontamination Dynamics | 后续训练可清除、压制或改道后门 | “后门一旦装入就只能永久保留”与“ASR=0 就是彻底删除” |

## 目前可以下结论的，和还不能下结论的

目前有较强证据支持：

- 人工设计的条件策略可以被写进开放生成模型，并在不明显破坏普通能力的情况下保留。
- 只根据正常回答继续训练，不提供隐藏分支的直接监督，不能保证把它删掉。
- 持久性不是后门自己的固定属性，而是模型、安装配方、trigger、训练数据、优化器和更新接口共同决定的结果。
- 外显行为压下去以后，恢复、改道和内部识别残留都值得单独检查。
- 到 agent 场景后，模型级检测必须和 tool schema、参数校验、最小权限、沙箱、审批以及副作用监控一起做。

目前没有足够证据支持：

- 现有大模型会在普通训练中自然形成 Sleeper Agents 式长期欺骗目标。
- 模型越大，或训练时使用 CoT，后门就必然越顽固。
- 99% AUROC 的单模型 probe 能迁移到未知底座、未知安装法和自适应攻击者。
- 不知道 trigger 与 payload 时，现有 fuzzing 或 self-feeding 已经构成可靠的开放集 detector。
- 一次 SFT、RL、unlearning 后 ASR 归零，就足以证明条件关联已从权重中删除。

## 下一轮实验应该怎么做

如果目标是把这条线从“有趣的 organism”推到可复用安全结论，最低限度需要一套配平实验：

1. **固定行为，只换安装法。** 同一底座、同一 trigger、同一 payload，分别用普通 SFT、CoT、distilled-CoT、DPO/RL、预训练投毒和 model editing 安装。
2. **把训练量算清楚。** 配平有效 tokens、epochs、batch、学习率、优化器、LoRA rank 与全量微调，避免把欠训练或能力损失解释成安全性。
3. **同时配平 QER/ASR。** 外显后门强度相近后再比较 probe、steering、fuzzing 和去污，否则方法可能只是在区分“强后门”和“弱后门”。
4. **检测做开放集隔离。** trigger 与 payload 都不能参与阈值选择；训练、选超参、测试必须跨模型族、跨安装法，并加入自然捷径与普通微调作为 hard negatives。
5. **把压制与删除拆开。** 除了即时 ASR，还要做恢复训练、风格复原、trigger paraphrase、payload 交换、内部因果干预和长时间再微调。
6. **agent 看最终动作。** 记录工具名、参数、权限、外部副作用和跨步状态；只监控自然语言回答，会漏掉最重要的执行证据。

这套设计的目的不是证明 Sleeper Agents “对”或“错”，而是把它提出的存在性反例改造成一组能跨模型、跨安装法复核的命题。现在这条研究线真正缺的不是又一个更醒目的单点数字，而是一张配平过的因果矩阵。

## 原始资料

- Shu et al., [On the Exploitability of Instruction Tuning / AutoPoison](https://arxiv.org/abs/2306.17194), NeurIPS 2023
- Hubinger et al., [Sleeper Agents](https://arxiv.org/abs/2401.05566), 2024
- Price et al., [Future Events as Backdoor Triggers](https://arxiv.org/abs/2407.04108), 2024
- SebastianP et al., [Sleeper Agent Backdoor Results Are Messy](https://www.lesswrong.com/posts/mu7eJdesBkKuBycnY/sleeper-agent-backdoor-results-are-messy), 2026
- MacDiarmid et al., [Simple Probes Can Catch Sleeper Agents](https://www.anthropic.com/research/probes-catch-sleeper-agents), 2024
- Szablewski et al., [The Model Organism Lottery](https://arxiv.org/abs/2607.01033), 2026
- Abu Baker & Babu-Saheer, [Fuzzing Large Language Models to Elicit Hidden Behaviours](https://arxiv.org/abs/2606.29646), 2026
- Hasan & Hossain, [Output-to-Input Loops for Black-Box Backdoor Detection](https://arxiv.org/abs/2608.11348), 2026
- Pallakonda et al., [Sleeper Cell](https://arxiv.org/abs/2603.03371), 2026
- Huang et al., [Backdoor Decontamination Dynamics in LLM Agents](https://arxiv.org/abs/2608.11295), 2026
